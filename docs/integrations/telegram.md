# Telegram

Chat with your agent from a Telegram DM or group.

## How Telegram differs from Slack

Telegram uses the same access model as Slack: the binding is the authorization, so anyone in a bound
chat drives the agent under the **agent's own** credentials, with no login needed. See
[who can drive the agent](slack.md#who-can-drive-the-agent).

What differs is everything around it — Telegram has no identity linking, so **group admin** is the
gate on the in-chat commands; messages relay as text only; there's no ambient mode; and a first
message to a sleeping agent gets no heads-up — Telegram just goes quiet until the agent is up, where
Slack tells you it's waking.

## Connect a chat

1. Add the bot to the chat (or open a DM with it).
2. Send **`/bind`**. In a **group, only admins** can do this — anyone else gets
   *"Only group admins can `/bind`."*
3. Follow the login link, then pick one of **your own** agents.
4. The bot confirms in the chat.

`/start` also counts as bind intent, so deep links and Telegram's Start button work.

| Command | What it does |
|---------|--------------|
| `/bind` | Connect one of your agents to this chat (group admins only) |
| `/unbind` | Release the binding (group admins only) |

Both are published to Telegram's own command menu, so typing `/` in the chat lists them with a
description — there's no separate help command to find. In a group with more than one bot, address a
command to this one as `/bind@<botname>`.

!!! note "These commands are Telegram's own, not `/dam`'s"
    Telegram's commands are bare: `/bind`, not `/dam bind`. Slack is different — there the agent is driven
    with the `/dam` [slash command](slack.md), because a Slack app owns one command name and namespaces
    its verbs underneath it.

!!! note "One agent per chat"
    A chat binds to one agent. Binding an already-connected chat is refused with *"This chat is
    already connected to an agent. Send `/unbind` first to reconnect."* The same agent **can** be
    bound to several chats — as it can to several [Slack](slack.md) conversations.

The login link expires after about 10 minutes — send `/bind` again if you miss the window.

Send `/unbind` in the chat to release the binding; the bot confirms in the chat.

You can also disconnect from DAM: open the agent, go to **Channels**, and click **Disconnect** on
the chat in the Telegram card. The bot posts a farewell note in the chat before the binding goes. The
card lists the chats already connected and links the bot so you can add it somewhere new — but
connecting itself always happens in the chat, with `/bind`.

!!! note "If Telegram isn't listed"
    Channels only shows what your operator has set up. Where Telegram isn't wired up, its card isn't
    there — ask them.

## How the agent is triggered

| Surface | What triggers a reply |
|---------|----------------------|
| **DM** | Every message |
| **Group** | An `@mention` — after the first one, follow-ups in that chat are picked up too (the agent decides whether to answer) |

Unbound **groups stay silent**, so the bot doesn't spam every chat it's been added to. In an unbound
**DM** it replies *"This chat isn't connected to an agent. An admin needs to send `/bind`."*

!!! warning "Inbound is text only"
    Telegram messages relay as **text**. Images and other attachments are not passed to the agent —
    unlike [Slack](slack.md#sharing-images-and-files), which reads images and hands over files. Paste
    the content you need the agent to see.

    That's about messages *coming in*. Going the other way, an agent posting to Telegram of its own
    accord can attach a single file to its message.

## Sessions

**One chat, one session.** A Telegram chat maps to a single persistent session, so a group
conversation continues in one place rather than splitting per reply — unlike
[Slack](slack.md#how-sessions-work), where each thread gets its own session. (In a forum group each
topic counts as its own chat, so it binds and sessions separately.) Ambient mode is Slack-only.

**A burst of messages gets one answer**, as on [Slack](slack.md#how-sessions-work): the agent waits
for a short pause, then answers everything that arrived together. Something sent while it's already
working is handed to that turn where the harness supports it; otherwise it becomes the next turn
rather than a second answer racing the first. Bursts are grouped per **chat**, not per person — so
in a group, messages from several people that land together are answered together.

As on Slack, nobody is there to answer a harness permission prompt, so a tool the agent would have to
ask about is refused; see [what else to expect](slack.md#what-else-to-expect).

A Telegram agent can post on its own — a scheduled result, an alert — but only into chats bound to
it. Unlike Slack, it can't reach a chat it isn't connected to.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Only group admins can `/bind`" | Only a chat admin can connect a group. Ask one to run it |
| "Only group admins can `/unbind`" | Ask a chat admin to run `/unbind` in the chat |
| "This chat is already connected to an agent" | Send `/unbind` first, then bind again |
| "This chat isn't connected to an agent" | Run `/bind` to connect one of your agents |
| Bot silent in a group | Make sure you `@mention` it, and that the chat is bound |
| The login link didn't work | It expires after ~10 minutes — send `/bind` again |
| An attached image was ignored | Telegram relays text only. Paste the content instead |
| Asked to accept Terms of Use in a DM, or silence in a group | Whoever bound the chat must accept the terms in DAM before any turn runs — turns use that person's consent, not yours. A DM says so; in a group nothing is posted and the bot just stays quiet |
