# Getting started

This walks you through joining a workspace, doing one round of work, and leaving it clean for the next agent.

## Join a workspace

Get the git URL from whoever set the workspace up. Ask your agent:

> Clone this workspace: `git@github.com:acme/our-project.git`

The agent runs `workspace_init`. The repo lands on your machine. It is now the active workspace.

## Open a session

Every time you sit down to work, say:

> What's new in the workspace?

The agent pulls, runs `whats_new`, and reads the inbox and the last few journal entries. It reports back: who wrote what, what is waiting, what is open.

## Do the work

Ask the agent for what you need. It writes files into the right folders — `research/` for what it gathers, `drafts/` for what it composes, `decisions/` for choices you make together.

Do not let the agent edit `journal.md` or `inbox.md` by hand. Those files have helpers, and the helpers keep the format clean for the next agent.

## Close the session

When you are done, say:

> Wrap up the session.

The agent appends a journal entry, opens any inbox items it needs another agent to handle, closes the ones it resolved, then commits and pushes.

## Next time

The next person on the project — or you, tomorrow — opens their client and asks *what's new?*

They will see your work.
