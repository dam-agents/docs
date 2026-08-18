# Sessions

Sessions are persistent conversations with your agent. A session runs in one of two modes — **chat**
or **terminal**. Resuming a chat session from the CLI's terminal switches its mode — see
[Terminal via CLI](#terminal-via-cli).

**SSH** is a third way in, but it isn't a session: it has no history, no entry in the sessions list,
and no harness involvement — just a login shell. See the
[VS Code/Zed (SSH) guide](../guides/vs-code-zed-ssh.md).

## Chat Sessions

Open your sandbox in the DAM UI and type a message. A new session is created automatically.

- **Resume** — click any previous session in the sidebar to continue
- **New chat session** — from the **New** menu; starts fresh conversation context (workspace files
  are unchanged)
- **History** — scroll up to see full conversation including tool calls and file changes
- **Slack threads** — each thread maps to one session (see [Slack](../integrations/slack.md))

Sessions live with the sandbox's own files, so they survive hibernation, restarts, and platform updates. All sessions share the same workspace files.

The sessions list is ordered by last activity: each session tracks the time of its most recent message, and that timestamp is what the list sorts and displays by — not the underlying harness file's modification time. (Terminal sessions carry no unread state — their timestamp is bumped by restarts and TUI repaints, not by you reading them.)

### Status Indicators

Sessions in the sidebar carry markers so you can tell at a glance which need you and which are
working on their own:

| Indicator | Meaning |
|-----------|---------|
| Animated dots | The agent is actively working the turn. |
| Solid dot ("Needs your approval") | Blocked waiting for you to approve an action. Takes priority over working. |
| Bold title | Unread — activity since you last opened it (chat sessions only). |
| Terminal icon | A terminal (PTY) session, which has no agent turn of its own. |
| Clock icon | A session created by a schedule. |
| `#` icon | A channel (Slack/Telegram) session; a superscript **A** marks a channel's ambient reader. |

If the agent asks permission for a tool call and **no session is attached** to receive the prompt, the
request expires after ten minutes and the agent is told it failed, so the tool call aborts cleanly.
That's separate from the longer window on
[network approvals](network-access.md#approvals), which keep waiting for you in the Inbox.

An idle chat session waiting on you shows no marker at all. The sidebar also has a filter over
**Chats**, **Experiment runs**, **Scheduled**, **Channels**, and **Terminal** (all on by default),
and a **New** menu offering a new chat or terminal session. Deleting a session is in its row menu,
behind a confirm.

### Environment variables

Environment variables you set in the sandbox's Environment editor are persisted separately and delivered to the running sandbox, applied on the agent's **next turn** — editing env does not restart the pod. User-typed env shadows env that comes from connections when the names collide.

Whether a turn already in flight is disturbed depends on what you changed. **Adding or removing a variable** interrupts a turn still running about a minute after you save, so the new set can take effect. **Changing only a value** never interrupts a running turn — it applies at the next turn boundary.

### Model settings

The **Model** settings panel sets per-sandbox harness defaults. These are written to the harness's own config file and apply to **new sessions only**; sessions already underway keep the settings they started with.

The panel lives on the sandbox's home page, under **Sandbox Setup**; chat shows the model in use and a gear that opens that page. Which settings appear, and what they're called, depends on the harness — Claude Code offers a model, a permission mode, and a reasoning-effort level; others offer fewer, and some expose no model settings at all — and some choices are hidden for models that don't support them. While the sandbox is asleep the panel is read-only, showing the last-known values with a **Start agent to edit** action.

## Terminal Sessions

Terminal sessions give you PTY-based shell access to the agent's environment — the same interactive TUI you'd get running the harness locally.

Use terminal for:

- Running commands directly (build, test, deploy)
- Interactive debugging with the harness TUI
- Operations that are easier in a shell than in chat

| Behavior | Detail |
|----------|--------|
| Single viewer | One viewer at a time per terminal session — no fan-out, unlike chat |
| Workspace access | Same filesystem as chat sessions (`/home/agent/work`) |
| Scrollback | The last ~1,000 lines, replayed when you reattach to a live terminal |

### Reconnection

If you lose connection (close tab, network drop), the terminal isn't killed just because you left:

- Reattach within **~30 seconds** and you always get the session back, scrollback included.
- Past that, the terminal survives as long as it's still **producing output** — a running build or a
  streaming response keeps it alive, and you can reattach to it live.
- It's reaped only once it's both detached past the grace window **and** quiet for five minutes.
- Workspace files are always unaffected.

### Terminal via CLI

```bash
dam chat my-agent               # New terminal session
dam chat my-agent --continue    # Reattach to your open terminal session
dam chat my-agent --resume <id> # Resume a specific session by id
dam chat my-agent --reset       # Kill the PTY and start it fresh
```

`--continue` needs exactly one open **terminal** session. With none it tells you to start one; with
several it lists their ids and asks you to name one with `--resume`. Resuming a *chat* session in the
terminal switches its mode, so the CLI confirms before it does — declining leaves the session alone.
On disconnect the CLI prints a ready-to-paste resume command.

## When to Use Which

| Terminal | Chat |
|----------|------|
| Running a specific command | Asking the agent to figure out what to do |
| Interactive debugging | Delegating a multi-step task |
| Need raw shell access | Wanting explanations alongside actions |
