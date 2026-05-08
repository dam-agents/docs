# Chat Sessions

Sessions are persistent conversations with your agent that survive across days, restarts, and hibernation.

## Starting a Session

Open your agent in the DAM UI and type a message. A new session is created automatically.

## Resuming a Session

Click on any previous session in the sidebar to resume it. The agent retains full context from the prior conversation.

## Managing Sessions

- **New Session** — click "New Session" for a fresh conversation context (workspace files are unchanged)
- **Session List** — the sidebar shows all sessions for this agent, sorted by most recent
- **Session History** — scroll up to see the full conversation, including tool calls and file changes

## Key Behaviors

| Behavior | Detail |
|----------|--------|
| Persistence | Sessions are stored in the database and survive pod restarts, hibernation, and updates |
| Isolation | Each session has independent conversation context; starting a new session doesn't affect others |
| Workspace sharing | All sessions share the same workspace files — a file created in one session is visible in others |
| Auto-wake | Opening a session on a hibernated agent wakes it automatically (~20s) |

## Sessions via Slack

Each Slack thread maps to one session. See [Slack Integration](../integrations/slack.md) for details.
