# Slack

Chat with your agent directly from Slack — in a channel, a group DM, or a 1:1 DM with the bot.

## Overview

Connecting a Slack conversation to an agent lets you:

- Talk to your agent from Slack instead of the DAM UI
- Let a whole channel work with one agent — and reach the same agent from several conversations
- Share screenshots, images, and files the agent can actually work with
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

!!! warning "Channels bound for named people only are gone"
    Earlier versions let you bind a channel for named people only. That mode was removed, and those
    bindings were **deleted** on upgrade rather than widened to the whole channel — that's a decision
    only you can make. If a channel went quiet after the upgrade, run `/dam bind` again to reconnect
    it.

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
| `/dam unbind [agent]` | Release a binding — allowed for whoever bound it, or the agent's owner |
| `/dam default [agent]` | Make an agent the one that unnamed mentions reach — the agent's owner only |
| `/dam ambient [agent] on\|off` | Turn ambient mode on or off for one agent |
| `/dam ambient [agent]` | Report whether ambient mode is currently on |

The agent name is needed only where more than one is connected in the conversation. A bare `/dam`
prints the usage help.

!!! note "When `/dam login` is required"
    Talking to the agent never needs a linked account. Linking only authorizes the *management*
    commands — `/dam unbind` and `/dam ambient on|off`. `/dam bind` links you automatically, so if
    that's how you connected the conversation you're already set; **if you bound from the CLI, you
    were never linked**, and those two commands will ask you to run `/dam login` first.
    Reading the current state with a bare `/dam ambient` never needs a link. `/dam logout` unlinks.

!!! info "Bindings are many-to-many"
    One agent can be connected to **several conversations at once** — channels, group DMs, and DMs in
    any mix — so the same workspace, memory, and skills are reachable from all of them without
    duplicating the agent. And a **conversation can hold several agents**: running `/dam bind` where
    one is already connected *adds* to it rather than replacing it, and Slack tells you who's already
    there before you confirm. See [Several agents in one conversation](#several-agents-in-one-conversation).

    The only bind that's refused is the **same agent twice** in the same conversation.

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

## Several agents in one conversation

A conversation with **one** agent behaves exactly as described above — everything in this section only
starts to matter once you `/dam bind` a second one.

**Reaching a specific agent.** Start your message with its name, right after the bot mention:

```
@dam Ops deploy the staging branch
```

Matching is case-insensitive and ignores extra spacing, and the name has to come first. Anything else
— a bare `@dam` with no name, a name that isn't connected here, or a name two connected agents both
answer to — goes to the conversation's **default agent** instead, which is quietly the common case: it
means a channel keeps working even when nobody remembers the roster. An agent handed an ambiguous name
is told as much and will say so rather than guess.

**The default agent** is whichever was connected first. Change it with `/dam default <agent>` —
allowed for that agent's owner only, since the default absorbs every unnamed mention. A conversation
can also end up with *no* default: disconnecting the default doesn't promote a replacement, and until
someone runs `/dam default`, an unnamed mention reaches nobody and says so.

!!! note "Slash commands are stricter than mentions"
    `/dam unbind`, `/dam ambient`, and `/dam default` need the name to match a connected agent
    exactly, and refuse rather than falling back to the default — so a management command never acts
    on an agent you didn't mean.

**Reading along.** [Ambient mode](#ambient-mode) is per agent, so you can have one agent following the
channel while others stay mentions-only. Where several are ambient, they take a message **one at a
time** rather than all at once, and each later one can see what the earlier ones already posted — so
they don't pile three answers onto the same question. The order isn't fixed, beyond the default going
first.

**Handing a message over.** An agent that judges a teammate better placed to answer can pass the
message to it, and that agent replies in the thread itself. A handed-over message can't be handed on
again, and attachments don't travel with it — the receiving agent gets the text.

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

Where several agents are connected, name the one you mean — `/dam ambient Ops on`. Ambient is set per
agent, so turning it on for one leaves the others mentions-only.

Changing it is allowed for whoever bound the channel, or the agent's owner — the same permission
as `/dam unbind`, and like `unbind` it needs your account
[linked](#connect-a-channel-dam-bind). It can be flipped any time, with no reconnect needed.

What to expect when it's on:

- **Dropping the `@` still reaches it.** People rarely tag a bot mid-conversation, so an ambient
  agent treats a message that simply calls it by name — *DAM*, *dam*, or the agent's own name — as
  addressed to it, and answers as it would a mention. That only applies while ambient is on: in a
  mentions-only channel a typed name is never delivered at all, so nothing is quietly missed.
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
  way: an attachment that couldn't be used is reported to the agent but never to the channel.

!!! tip "Tell the channel"
    Since ambient isn't announced, let the channel know when you enable it — people generally
    want to know an agent is reading along.

## Sharing images and files

Paste or drop an attachment into a message and the agent gets it — a screenshot of a failing UI, a
diagram, a CSV to analyse, a PDF to summarise. It works on mentions, DMs, and ambient turns alike.
Images and other files travel by different routes, so they have separate rules.

**Images** are read directly, in the message itself:

- **PNG, JPEG, GIF, and WebP.** Another picture format — an HEIC photo straight off a phone, an
  SVG, a BMP — is withheld, and the agent is told an attachment was withheld rather than left to
  guess at its contents.
- **Up to 30 MB of images per message.** Go over and the images are left out — your text still goes
  through, and the agent is told what it didn't get.

**Everything else** is saved into the agent's workspace and handed over as a file path, so the agent
can open it with ordinary tools rather than having to read it as a picture:

- **Up to 20 MB per file, and 20 MB per message.** Over either and the file is left out with a
  reason; send them a few at a time.
- The agent is given each file's name and where it landed. A document too dull to read as an image —
  a log, a spreadsheet export, a PDF — is now something it can actually work with.

For either kind, a file that can't be fetched is named and left out, and the agent gets your text plus
whatever did load. Nothing is guessed at: an attachment that didn't make it is reported to the agent as
withheld, along with why.

!!! note "Withheld notes are private, and ambient turns get none"
    On a mention or DM you get a private, sender-only note naming anything that was left out. On an
    [ambient](#ambient-mode) turn nothing is posted in the channel either way — the agent is still told
    what was withheld, but the channel stays quiet.

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
  longer or fails, that's posted in the thread for everyone. A **stopped** agent is different — it
  stays down, and the thread says so: someone has to start it from DAM, or wait for its next
  scheduled run.
- **Every post is signed.** Under each message the agent posts, Slack shows *<agent name> - Powered
  by DAM* as a link. One bot posts for every agent, so the name is how you tell them apart. Anyone
  can follow the link — it lands on the agent's public page, described below, not in the
  conversation; from there someone with a DAM account can **Open in DAM** to reach the thread's
  session itself.
- **Agents can post first.** A bound agent can post on its own — results when a scheduled run
  finishes, or an alert.
- **A blocked network request won't stall the turn.** Nobody in the channel can answer an approval, so
  with no chat, terminal, or SSH session of your own open on the agent, the request is refused
  straight away — and instead of going quiet, the agent tells you which host it needed. It's still
  recorded on [Home](../core-concepts/network-access.md#where-approvals-appear); allowing it there writes the
  rule the agent picks up on its next attempt. With a session of your own open, the request holds and
  resolves as usual.

!!! tip "How the agent posts"
    Proactive posts work through two built-in tools the agent calls itself: `describe_channel`
    (list the chats it can post to) and `send_channel_message` (post text, optionally with a
    single file attached — a large file is refused). Nothing to configure — but the agent can only
    post where the DAM bot is already a member, so a channel it hasn't been invited to is refused.

    A reply from a Slack user to one of these posts is routed by that conversation's own binding.
    An agent can also open a DM with someone, but that DM isn't bound by the post — for their
    replies to reach the agent, the DM needs its own `/dam bind`.

## The agent's public page

For most people in a shared channel, that signature link is the first bit of DAM they ever open — and
they may not have an account. So it goes to a **page that needs no login**, naming the agent, showing
who created it, and explaining that this agent is worked with by mentioning the bot in Slack.
Underneath, it invites the reader to create their own agent; if they already have an account,
**Open in DAM** takes them into the conversation instead.

The page names an agent only while that agent is **connected to at least one channel** — which every
agent posting in Slack is. Anything else shows the same generic introduction to the platform: an
agent nobody has connected to a channel, an address that was never an agent, one that has since been
deleted. That is deliberate, and it means the page is **not** a way to find out whether an agent
exists or to check who may open it — those two states aren't a permission check, and the page can't
tell an owner from a stranger.

!!! note "Disconnecting a channel un-names past links"
    Because naming follows the agent's current channels, releasing the last one turns every
    signature link the agent ever posted — going back through the channel's history — into the
    generic page. The links keep working; they just stop naming the agent.

## Other ways to bind

You can also connect a **channel** from the DAM UI or the CLI. DMs and group DMs bind in-chat with
`/dam bind`.

From the **UI**, open the agent and go to **Channels**. The Slack card lists the channels already
connected — each with whether ambient is on, and a **Default** badge where this agent is the one
unnamed mentions reach — and **Connect channel** adds another by its channel id.
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
same agent adds a binding rather than replacing one, and so does connecting a *different* agent to a
channel that already has one — see
[several agents in one conversation](#several-agents-in-one-conversation). There's no `default`
command in the CLI; that's set in-chat with `/dam default`.

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
| Binding refused as already connected | That **same** agent is already connected here — binding a *different* one is allowed and adds it |
| A mention with no name reached nobody | The conversation has no [default agent](#several-agents-in-one-conversation) — its owner can set one with `/dam default <agent>` |
| The wrong agent answered | Start the message with the agent's name; an unknown or ambiguous name falls back to the default |
| "Link your account first" on `unbind` or `ambient` | Run `/dam login` once in the channel, then retry — expected if you bound from the CLI |
| Can't unbind | Only the person who bound the conversation, or the agent's owner, can release it |
| Agent replies to things nobody asked it | Ambient mode is on — `/dam ambient off` for mentions-only |
| An attached file was ignored | You'll have had a private note saying why — usually over a size cap, or a picture format the agent can't read |
| "they total … over the 30 MB of images" | 30 MB of images per message — send fewer or smaller |
| "the files on this message add up to more than …" | 20 MB of files per message — send them a few at a time |
| Slow first reply | It's likely waking from hibernation — up to a minute or two |
| It answered without the image | The agent's harness may not support images; you'll have had a private note |
