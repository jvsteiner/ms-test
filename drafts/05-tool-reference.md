# Tool reference

All tools live on the `workspace` MCP server, installed by the Multisphere plugin.

## Workspace lifecycle

- `workspace_init` — clone a remote workspace and make it active.
- `workspace_info` — name, path, remote, HEAD, branch.
- `workspace_list` — all configured workspaces.
- `workspace_switch` — change the active workspace.

## Git operations

- `pull` — fast-forward only. Returns `{error: "conflict"}` if a merge is needed.
- `push` — push to remote. Returns `{error: "rejected"}` on non-fast-forward.
- `fetch` — refresh remote refs.
- `status` — working-tree status.
- `diff` — unified diff. Working tree by default; pass `since` for a ref.
- `log` — commit log.
- `add` — stage paths.
- `commit` — create a commit. Message format: `[<agent-id>] <summary>`.
- `whats_new` — commits and files since this agent's last visit. Updates the pointer.

## File operations

- `read` — read a UTF-8 file.
- `write` — write or append a UTF-8 file. Pass `mode: "append"` to append.
- `list` — directory entries.
- `search` — substring search across text files.

## Protocol helpers

- `journal_append` — add an entry to `journal.md`. Pass `summary` and optional `details` and `todos`.
- `inbox_add` — open an inbox item. Returns the assigned `INB-NNN` id.
- `inbox_close` — close an item by id. Pass the resolution.

Use the helpers for `journal.md` and `inbox.md`. Do not hand-format them.
