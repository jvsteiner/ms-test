# Five-page multi-page structure for the user docs

Date: 2026-05-23
Decided by: jamie-cowork
Status: accepted

## Context

We need user-facing docs for the Multisphere plugin. The technical brief in `research/technical-overview.md` covers five distinct subjects: what Multisphere is, how to install it, how to use it the first time, the protocol agents follow, and the MCP tools available. One long page would force the reader to scroll past concept material to reach reference material, and hide the entry-level pages under everything else.

## Decision

Five pages, in this order:

1. `01-concept.md` — what Multisphere is and why it exists.
2. `02-install.md` — marketplace commands, identity config, client support.
3. `03-getting-started.md` — first workspace, first session, first commit.
4. `04-protocol-reference.md` — entry sequence, exit sequence, file formats.
5. `05-tool-reference.md` — MCP tools grouped by purpose.

A reader who only reads page 1 should walk away knowing what Multisphere is and why it exists. A reader who reads pages 1 through 3 should be able to join a workspace and do useful work. Pages 4 and 5 are for the reader who wants the full surface.

## Consequences

First drafts land in `drafts/` with the numbered filenames above. Concept and getting-started get the first edit pass; the two reference pages will mostly mirror the technical brief and need lighter editing.

If the doc set grows beyond five pages — for example, an FAQ or a troubleshooting page — we add files but keep the numeric ordering of the core five.
