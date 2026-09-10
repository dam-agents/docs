# Sessions

Sessions are persistent conversations with your agent. A session runs in one of two modes — **chat**
or **terminal**. Resuming a chat session from the CLI's terminal switches its mode — see
[Terminal via CLI](#terminal-via-cli).

**SSH** is a third way in, but it isn't a session: it has no history, no entry in the sessions list,
and no harness involvement — just a login shell. See the
[VS Code/Zed (SSH) guide](../integrations/vs-code-zed-ssh.md).

## Chat Sessions

Open your agent in the DAM UI and type a message. A new session is created automatically.

- **Resume** — click any previous session in the sidebar to continue
- **New chat session** — **New** in the sidebar; starts fresh conversation context (workspace files
  are unchanged)
- **History** — scroll up to see the conversation including tool calls and file changes. A session
  opens on its most recent stretch and loads older parts as you scroll back. A very long session's
  history is eventually trimmed from the front; where that's happened, the top of the conversation
  says so instead of pretending it's the beginning
- **Slack threads** — each thread maps to one session (see [Slack](../integrations/slack.md))

Sessions live with the agent's own files, so they survive hibernation, restarts, and platform updates. All sessions share the same workspace files.

A chat session has its **own URL**, so the browser's back and forward buttons move between the sessions
you've opened, and a link to one can be bookmarked or shared with anyone who can open that agent.
Terminal sessions aren't addressed this way — the URL stays on the agent.

If a link can't be opened, the conversation says which case it is rather than failing blankly: it may
have been deleted or belong to someone else; the agent may still list it while no longer holding its
history, in which case **Delete this session** clears the leftover entry; or the agent simply didn't
answer, which **Try again** retries once it's awake.

The sessions list is ordered by last activity. A **chat** session tracks the time of its most recent message, and that timestamp is what the list sorts and displays by — not the underlying harness file's modification time. A **terminal** session records no message activity, so its timestamp is the harness file's modification time, which a restart or a redrawing full-screen program can bump without you having typed anything. (Terminal sessions carry no unread state at all either — a terminal row never goes bold, however long you leave it.)

### Status Indicators

Sessions in the sidebar carry markers so you can tell at a glance which need you and which are
working on their own:

| Indicator | Meaning |
|-----------|---------|
| Animated dots | Busy. On a chat session, the agent is working the turn; on a terminal session, the program in it produced output in the last second or so. |
| Slower green dots | The agent has **background work** running in that session — something it kicked off and left going between turns. Hover for what's running. |
| Solid dot ("Needs your approval") | Blocked waiting for you to approve an action. Takes priority over working. |
| Bold title | Unread — activity since you last opened it (chat sessions only). |
| Pencil icon | You have an unsent **draft** in that session. |
| Terminal icon | A terminal (PTY) session, which has no agent turn of its own. |
| Clock icon | A session created by a schedule. |
| `#` icon | A channel (Slack/Telegram) session; a superscript **A** marks a channel's ambient reader — Slack only. |

A session with no title yet shows `(no title · <id>)` in muted italics, and stays that way however
much unread activity it has — the bold-for-unread styling needs a title.

If the agent asks permission for a tool call and **no session is attached** to receive the prompt, the
request expires after ten minutes and the agent is told it failed, so the tool call aborts cleanly.
That's separate from the longer window on
[network approvals](network-access.md#approvals), which wait for you on Home until that window
runs out and are then denied.

An idle chat session waiting on you shows no marker at all. The sidebar also has a filter over
**Chats**, **Experiment runs**, **Scheduled**, **Channels**, **Runs**, and **Terminal** (all on by default),
and **New**, which opens a chat session straight away — a terminal starts from the launch options
beside the message box (see [Terminal Sessions](#terminal-sessions)). **Delete session** is in a
session's row menu, behind a confirm — right-clicking or long-pressing the row offers the same item.

!!! note "Sending while the agent isn't ready"
    A message you send before the agent can take it is queued and delivered once it can — most often
    a turn already in flight, but also an agent still coming up or a channel not yet engaged. The queue
    is bounded: past about thirty queued messages on one session, further sends are refused rather than
    piling up.

!!! tip "Drafts stay with their session"
    A half-written message is kept with the session you wrote it for, so switching sessions — or
    closing the tab — doesn't lose it. Anything you'd **attached** doesn't survive a reload: the draft
    comes back as text, with a note naming the attachments you'll need to add again.

!!! note "An older agent shows polled data"
    Sessions and files normally update live. An agent still running an image from before that shipped
    can't push updates, so DAM falls back to polling and says so above the conversation: *Live updates
    need a newer agent runtime — showing polled data.* Everything still works, just less promptly. Where
    an update for the agent's image is available the notice offers it right there; where none is yet, it
    tells you that too, so a stale notice isn't something you can act on and fail.

An agent's **background work** — a long build, a batch job it started and left running — keeps showing
in the header and on the session row while it runs, and the result arrives in the session as usual when
it finishes. The indicator only updates while you have the chat open on a running agent, so it isn't a
monitor to watch; it's there so a session between turns doesn't look dead.

### A message that never got through

A prompt that never reached the agent isn't lost and isn't silently dropped. It stays in the
conversation marked **not delivered**, with a note saying why where the cause is known — a rejected
credential, a rate limit, a full queue — and its **⋯** menu offers **Retry**, **Copy text**, and
**Delete**.

That survives a reload. The message is kept with the agent, and your browser keeps its own copy of the
text while the agent is unreachable, handing it back the next time the session loads. So closing the tab
on a send that didn't land still leaves you something to retry.

Two limits worth knowing. Large images attached to such a message aren't kept — the marker names what
won't come back with a retry, so you know to attach it again. And the copies your *browser* is holding
are dropped when you sign out or switch user, which is the one case where retry is gone.

!!! note "A conversation cut short resumes itself"
    If the agent's harness dies unexpectedly mid-turn, the conversation says so — *interrupted by an
    unexpected restart (often out of memory) and is resuming automatically* — and picks the task back
    up on its own. Nothing to retry and nothing to restart.

### Environment variables

Environment variables you set in the agent's Environment editor are persisted separately and delivered to the running agent, applied on the agent's **next turn** — editing env does not restart the pod. User-typed env shadows env that comes from connections when the names collide.

Whether a turn already in flight is disturbed depends on what you changed. **Adding or removing a variable** interrupts a turn still running about a minute after you save, so the new set can take effect. **Changing only a value** never interrupts a running turn — it applies at the next turn boundary.

### Model settings

The **Model** settings panel sets per-agent harness defaults. They apply to **new sessions only**; sessions already underway keep the settings they started with.

The panel lives on the agent's home page, under **Agent Setup**. Chat names the model in use under the
composer; click it and a **Current model** note says which model this agent is using and where to change
it, with **Configure model** to go straight there — so you can check without leaving the conversation.
While the agent is waiting on a permission prompt, chat hides the model line until you've answered.

Which settings appear, and what they're called, depends on the harness — Claude Code offers a model, a permission mode, and a reasoning-effort level; others offer fewer, and some expose no model settings at all — and some choices are hidden for models that don't support them.

Every setting also offers **Default** — *determined by harness settings* — which is what it reads when
you haven't chosen anything, and what to pick to hand a choice back to the harness. Changing the model
clears any companion setting the new model doesn't support, rather than leaving an invalid pair in
place.

While the agent is asleep the panel is read-only, with a **Start agent to edit** action. What it shows then depends on what's on record: normally the last-known values, but an agent that has **never run** has nothing to show at all and says so — settings are resolved when it starts, so the page fills in after the first start.

## Terminal Sessions

Terminal sessions give you PTY-based shell access to the agent's environment — the same interactive TUI
you'd get running the harness locally. Start one with **Terminal (browser)**, beside the message box on
a session you haven't sent anything in yet — alongside **Terminal (local)** and
**VS Code / Zed (local)**, which hand you a copyable command for your own machine instead.

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
