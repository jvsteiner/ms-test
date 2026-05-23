# What is Multisphere

Multisphere lets several AI agents work on the same project. Each agent belongs to a different person. They are often in different places, working at different times.

The agents do not talk to each other. They write files in a shared git repository — a *workspace* — and read what the others wrote when they show up next. No chat, no messages. Just files in a folder, and a protocol that keeps the folder readable.

## Why it exists

You want a teammate's agent to pick up where yours left off. You went to sleep. They woke up. The work has to wait somewhere, in a form the next agent can find and trust.

Email scatters. Chat rolls past. A shared doc loses its history. Git remembers everything, and lets two people change the same file without losing either change.

Multisphere is git, plus a protocol for what to write and where to write it.

## The mental model

A workspace is a git repo with a fixed layout. Research goes in `research/`. Drafts go in `drafts/`. Decisions go in `decisions/`. One journal records what every agent did. One inbox holds open questions for the others.

When your agent enters the workspace it pulls the latest, reads what's new, then reads the inbox. When it leaves it writes a journal entry, opens or closes inbox items, commits, and pushes.

That is the whole idea. The rest is detail.

## How it's delivered

One Claude Code plugin. Inside it: a skill called `a2a` that teaches your agent the protocol, and an MCP server that gives the agent the tools to do the work. Same install in Claude Code and in Cowork.

The next page covers how to install it.
