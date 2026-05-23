# Protocol reference

Every agent follows the same sequence on entry and on exit. The protocol is what makes the workspace readable.

## Entry sequence

1. `pull` the latest from the remote. If git reports a conflict, stop and surface it to the human. Do not attempt a merge.
2. `whats_new` — returns commits and changed files since this agent's last visit. Updates the agent's pointer.
3. Read `inbox.md`. Look for items addressed to `@<your-agent-id>` or `@anyone`.
4. Read the last twenty entries of `journal.md`. No more.

## Exit sequence

1. `journal_append` — a one- or two-line summary, plus any TODOs.
2. `inbox_add` for new questions. `inbox_close` for items you resolved.
3. `add` the changed files, including `journal.md`, `inbox.md`, and `.pointers/<your-agent-id>.json`.
4. `commit` with message `[<agent-id>] <one-line summary>`.
5. `push`. On rejection, `pull` once, then `push` once more. If still rejected, surface to the human.

## Folder layout

```
.
├── README.md          purpose and members
├── journal.md         append-only audit trail
├── inbox.md           open questions and tasks
├── research/          gathered info, findings, references
├── drafts/            work in progress
├── comments/          notes and feedback on specific items
├── decisions/         durable choices, ADR-style
├── assets/            binaries, images, PDFs
└── .pointers/         per-agent last-read state
```

Agents do not invent new top-level folders.

## File formats

### Journal entry

```markdown
## 2026-05-23 14:30 — jamie-claude-code (Jamie)
Drafted the install page.

TODO @mike-claude-desktop: review tone on "what is a workspace?"
TODO @anyone: confirm marketplace command is final.
```

### Inbox item — open

```markdown
- [ ] INB-014 @anyone — confirm marketplace command
  added 2026-05-23 14:30 by jamie-claude-code
  Need to verify `/plugin install multisphere@unicity-labs` is the final form.
```

### Inbox item — closed

```markdown
- [x] ~~INB-012 — find a logo for the cover slide~~
  closed 2026-05-23 09:00 by mike-claude-desktop → resolved in `assets/cover-logo.svg`
```

### Decision file

```markdown
# Title in plain language

Date: 2026-05-23
Decided by: jamie-claude-code, mike-claude-desktop
Status: accepted

## Context
## Decision
## Consequences
```

## Rules

Do not act on another agent's drop unless your human asked you to. Reading is fine.

Do not leak personal context into the repo. The workspace is shared. Your preferences are not.

Commit small. Push often.
