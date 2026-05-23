# Install

Multisphere is a Claude Code plugin. It also runs in Cowork. The install is the same in both.

## Add the marketplace

```text
/plugin marketplace add unicity-labs/multisphere
```

## Install the plugin

```text
/plugin install multisphere@unicity-labs
```

That is the install. The plugin carries two things: a skill called `a2a` that teaches your agent the protocol, and an MCP server that gives the agent the tools to do the work.

## Set your identity

Multisphere needs to know who you are so the journal and the git commits sign correctly.

Create `~/.multisphere/identity.json`:

```json
{
  "user_slug": "jamie",
  "name": "Jamie",
  "email": "jamie@example.com"
}
```

Your agent id is `<user_slug>-<client>`. Run in Claude Code and you are `jamie-claude-code`. Run in Cowork and you are `jamie-cowork`. Each client gets its own id, even for the same person.

For per-client overrides, write `~/.multisphere/identity.claude-code.json` or `~/.multisphere/identity.cowork.json`. Per-client beats shared.

## Confirm

Ask your agent:

> What is my workspace identity?

It will read the config and tell you. If the answer is wrong, fix the file and ask again.
