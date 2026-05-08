# MCP Tools

Your agent exposes a set of built-in tools via MCP (Model Context Protocol) that enable programmatic interaction with the DAM platform.

## Available Tools

| Tool | Description |
|------|-------------|
| `list_skill_sources` | List all configured skill sources (platform, template, user) |
| `list_skills_in_source` | List available skills within a specific source |
| `install_skill` | Install a skill from a source onto the agent |
| `uninstall_skill` | Remove an installed skill |
| `publish_skill` | Publish a local skill to a git-based source |
| `send_channel_message` | Send a message to a connected channel (Slack, Telegram) |
| `describe_channel` | Get info about a connected channel |

## How It Works

MCP tools are exposed as a local endpoint within your agent's pod. The agent harness (Claude Code, Pi Agent, etc.) can invoke these tools as part of normal operation — for example, installing a skill mid-conversation or posting a notification to Slack.

## Use Cases

- **Self-installing skills** — an agent can discover and install skills it needs to complete a task
- **Proactive notifications** — agents can message your Slack channel when long-running tasks complete
- **Skill publishing** — after developing a skill locally, the agent can publish it to a shared source

## Example

In a chat session, you can ask your agent:

```
List all available skills from the platform source and install
the "review" skill.
```

The agent uses `list_skills_in_source` and `install_skill` behind the scenes.

!!! info "No configuration required"
    MCP tools are available automatically on every agent instance. You don't need to enable or configure them.
