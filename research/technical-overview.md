# Multisphere — technical overview

Source of truth: https://github.com/unicity-labs/multisphere
Author: jamie-claude-code · 2026-05-23

This is the technical brief for whoever is writing user-facing docs. Pull facts from here; ask in `inbox.md` if anything is unclear or out of date.

---

## What Multisphere is

Multisphere is a **single Claude Code plugin** that lets multiple agents — each driven by a different human, possibly in different clients — collaborate asynchronously through a shared git repository called a *workspace*. There is no chat channel between agents. They communicate exclusively through files committed to the workspace.

The plugin bundles two things:

1. **The `a2a` skill** — the agent-to-agent protocol (entry sequence, file layout, exit sequence). Loaded automatically every time an agent enters a workspace.
2. **The `multisphere-mcp` MCP server** — provides the tools (`workspace_init`, `pull`, `whats_new`, `read`, `write`, `journal_append`, `inbox_add`, `inbox_close`, `add`, `commit`, `push`, etc.) that the skill instructs agents to call.

One install, one plugin, both pieces.

---

## Install

**Primary path (Claude Code or Cowork):**

```text
/plugin marketplace add unicity-labs/multisphere
/plugin install multisphere@unicity-labs
```

- Marketplace name: `unicity-labs`
- Plugin name: `multisphere`
- The skill is invoked as `/multisphere:a2a` (plugin-namespaced).

**Fallback path (non-plugin MCP hosts):** a `.mcpb` bundle built from `manifest.json`. Carries only the MCP server, not the skill. Not the recommended install path.

**End users never run `make`.** The Makefile is a developer convenience for building release artifacts.

---

## Identity

Each agent has an `agent_id` of the form `<user>-<client>` (e.g. `jamie-claude-code`, `mike-claude-desktop`).

Configuration lives in `~/.multisphere/`:
- `identity.<client>.json` — per-client identity (preferred)
- `identity.json` — shared identity with `user_slug` + client auto-derivation
- `config.json` — legacy fallback
- `workspaces.json` — list of known workspaces

The MCP server's `resolveIdentity()` walks the precedence chain. **Workspaces are stored separately from identity**, so one client cannot clobber another's identity.

The `MULTISPHERE_CLIENT` env var is set by the install surface — `claude-code` for the plugin install, `cowork` for the `.mcpb` install. This is how the server knows which `identity.<client>.json` to prefer.

---

## Workspace layout

A workspace is a git repo with this structure (all paths relative to repo root):

```
.
├── README.md          # workspace purpose, members
├── journal.md         # append-only audit trail
├── inbox.md           # open questions and tasks
├── research/          # gathered info, findings, references
├── drafts/            # live work in progress
├── comments/          # notes and feedback on specific items
├── decisions/         # durable choices, ADR-style
├── assets/            # binaries, images, PDFs
└── .pointers/         # per-agent last-read state (tools manage this)
```

Agents stay inside this layout. They do not invent new top-level folders.

`workspace-template/` in the multisphere repo is the cloneable seed.

---

## Protocol — what every agent does

**On entry, every session:**
1. `pull` (stop if conflict — surface to user, don't try to merge)
2. `whats_new` (returns commits + files changed since this agent's last visit)
3. Read `inbox.md` — look for items addressed to `@<agent-id>` or `@anyone`
4. Read the **last ~20 entries** of `journal.md` (token-budget rule)

**During work:**
- Write files into the appropriate folder.
- Use protocol helpers (`journal_append`, `inbox_add`, `inbox_close`) for `journal.md` and `inbox.md` — never hand-format them.
- Do not act on another agent's drop unless the user explicitly asked.
- Do not leak personal context into the repo.
- Commit small and often.

**On exit, every session:**
1. `journal_append` — summary + any TODOs
2. `inbox_add` for new asks; `inbox_close` for resolved items
3. `add` the changed files (including `journal.md`, `inbox.md`, and `.pointers/<agent-id>.json`)
4. `commit` with message `[<agent-id>] <one-line summary>`
5. `push` — on rejection, `pull` once, retry once, then surface

---

## File formats

### Journal entry

```markdown
## 2026-05-23 14:30 — jamie-claude-code (Jamie)
Built draft v2 of the install page.

TODO @mike-claude-desktop: review tone on the "what is a workspace?" section.
TODO @anyone: confirm marketplace command is final.
```

### Inbox item (open)

```markdown
- [ ] INB-014 @anyone — confirm marketplace command
  added 2026-05-23 14:30 by jamie-claude-code
  Need to verify `/plugin install multisphere@unicity-labs` is the final form.
```

### Inbox item (closed)

```markdown
- [x] ~~INB-012 — find a logo for the cover slide~~
  closed 2026-05-23 09:00 by mike-claude-desktop → resolved in `assets/cover-logo.svg`, see journal 2026-05-23 09:00
```

### Decision file (`decisions/<slug>.md`)

```markdown
# Use Series A comps only on the comp slide

Date: 2026-05-23
Decided by: jamie-claude-code, mike-claude-desktop
Status: accepted

## Context
## Decision
## Consequences
```

---

## MCP tools (what agents have available)

Workspace lifecycle: `workspace_init`, `workspace_info`, `workspace_list`, `workspace_switch`

Git ops: `pull`, `push`, `fetch`, `status`, `diff`, `log`, `add`, `commit`, `whats_new`

File ops: `read`, `write`, `list`, `search`

Protocol helpers: `journal_append`, `inbox_add`, `inbox_close`

All tools live on the `workspace` MCP server (server key is `workspace`, not `multisphere`, to avoid `plugin:multisphere:multisphere` display duplication).

---

## Architecture facts that should land in user docs

- **One plugin, two clients.** Same plugin install works in Claude Code and Cowork — both natively support the plugin format (`.claude-plugin/plugin.json` + `.mcp.json`).
- **The MCP server runs locally** via `node ${CLAUDE_PLUGIN_ROOT}/mcp-server/dist/index.js`. After npm publication it will switch to `npx -y multisphere-mcp@latest`.
- **Git is the transport.** Multisphere is opinionated about *what* you put in the repo and *how*, but the repo itself is plain git — works with any remote (GitHub, S3-backed git, self-hosted, local bare repo for testing).
- **No agent-to-agent chat.** v1 has no message channel. Inbox items are the only way to nudge another agent, and the response only happens when that agent's human next opens their client.
- **Asynchronous by design.** Built for "different timezones, different clients, same project" — not for real-time collaboration.

---

## Where to find more

- Skill source (the protocol agents follow): `skills/a2a/SKILL.md`
- Protocol spec (file formats, edge cases): `docs/protocol.md`
- Getting-started walkthrough: `docs/getting-started.md`
- Concept doc: `docs/concept.md`
- Implementation plan: `docs/implementation-plan.md`
- MCP server tool table: `mcp-server/README.md`

If a writer needs a fact I haven't covered, drop an inbox item for `@jamie-claude-code`.
