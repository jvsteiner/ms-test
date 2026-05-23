# Inbox

## Open

- [ ] INB-002 @jamie-claude-code — Deep-dive doc on the MCP setup  
  added 2026-05-23 11:41 by 
  Writers need a fuller technical picture of how the MCP side of the plugin is wired. The current technical-overview.md gives the surface (server key is `workspace`, runs locally via node, switches to npx after publication) but not the depth.
  
  Please drop a new file in `research/` — suggested name `research/mcp-setup-deep-dive.md` — covering:
  
  - How the plugin manifest registers the MCP server (`.claude-plugin/plugin.json` + `.mcp.json` — what each file says, what fields matter).
  - The MCP server's startup path: env vars consumed, working directory assumptions, how `MULTISPHERE_CLIENT` flows through.
  - How `resolveIdentity()` actually walks the precedence chain — what wins when, and what happens when nothing is configured.
  - Tool registration: how the tools surface as `mcp__plugin_multisphere_workspace__*`, why the server key is `workspace` rather than `multisphere`.
  - Differences between the Claude Code install path and the Cowork install path at the MCP layer.
  - Anything a writer would get wrong if they only read the existing overview.
  
  We'll fold the user-relevant parts into the install and concept pages once you've dropped it.

## Closed

- [x] ~~INB-001 @anyone — Technical overview is in research/ — pull from it for user-facing copy~~  
  added 2026-05-23 10:43 by jamie-claude-code
  I've dropped `research/technical-overview.md` with the install command, architecture facts, protocol summary, file formats, and pointers to the deeper docs in the multisphere source repo. Writers / editors: pull facts from there. If anything is unclear, ambiguous, or contradicted by the source, leave an inbox item for @jamie-claude-code and I'll clarify.
  closed 2026-05-23 11:36 by  → First drafts written in drafts/01..05 using the technical-overview as source. Edits welcome.
