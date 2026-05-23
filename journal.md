# Journal

## 2026-05-23 10:43 — jamie-claude-code (Jamie)
Bootstrapped the multisphere-docs workspace; dropped a technical overview in research/ as the source-of-truth brief for the writers.

- Customized README.md with the project purpose (user-facing docs for the Multisphere plugin) and the members table.
- Wrote research/technical-overview.md covering: what Multisphere is, install commands, identity & client-aware config, workspace layout, on-entry/on-exit protocol, file formats (journal, inbox, decisions), MCP tool inventory, architecture facts worth surfacing in user docs, and pointers to the canonical source-repo docs.
- Opened INB item pointing the writers at the research drop.

TODO @anyone: Read research/technical-overview.md before drafting any user-facing copy
TODO @anyone: Decide doc structure (single getting-started page vs. multi-page guide) and drop a decision in decisions/

## 2026-05-23 11:36 —  ()
Picked a five-page doc structure and dropped first drafts of all five pages.

Decision in decisions/doc-structure.md: concept → install → getting-started → protocol reference → tool reference. Five files in drafts/ numbered 01–05. Wrote in a terse style, source pulled from research/technical-overview.md. Closed INB-001 since the writing pass it asked for is now underway.

TODO @anyone: Edit pass on 01-concept and 03-getting-started — these set the tone for the rest
TODO @jamie-claude-code: Confirm the install commands and identity filenames against the live plugin before publishing
TODO @anyone: Decide where the docs eventually live (in-repo /docs, separate site, plugin readme) and drop a second decision

## 2026-05-23 11:41 —  ()
Asked jamie-claude-code for an MCP setup deep-dive in research/.

Opened INB-002 — the current technical overview is enough for surface copy but not enough to write the install page with confidence. Need depth on manifest wiring, identity resolution, tool registration, and Claude Code vs Cowork differences at the MCP layer.

## 2026-05-23 11:46 —  ()
Wrote MCP setup deep-dive in research/ to fulfill INB-002.

research/mcp-setup-deep-dive.md covers: the two install paths (plugin vs MCPB), the plugin.json + .mcp.json manifests, the MCPB manifest.json, the resolveIdentity() precedence chain in mcp-server/src/config.ts, tool registration and how prefixes differ between install paths, env var + ~/.multisphere/ file references, and a "things a writer will get wrong" punch list.

Grounded in the source: .claude-plugin/plugin.json, .mcp.json, manifest.json, mcp-server/src/config.ts, mcp-server/src/index.ts, and CLAUDE.md.

Flagged one repo-side discrepancy worth fixing: CLAUDE.md says .mcp.json "Sets MULTISPHERE_CLIENT=claude-code" but the shipped file has no env block. Either CLAUDE.md is stale or the env needs to go back into .mcp.json. Per-client identity files and the user_slug+client auto-derivation path are inactive on the Claude Code install path until that's resolved.

INB-002 closed.

TODO @anyone: Edit pass on drafts/01-concept and drafts/03-getting-started (still outstanding from previous journal)
TODO @anyone: Decide where the docs eventually live (in-repo /docs vs separate site vs plugin README) and drop a second decision file
TODO @jamie-claude-code: Fix MULTISPHERE_CLIENT discrepancy on the multisphere repo side — either restore the env block in .mcp.json or update CLAUDE.md to match

