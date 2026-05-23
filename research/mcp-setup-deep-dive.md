# MCP setup deep-dive

> Companion to `research/technical-overview.md`. The overview gives the surface; this file gives the wiring. Writers: pull facts from here for the install and concept pages. If anything contradicts the source, that's a bug — flag it via inbox.
>
> Source-of-truth files in the multisphere repo (referenced throughout):
> - `.claude-plugin/plugin.json`
> - `.mcp.json`
> - `manifest.json` (root)
> - `mcp-server/src/config.ts`
> - `mcp-server/src/index.ts`
> - `CLAUDE.md`

---

## 1. Two install paths, one server

Multisphere is **one MCP server** (`multisphere-mcp`, published to npm) reachable through **two install surfaces**:

| | Claude Code (and Cowork, now) | MCPB-only hosts (fallback) |
|---|---|---|
| **Manifest** | `.claude-plugin/plugin.json` + `.mcp.json` | `manifest.json` (root) |
| **What ships** | the plugin (skill + MCP wiring) | `.mcpb` bundle (MCP server only, no skill) |
| **Discovery** | `/plugin marketplace add unicity-labs/multisphere` then `/plugin install multisphere@unicity-labs` | host-specific MCPB install dialog |
| **Server binary** | `npx -y multisphere-mcp@latest` (npm-resolved at launch) | bundled `mcp-server/dist/index.js` (frozen at build time) |
| **Identity source** | files under `~/.multisphere/` | host's MCPB user_config dialog → env vars |
| **`MULTISPHERE_CLIENT` env** | not set in the shipped `.mcp.json` | hard-coded to `"cowork"` |
| **Carries the `a2a` skill** | yes | no — MCPB only carries the server |
| **Updates** | `npx@latest` resolves new server versions; plugin changes need a `plugin.json` version bump | rebuild + redistribute the `.mcpb` |

Cowork supports the plugin format now too, so the MCPB path is genuinely fallback — kept for non-plugin MCP hosts. Most users will be on the plugin path.

## 2. Plugin manifest — `.claude-plugin/plugin.json`

Pure metadata. It does **not** describe the MCP server; it just declares the plugin so the marketplace can list it and the client can register it.

Fields that matter:

- `name` — `"multisphere"`. Becomes the plugin id; appears in tool prefixes (see §6).
- `version` — gated by the plugin manager. **Bump this on any plugin-side change** (`.mcp.json`, skills, plugin.json itself, workspace template). Without a bump, existing installs do not see updates and users have to uninstall the whole marketplace to refresh the cache. See `CLAUDE.md` § "Releasing — version bumps matter".
- `displayName`, `description`, `author`, `repository`, `license`, `keywords` — surface metadata.

The companion `marketplace.json` (also in `.claude-plugin/`) is publisher-level (`name: "unicity-labs"`) and lists this plugin as a single entry pointing at `./`. Same directory because they're both manifests, not components.

## 3. MCP wiring — `.mcp.json`

```json
{
  "mcpServers": {
    "workspace": {
      "command": "npx",
      "args": ["-y", "multisphere-mcp@latest"]
    }
  }
}
```

Three things to know:

1. **Schema is the standard MCP config**: `{ "mcpServers": { "<key>": { "command", "args", "env"? } } }`. Whatever you put under `mcpServers` works the same as it does in any top-level MCP config.
2. **The server key is `workspace`, not `multisphere`.** Reason: client tool prefixes are built from the plugin name and the server key. Naming the server `multisphere` produced ugly `plugin:multisphere:multisphere:*` displays. `workspace` reads naturally (`plugin:multisphere:workspace:read`, etc).
3. **No env vars are set on the Claude Code path.** This is a real consequence: `MULTISPHERE_CLIENT` is unset, so the per-client identity file path (`~/.multisphere/identity.claude-code.json`) and the `user_slug + client → agent_id` auto-derivation path are both inactive. Identity has to come from a complete `~/.multisphere/identity.json` (with explicit `agent_id`) or from env vars set by the user's shell. See §5.

> ⚠️ Discrepancy to flag: `CLAUDE.md` currently says the `.mcp.json` "Sets `MULTISPHERE_CLIENT=claude-code`". The shipped file does not. Either CLAUDE.md is stale, or the env block needs to be added back. Writers should describe what the file does, not what the doc claims.

The plugin uses `${CLAUDE_PLUGIN_ROOT}` as the only safe way to reference files inside the plugin from `.mcp.json` — but the published version doesn't need it because it resolves the server via `npx`, not via a local path.

## 4. MCPB manifest — `manifest.json`

For MCPB-only hosts. The same server, but bundled locally and with explicit env wiring.

Key blocks:

- `server.entry_point` → `mcp-server/dist/index.js` (must exist in the `.mcpb`; built by `scripts/build-mcpb.sh`).
- `server.mcp_config.command` → `node`; `args` → `${__dirname}/mcp-server/dist/index.js`.
- `server.mcp_config.env`:
  - `MULTISPHERE_AGENT_ID/_NAME/_EMAIL` ← `${user_config.*}` (filled by the host's install dialog).
  - `MULTISPHERE_CLIENT` ← `"cowork"`, hard-coded.
  - `PATH` ← explicit; required so `simple-git` can find the system `git` binary. `platform_overrides` extend it per OS (darwin adds `/opt/homebrew/bin`; win32 uses Git-for-Windows paths).
- `user_config` — the install-time dialog: agent_id, agent_name, agent_email. All `required: true`.
- `tools` — a static manifest listing the 20 MCP tools the server exposes. Used by hosts that want to show a tool inventory before install. Must be kept in sync with `src/index.ts` if you add/rename a tool.

## 5. `resolveIdentity()` — precedence chain

Defined in `mcp-server/src/config.ts`. Called on every server startup; result is bundled with workspace state into `MultisphereConfig`.

Precedence, in order:

1. **Env vars** — `MULTISPHERE_AGENT_ID` + `MULTISPHERE_AGENT_NAME` + `MULTISPHERE_AGENT_EMAIL`. All three must be present. Returns immediately. Identity sourced from env is **never persisted to disk** — that's deliberate, so MCPB hosts (and tests, and `direnv`) don't leak identity into the per-client files. Wins on the MCPB/Cowork path because the manifest fills these in.
2. **Per-client identity file** — if `MULTISPHERE_CLIENT` is set, read `~/.multisphere/identity.<client>.json`. Must have `agent_id`, `agent_name`, `agent_email`. Lets one human run multiple clients (Claude Code, Cowork, Claude Desktop) on the same machine without identity collisions.
3. **Default identity file** — `~/.multisphere/identity.json`, tried two ways:
   - If it has `agent_id` + `agent_name` + `agent_email`, use them directly.
   - Otherwise, if it has `user_slug` + `agent_name` + `agent_email` *and* `MULTISPHERE_CLIENT` is set, derive `agent_id = <user_slug>-<client>` (e.g. `jamie` + `claude-code` → `jamie-claude-code`). Useful when a human wants the same `user_slug` across all clients and only varies the client suffix.
4. **Legacy** — `~/.multisphere/config.json`, read-only.
5. **Empty** — `assertIdentity()` will then throw with a setup hint tailored to whether `MULTISPHERE_CLIENT` is set.

**What this means in practice today on the Claude Code path** (no env, no client): only paths 3-direct or 4 fire. The user needs a `~/.multisphere/identity.json` with a complete `{agent_id, agent_name, agent_email}` triple — or they need to set the env vars in their shell. The auto-derivation path (3b) cannot activate.

**Storage rule worth repeating**: `saveConfig()` writes only `workspaces.json`. It never touches any identity file. This is what keeps one client from clobbering another's identity. Documented inline in `config.ts:139–149`.

## 6. Tool registration and how tools surface in the client

Inside the server (`src/index.ts`):

- Server constructor: `new Server({ name: 'multisphere', version: VERSION }, ...)`.
- Transport: `StdioServerTransport`. Stdout is reserved for MCP framing; the server logs only to stderr (the boot banner is `multisphere-mcp <version> listening on stdio\n` on stderr).
- Each tool is registered via `server.tool(name, description, zodSchema, wrap(handler))`. The `wrap()` helper catches throws and returns `{error: ...}` payloads so failures come back as structured data rather than crashing the JSON-RPC stream.
- 20 tools, grouped by module: workspace lifecycle (`workspace.ts`), git ops (`git-ops.ts`), filesystem (`fs-ops.ts`), protocol (`protocol.ts`).

How the client sees them depends on install path:

- **Plugin install (Claude Code, Cowork-as-plugin):** prefix is `mcp__plugin_<plugin-name>_<server-key>__<tool>`. So `read` surfaces as `mcp__plugin_multisphere_workspace__read`. The `workspace` segment comes from the `.mcp.json` server key — **not** from the server's internal `name: 'multisphere'`. That distinction is the reason the key is `workspace`: it controls the user-visible prefix.
- **MCPB install:** prefix is `mcp__<server-key>__<tool>`, no `plugin_` segment. In Cowork-via-MCPB the server is also keyed `workspace` (via the manifest's wiring), so tools surface as `mcp__workspace__<tool>`.

A user moving between the two paths will see different prefixes for the same tools. Docs should pick one and stick to it (the plugin path is the default to document).

## 7. Things a writer will get wrong if they only read the overview

Concrete pitfalls — call these out in the install/concept pages:

- **Claiming `.mcp.json` sets `MULTISPHERE_CLIENT=claude-code`.** It does not (see §3). Until that's fixed in the repo, per-client identity files and `user_slug` auto-derivation are inactive on the plugin path.
- **Treating the plugin install as "one install, both clients including the MCPB bundle".** No: the plugin path and the MCPB path are separate artifacts. Plugin = `.claude-plugin/` + `.mcp.json` + skill, server via npm. MCPB = `.mcpb` archive carrying a frozen server, no skill.
- **Calling the MCP server "multisphere" in tool prefixes.** The server's internal `name` is `multisphere`, but the user-facing tool prefix carries the `.mcp.json` *key* — `workspace`. Mixing them up will mislead anyone who tries to whitelist or reference the tools by name.
- **Suggesting users put identity in `~/.multisphere/config.json`.** That's the legacy path. New setups should write `~/.multisphere/identity.json` (single client) or `~/.multisphere/identity.<client>.json` (multi-client). `saveConfig()` deliberately won't write to either of these — identity is hand-edited or set via env.
- **Assuming git is on PATH everywhere.** On the MCPB path the manifest pins `PATH` per OS so `simple-git` can find `git`. On the plugin path the server inherits the user's shell PATH — if the user's shell can't find `git`, neither can the server.
- **Conflating plugin version with MCP server version.** Plugin updates need a `plugin.json` bump. MCP server updates need a `mcp-server/package.json` bump + `npm publish` — existing installs pick them up via `npx@latest` on next launch without any plugin action. Recommended practice in this repo: bump both in lockstep.
- **Writing about "the multisphere MCP" without naming the npm package.** It's `multisphere-mcp`. The plugin name is `multisphere`. Two different identifiers in two different registries.

## 8. Quick reference — env vars the server reads

| Var | Read in | Effect |
|---|---|---|
| `MULTISPHERE_AGENT_ID` | `resolveIdentity()` | Step 1 of precedence (with `_NAME` + `_EMAIL`). Never persisted. |
| `MULTISPHERE_AGENT_NAME` | `resolveIdentity()` | Same. |
| `MULTISPHERE_AGENT_EMAIL` | `resolveIdentity()` | Same. |
| `MULTISPHERE_CLIENT` | `resolveIdentity()`, `assertIdentity()` | Selects per-client identity file; enables `user_slug + client` derivation; tailors the setup-error hint. |

That's the whole env surface.

## 9. Quick reference — files in `~/.multisphere/`

| File | Written by server? | Purpose |
|---|---|---|
| `identity.json` | no (hand-edited or installer-written) | Default identity. Either `{agent_id, agent_name, agent_email}` direct, or `{user_slug, agent_name, agent_email}` for auto-derivation with `MULTISPHERE_CLIENT`. |
| `identity.<client>.json` | no | Per-client override. Selected when `MULTISPHERE_CLIENT` matches. |
| `workspaces.json` | yes, via `saveConfig()` | Workspace registry: `{ workspaces: {…}, active_workspace: "…" }`. Owned by the server. |
| `config.json` | no (legacy, read-only) | Old single-file config. Still read as a fallback for both identity and workspaces. |
