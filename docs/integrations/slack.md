# Slack

Chat with your agent directly from Slack — in a channel, a group DM, or a 1:1 DM with the bot.

## Overview

Connecting a Slack conversation to an agent lets you:

- Talk to your agent from Slack instead of the DAM UI
- Let a whole channel work with one agent — and reach the same agent from several conversations
- Share screenshots and images the agent can actually read
- Get proactive posts — results after a scheduled run, alerts when a long task finishes
- Keep a persistent session per thread

## Who can drive the agent

**The binding is the authorization.** Anyone Slack admits to the conversation can drive the agent,
and every turn runs under the **agent's own** credentials and connections — never the sender's.
There is no per-person mode, no login needed to talk to the agent, and no allow-list.

!!! warning "What a binding delegates"
    Connecting a conversation lends the agent to everyone in it. A colleague who pushes to GitHub
    through the agent does so with the **agent's** token, attributed to the agent — not to
    themselves. Bind only where you're comfortable with that, and manage access the way you'd
    manage any Slack membership: channel membership is the gate.

The **binding owner's** Terms of Use acceptance gates every turn — the terms bind the party whose
credentials run the work, not whoever typed.

Turns are attributed in the security log by Slack user id. In channels and group DMs the agent sees
each message labelled with the sender's Slack mention, so a busy multi-speaker channel stays
readable and attributable — a 1:1 DM isn't labelled, since there's only one human in the room.

!!! warning "Connections from before this release were removed"
    Earlier versions let you connect a channel for named people only. That mode is gone, and those
    connections were **removed** on upgrade rather than widened to the whole channel — that's a
    decision only you can make. If a channel went quiet after the upgrade, run `/dam bind` again to
    reconnect it.

## Connect a channel (`/dam bind`)

A conversation does nothing until it's **bound** to an agent. The quickest path is from inside Slack:

1. Invite the **DAM** bot to the channel.
2. Send **`/dam bind`**.
3. Follow the link to authenticate, then pick one of **your own** agents from the picker.
4. You're done — `@mention` the agent and it answers in a thread.

!!! warning "Binding lends the agent to the whole conversation"
    Everyone Slack admits to the conversation drives the agent under its own credentials. Slack
    tells you as much before you confirm.

The same command binds a group DM or a 1:1 DM with the bot; see
[how the agent is triggered](#how-the-agent-is-triggered) for what each surface needs.

| Command | What it does |
|---------|--------------|
| `/dam bind` | Connect one of your agents to this conversation |
| `/dam unbind` | Release the binding — allowed for whoever bound it, or the agent's owner |
| `/dam ambient on\|off` | Turn ambient mode on or off |
| `/dam ambient` | Report whether ambient mode is currently on |

A bare `/dam` prints the usage help for those three commands.

!!! note "When `/dam login` is required"
    Talking to the agent never needs a linked account. Linking only authorizes the *management*
    commands — `/dam unbind` and `/dam ambient on|off`. `/dam bind` links you automatically, so if
    that's how you connected the conversation you're already set; **if you bound from the CLI, you
    were never linked**, and those two commands will ask you to run `/dam login` first.
    Reading the current state with a bare `/dam ambient` never needs a link. `/dam logout` unlinks.

!!! info "One conversation, one agent — but an agent can hold several"
    A Slack conversation binds to at most one agent. The reverse is **not** true: one agent can be
    connected to **several conversations at once** — channels, group DMs, and DMs in any mix — so the
    same workspace, memory, and skills are reachable from all of them without duplicating the
    sandbox. `/dam bind` in an already-connected conversation is refused, even for the same agent;
    run `/dam unbind` there first.

    Conversations never bleed into each other: threads and sessions belong to the conversation they
    happened in, so a reply can't surface in another one. `/dam unbind` releases only the
    conversation you run it in, leaving the agent's other bindings alone.

    A binding is fixed to its conversation — nothing moves one. To reach the same agent from
    somewhere else, bind that conversation too (and unbind this one if you no longer want it).

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

## Ambient mode

Normally the agent only answers when tagged. In **ambient mode** it reads along with the whole
channel and decides for itself when to speak up — answering a question it can answer, picking up
a task someone described, flagging a clear mistake — and stays quiet otherwise. Mentions still
work exactly as before; ambient only adds a second, quieter path in.

**Ambient is off by default** on every connect path, and turning it on is a deliberate choice: an
ambient agent reads *every* message in the channel, not just the ones aimed at it.

```
/dam ambient on      # the agent starts reading along
/dam ambient off     # back to mentions-only
/dam ambient         # report the current state
```

Changing it is allowed for whoever bound the channel, or the agent's owner — the same permission
as `/dam unbind`, and like `unbind` it needs your account
[linked](#connect-a-channel-dam-bind). It can be flipped any time, with no reconnect needed.

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
- **A blocked network request won't stall the turn.** If the agent needs a host you haven't allowed,
  it can't be approved from Slack — so instead of going quiet, the agent tells you which host it
  needs. The request waits in your [Inbox](../core-concepts/network-access.md#approvals); allow it
  there and the agent can pick up where it left off.

!!! tip "How the agent posts"
    Proactive posts work through two built-in tools the agent calls itself: `describe_channel`
    (list the chats it can post to) and `send_channel_message` (post text, optionally with a
    file attached). Nothing to configure — but the agent can only post where the DAM bot is
    already a member, so a channel it hasn't been invited to is refused.

    A reply from a Slack user to one of these posts is routed by that conversation's own binding.
    An agent can also open a DM with someone, but that DM isn't bound by the post — for their
    replies to reach the agent, the DM needs its own `/dam bind`.

## Other ways to bind

You can also connect a **channel** from the DAM UI or the CLI. DMs and group DMs bind in-chat with
`/dam bind`.

From the **UI**, open the sandbox and go to **Channels**. The Slack card lists the channels already
connected — each with whether ambient is on — and **Connect channel** adds another by its channel id.
A connected channel's **⋯** menu offers **Edit** (to flip ambient) and **Disconnect**. The channel
itself can't be swapped for a different one: connect the channel you want and disconnect this one.

!!! note "If the section says no messenger is set up"
    Channels only lists what your operator has configured. Where no messenger is wired up, the
    section says so and points you at them rather than showing an empty list.

From the **CLI**:

```bash
dam channel slack connect <agent> --channel-id <C0…>              # mentions-only
dam channel slack connect <agent> --channel-id <C0…> --ambient    # ambient on from the start
```

Re-running `connect` for the **same** agent and channel updates that binding in place — which is how
you change ambient mode from the CLI, with no disconnect needed. Connecting a *further* channel to the
same agent adds a binding rather than replacing one. Pointing a channel that's already bound at a
*different* agent is refused until that binding is released.

Because an agent can hold several channels, `disconnect` needs to know which one to release:

```bash
dam channel slack disconnect <agent>                          # only when one channel is connected
dam channel slack disconnect <agent> --channel-id <C0…>       # required once there are several
```

See the [CLI reference](../reference/cli.md#channels).

`/dam bind` from inside Slack does the same job, and is the quickest route for a channel you're
already in.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't respond in a channel | Make sure you **`@mention`** it, and that the conversation is bound (`/dam bind`) |
| "This conversation isn't connected" / "No instance connected to this channel" | Run `/dam bind` to connect one of your agents |
| Binding refused as already connected | The conversation is already bound. `/dam unbind` first |
| "Link your account first" on `unbind` or `ambient` | Run `/dam login` once in the channel, then retry — expected if you bound from the CLI |
| Can't unbind | Only the person who bound the conversation, or the agent's owner, can release it |
| Agent replies to things nobody asked it | Ambient mode is on — `/dam ambient off` for mentions-only |
| An attached file was ignored | Only images are read. Paste document contents as text instead |
| "Attached images total … over the cap" | 30 MB of images per message — send fewer or smaller |
| Slow first reply | It's likely waking from hibernation — up to a minute or two |
| It answered without the image | The agent's harness may not support images; you'll have had a private note |
