# Sessions

Sessions are persistent conversations with your agent. DAM supports two session modes: chat and terminal.

## Chat Sessions

Open your agent in the DAM UI and type a message. A new session is created automatically.

- **Resume** — click any previous session in the sidebar to continue
- **New Session** — start fresh conversation context (workspace files are unchanged)
- **History** — scroll up to see full conversation including tool calls and file changes
- **Slack threads** — each thread maps to one session (see [Slack](../integrations/slack.md))

Sessions are stored as files on the agent's persistent volume (PVC) — not in a central database — and survive pod restarts, hibernation, and updates. All sessions share the same workspace files.

### Status Indicators

Each session in the sidebar shows a status dot so you can tell at a glance which
sessions need you and which are working on their own:

| Indicator | Meaning |
|-----------|---------|
| **Working** | The agent is actively working the turn. |
| **Needs your approval** | The agent is blocked waiting for you to approve an action. This takes priority over any other state. |
| **Waiting for you** | The agent is idle, waiting for your next message. |
| **Terminal** | A terminal (PTY) session, which has no agent turn of its own. |

### Environment variables

Environment variables you set in the agent's Environment editor are persisted separately and delivered to the running agent over the runtime channel, applied on its **next turn** — editing env does not restart the pod. User-typed env shadows env that comes from connections or secrets when the names collide. Because the change lands on the next turn rather than at startup, an in-flight turn keeps the values it began with.

### Model settings

The **Model** settings panel sets per-agent harness defaults — model, mode, and reasoning effort. These are written to the harness's own config file and apply to **new sessions only**; sessions already underway keep the settings they started with.

## Terminal Sessions

Terminal sessions give you PTY-based shell access to the agent's environment — the same interactive TUI you'd get running the harness locally.

Use terminal for:

- Running commands directly (build, test, deploy)
- Interactive debugging with the harness TUI
- Operations that are easier in a shell than in chat

| Behavior | Detail |
|----------|--------|
| Single viewer | One viewer at a time per terminal session |
| Workspace access | Same filesystem as chat sessions (`/home/agent/work`) |
| Scrollback | Full xterm buffer preserved during the session |

### Reconnection

If you lose connection (close tab, network drop):

- **~30 second grace window** to reconnect with session intact
- After 30s, the PTY process terminates — start a new terminal session
- Workspace files are always unaffected

### Terminal via CLI

```bash
dam chat my-agent              # New terminal session
dam chat my-agent --continue   # Reattach to most recent
dam chat my-agent --resume <id> # Resume specific session
```

## When to Use Which

| Terminal | Chat |
|----------|------|
| Running a specific command | Asking the agent to figure out what to do |
| Interactive debugging | Delegating a multi-step task |
| Need raw shell access | Wanting explanations alongside actions |
