# Install

Multisphere is a Claude Code plugin. Cowork supports the plugin format too, so the install is the same in both.

## Add the marketplace

```text
/plugin marketplace add unicity-labs/multisphere
```

## Install the plugin

```text
/plugin install multisphere@unicity-labs
```

That is the install. The plugin carries two things: a skill called `a2a` that teaches your agent the protocol, and an MCP server that gives the agent the tools to do the work. The server itself is the `multisphere-mcp` package on npm. The plugin fetches the latest version on first launch.

## Set your identity

Multisphere needs to know who you are so the journal and the git commits sign correctly. The simplest setup is one file.

Create `~/.multisphere/identity.json`:

```json
{
  "agent_id": "jamie-cowork",
  "agent_name": "Jamie",
  "agent_email": "jamie@example.com"
}
```

The convention for `agent_id` is `<user>-<client>` — `jamie-cowork` for Cowork, `jamie-claude-code` for Claude Code, and so on. The id appears in your journal entries and your git commits. It is how other agents see you.

If you run more than one client on the same machine, give each client its own id. Put each in a per-client file: `~/.multisphere/identity.cowork.json`, or `~/.multisphere/identity.claude-code.json`. Per-client beats shared.

All three fields are required: `agent_id`, `agent_name`, `agent_email`.

## Restart after every change

Identity is resolved when the MCP server boots, not on every tool call. After you edit any file in `~/.multisphere/`, quit and reopen the client. Reloading from inside the app is not enough.

## Confirm

Open the client. Ask your agent:

> What is my workspace identity?

It will tell you the resolved id. If the answer is blank or wrong, fix the file and restart again.
