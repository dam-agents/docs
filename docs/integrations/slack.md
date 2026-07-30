# Slack

Chat with your agent directly from Slack — in a channel, a group DM, or a 1:1 DM with the bot.

## Overview

Connecting a Slack conversation to an agent lets you:

- Talk to your agent from Slack instead of the DAM UI
- Either give each teammate their own credentials in the channel, or let the whole channel share one agent
- Share screenshots and images the agent can actually read
- Get proactive posts — results after a scheduled run, alerts when a long task finishes
- Keep a persistent session per thread

## Access modes

A Slack binding carries an **access mode**, chosen when the conversation is connected:

| Mode | Who can drive the agent | Whose credentials run the turn |
|------|-------------------------|--------------------------------|
| **Person-scoped** (default) | The agent's owner, plus anyone on the **allowed users** list. Each person links their own Slack account first. | Each person's **own** credentials |
| **Shared** | Anyone Slack admits to the conversation — no login | The **agent's own** credentials |

!!! warning "The mode is fixed for the binding's lifetime"
    You can't switch a binding between modes. Changing it means **disconnect, then reconnect** in
    the new mode — re-running `connect` with a different `--mode` is refused. (Ambient mode, by
    contrast, *can* be flipped in place.)

!!! warning "What a shared binding delegates"
    In **shared** mode the binding itself is the authorization: everyone Slack admits to that
    conversation drives the agent under the **agent's** credentials and connections, not their own.
    Bind shared only where you're comfortable with that, and manage access the way you'd manage any
    Slack membership. The binding owner's Terms of Use acceptance covers every shared turn.

Turns are attributed in the security log by Slack user id. On shared turns in channels and group
DMs the agent sees each message prefixed with the sender's id, and can look that up to get the
person behind it, so a busy channel stays readable — a 1:1 DM isn't labelled, since there's only
one human in the room.

## Connect a channel (`/dam bind`)

A conversation does nothing until it's **bound** to an agent. The quickest path is from inside Slack:

1. Invite the **DAM** bot to the channel.
2. Send **`/dam bind`**.
3. Follow the link to authenticate, then pick one of **your own** agents from the picker.
4. You're done — `@mention` the agent and it answers in a thread.

!!! warning "`/dam bind` always creates a *shared* binding"
    In-chat binding lends the agent to the whole conversation under the agent's own credentials.
    Slack tells you as much before you confirm. For a **person-scoped** binding, connect from the
    [DAM UI or the CLI](#other-ways-to-bind) instead.

The same command binds a group DM or a 1:1 DM with the bot; see
[how the agent is triggered](#how-the-agent-is-triggered) for what each surface needs.

| Command | What it does |
|---------|--------------|
| `/dam bind` | Connect one of your agents to this conversation (shared mode) |
| `/dam unbind` | Release the binding — allowed for whoever bound it, or the agent's owner |
| `/dam ambient on\|off` | Turn ambient mode on or off (shared bindings only) |
| `/dam ambient` | Report whether ambient mode is currently on |
| `/dam login` | Link your Slack account to DAM |
| `/dam logout` | Unlink your Slack account |

!!! warning "When `/dam login` is required"
    On a **person-scoped** binding, linking is required before you can talk to the agent **at all** —
    an unlinked message gets a private *"You need to link your account first. Use `/dam login` to
    get started."* and no turn runs.

    On any binding, `/dam unbind` and `/dam ambient on|off` also check for a linked account.
    `/dam bind` links you automatically, so if that's how you connected the conversation you're
    already set — but **if you bound from the CLI or the DAM UI, you were never linked**. Reading
    the current state with a bare `/dam ambient` never needs a link.

## Control who can use your agent

On a **person-scoped** binding, an **allowed users** list gates who may drive the agent:

- **Empty list** — any linked Slack user in the conversation can drive it.
- **Non-empty list** — only the owner and the listed people can. Everyone else still sees the
  agent's replies but gets *"You don't have access to this instance. Contact the instance owner to
  be added to the allowed users list."* when they try.

Manage it in the agent's **Channels** section in the UI, or from the CLI:

```bash
dam channel allow <agent> --user someone@example.com      # add (repeatable)
dam channel disallow <agent> --user someone@example.com   # remove
dam channel list <agent>                                  # bindings, modes, and the allow-list
```

Shared bindings never consult the list — Slack channel membership is the only gate.

!!! info "An allowed teammate acts as themselves"
    On a person-scoped binding, a reply from someone other than the owner runs in an isolated
    per-turn pod under **that person's own** credentials, torn down once the turn finishes. The
    owner's tokens are never shared. If that provisioning times out you'll see *"Could not run turn
    as you: fork provisioning timed out."*

!!! info "One conversation, one agent"
    A conversation binds to at most one agent, and an agent to at most one Slack conversation
    (channel **or** group DM **or** DM). `/dam bind` in an already-connected conversation is
    refused, even for the same agent — run `/dam unbind` first, then bind again.

## How the agent is triggered

| Surface | What triggers a reply |
|---------|----------------------|
| **Channel** | An `@mention` of the bot |
| **Group DM** | An `@mention` of the bot |
| **1:1 DM** | **Every message** — a DM is already addressed to the bot, so no mention is needed |

Editing an earlier message doesn't re-trigger a reply. Replies land in a thread, unless the agent
chooses Slack's *also send to channel*, which surfaces the post in both places.

Mentioning or DMing an **unbound** conversation gets you a private, sender-only note — in a DM or
group DM it points you at `/dam bind`; in a channel it just says no agent is connected. A plain,
un-addressed channel message gets no notice at all.

## Ambient mode (shared bindings only)

Normally the agent only answers when tagged. In **ambient mode** it reads along with the whole
channel and decides for itself when to speak up — answering a question it can answer, picking up
a task someone described, flagging a clear mistake — and stays quiet otherwise. Mentions still
work exactly as before; ambient only adds a second, quieter path in.

!!! warning "Ambient requires a shared binding"
    Ambient is unavailable on person-scoped bindings — the agent would be reading messages from
    people it can't run turns for. Asking for it there answers *"Ambient mode needs a shared
    connection — this channel is person-scoped. Disconnect the channel and reconnect it in shared
    mode first."* From the CLI, `--ambient` requires `--mode shared`.

**Ambient is off by default** on every connect path, and turning it on is a deliberate choice: an
ambient agent reads *every* message in the channel, not just the ones aimed at it.

```
/dam ambient on      # the agent starts reading along
/dam ambient off     # back to mentions-only
/dam ambient         # report the current state
```

Changing it is allowed for whoever bound the channel, or the agent's owner — the same permission
as `/dam unbind`, and like `unbind` it needs your account
[linked](#connect-a-channel-dam-bind). Unlike the access mode, ambient can be flipped any time.

What to expect when it's on:

- **It's asked to open with an emoji reaction** when it picks a message up — a quiet signal that
  notifies nobody. That's the agent's own judgement rather than an automatic acknowledgement, so
  a missing reaction doesn't reliably mean it passed on the message.
- **It replies in a thread** under the message it's answering. The follow-up conversation runs as
  an ordinary thread from there.
- **It follows the channel**, rather than starting cold on each message, so it can pick up on
  something described across several messages.
- **The change isn't announced in the channel.** Whoever flips it sees the confirmation privately;
  other members get no status post either way.
- **Nothing is ever posted about attachment problems.** Ambient turns stay out of the channel's
  way, so an oversized or unfetchable image is dropped silently and the agent answers on the text.

!!! tip "Tell the channel"
    Since ambient isn't announced, let the channel know when you enable it — people generally
    want to know an agent is reading along.

## Sharing images and screenshots

Paste or drop an **image** into a message and the agent reads it directly — screenshots of a
failing UI, a diagram, a chart. It works on mentions, DMs, and ambient turns alike.

- **Images only.** Non-image attachments (PDFs, CSVs, logs, `.txt`) are **ignored**, with no
  notice in the channel — the message relays as ordinary text. If you need the contents of a
  document, paste the relevant text into the message instead.
- **Up to 30 MB of images per message.** Go over on a mention or DM and the **whole message is
  dropped** — the agent never sees your text either — and you get a private note asking for fewer
  or smaller images.
- If an individual image can't be fetched, you get a private note naming it, and the agent still
  gets your text plus whichever images did load.
- If the agent's harness doesn't support images at all, you get a private note and it answers on
  the text alone.

!!! note "On ambient turns, nothing is reported"
    The notes above are for mentions and DMs. On an [ambient](#ambient-mode) turn nothing is ever
    posted about attachments — over-cap or unfetchable images are dropped silently and the agent
    answers on the text.

## How sessions work

- **One thread = one session.** Each Slack thread maps to its own persistent DAM session.
- **Sessions resume.** Come back to a thread days later and the agent normally picks up where it
  left off. If the session can't be restored, the thread starts fresh with recent messages injected
  as context — so the conversation continues, but the earlier reasoning is gone.
- **Threads run in parallel.** Several conversations with the same agent at once is fine. Within
  one thread, messages are handled in order rather than concurrently.
- **Fresh sessions get context.** A new top-level thread is given the channel's last 10 messages;
  picking up inside an existing thread injects up to 50 of that thread's replies. The window is
  deliberately short, and a resumed session isn't re-injected.
- **Ambient adds a channel-level session** for top-level messages, alongside the per-thread ones.

## What else to expect

- **A working status while it thinks.** On a mention or DM you'll usually see the agent's live
  status — thinking, the tool it's currently running, or waking. It rides Slack's assistant-status
  surface and shows nothing where the workspace can't display it. Ambient turns show no status,
  since nobody summoned it.
- **The first message wakes a sleeping agent.** If it's hibernated, expect a delay of up to a
  minute or two. The initial heads-up is private to whoever sent the message; if the wake takes
  longer or fails, that's posted in the thread for everyone.
- **Agents can post first.** A bound agent can post on its own — results when a scheduled run
  finishes, or an alert.

!!! tip "How the agent posts"
    Proactive posts work through two built-in tools the agent calls itself: `describe_channel`
    (list the chats it can post to) and `send_channel_message` (post text, optionally with a
    file attached). Nothing to configure — but the agent can only post where the DAM bot is
    already a member, so a channel it hasn't been invited to is refused.

    A reply from a Slack user to one of these posts is routed by that conversation's own binding.
    An agent can also open a DM with someone, but that DM isn't bound by the post — for their
    replies to reach the agent, the DM needs its own `/dam bind`.

## Other ways to bind

Unlike `/dam bind`, the CLI and UI paths let you choose the access mode.

From the **CLI**:

```bash
dam channel slack connect <agent> --channel-id <C0…>                       # person-scoped (default)
dam channel slack connect <agent> --channel-id <C0…> --mode shared         # shared
dam channel slack connect <agent> --channel-id <C0…> --mode shared --ambient
```

Re-running `connect` for the **same** agent in the **same** mode updates the binding in place —
which is how you change ambient mode from the CLI, with no unbind needed. Requesting a *different*
mode is refused (disconnect first), and pointing a channel at a *different* agent is refused until
the current binding is released.

See the [CLI reference](../reference/cli.md#channels).

The agent's **Channels** section in the DAM UI does the same job, and lists what's currently bound.
Open **Connect a Slack channel**, paste the **Channel ID** (starts with `C0…`), pick an **Access
mode**, and set **Allowed users** (person-scoped) or **Ambient mode** (shared). Each card shows the
binding's mode at a glance — e.g. *Shared · ambient* or *Person-scoped · 3 allowed users*.

The section sits behind the **Advanced connections** toggle, which is off by default — if you don't
see it, that's why. `/dam bind` needs no toggle.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't respond in a channel | Make sure you **`@mention`** it, and that the conversation is bound (`/dam bind`) |
| "This conversation isn't connected" / "No instance connected to this channel" | Run `/dam bind` to connect one of your agents |
| Binding refused as already connected | The conversation, or that agent, is already bound. `/dam unbind` first |
| "You need to link your account first" | A person-scoped binding requires a linked account before any turn — run `/dam login` |
| "You don't have access to this instance" | Person-scoped binding with an allow-list — ask the owner to add you (`dam channel allow`) |
| "Link your account first" on `unbind` or `ambient` | Run `/dam login` once in the channel, then retry — expected if you bound from the CLI or the UI |
| "Ambient mode needs a shared connection" | Ambient is shared-only. Disconnect and reconnect the channel in shared mode |
| Mode change refused on `connect` | The access mode is fixed per binding — `disconnect`, then connect again in the new mode |
| Can't unbind | Only the person who bound the conversation, or the agent's owner, can release it |
| Agent replies to things nobody asked it | Ambient mode is on — `/dam ambient off` for mentions-only |
| An attached file was ignored | Only images are read. Paste document contents as text instead |
| "Attached images total … over the cap" | 30 MB of images per message — send fewer or smaller |
| Slow first reply | It's likely waking from hibernation — up to a minute or two |
| It answered without the image | The agent's harness may not support images; you'll have had a private note |
