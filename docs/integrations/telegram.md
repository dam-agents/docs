# Telegram

Chat with your agent from a Telegram DM or group.

## How Telegram differs from Slack

Telegram uses the same access model as Slack: the binding is the authorization, so anyone in a bound
chat drives the agent under the **agent's own** credentials, with no login needed. See
[who can drive the agent](slack.md#who-can-drive-the-agent).

What differs is everything around it — Telegram has no identity linking, so **group admin** is the
gate on the in-chat commands; messages relay as text only; there's no ambient mode; one agent can be
bound to several chats; and a first message to a sleeping agent gets no heads-up — Telegram just goes
quiet until the agent is up, where Slack tells you it's waking.

## Connect a chat

1. Add the bot to the chat (or open a DM with it).
2. Send **`/dam bind`**. In a **group, only admins** can do this — anyone else gets
   *"Only group admins can `/dam bind`."*
3. Follow the login link, then pick one of **your own** agents.
4. The bot confirms in the chat.

`/start` also counts as bind intent, so deep links and Telegram's Start button work.

| Command | What it does |
|---------|--------------|
| `/dam bind` | Connect one of your agents to this chat (group admins only) |
| `/dam unbind` | Release the binding (group admins only) |
| `/dam` | Show the available commands |

!!! note "One agent per chat"
    A chat binds to one agent. Binding an already-connected chat is refused with *"This chat is
    already connected to an agent. Send `/dam unbind` first to reconnect."* Unlike Slack, the same
    agent **can** be bound to several Telegram chats.

The login link expires after about 10 minutes — send `/dam bind` again if you miss the window.

Send `/dam unbind` in the chat to release the binding; the bot posts a farewell in the chat before it
goes away.

## How the agent is triggered

| Surface | What triggers a reply |
|---------|----------------------|
| **DM** | Every message |
| **Group** | An `@mention` — after the first one, follow-ups in that thread are picked up too |

Unbound **groups stay silent**, so the bot doesn't spam every chat it's been added to. In an unbound
**DM** it replies *"This chat isn't connected to an agent. An admin needs to send `/dam bind`."*

!!! warning "Text only"
    Telegram messages relay as **text**. Images and other attachments are not passed to the agent —
    unlike [Slack](slack.md#sharing-images-and-screenshots), which does read images. Paste the
    content you need the agent to see.

## Sessions

**One chat, one session.** A Telegram chat maps to a single persistent session, so a group
conversation continues in one place rather than splitting per reply — unlike
[Slack](slack.md#how-sessions-work), where each thread gets its own session. Ambient mode is Slack-only.

A Telegram agent can post on its own — a scheduled result, an alert — but only into chats bound to
it. Unlike Slack, it can't reach a chat it isn't connected to.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Only group admins can `/dam bind`" | Only a chat admin can connect a group. Ask one to run it |
| "Only group admins can `/dam unbind`" | Ask a chat admin to run `/dam unbind` in the chat |
| "This chat is already connected to an agent" | Send `/dam unbind` first, then bind again |
| "This chat isn't connected to an agent" | Run `/dam bind` to connect one of your agents |
| Bot silent in a group | Make sure you `@mention` it, and that the chat is bound |
| The login link didn't work | It expires after ~10 minutes — send `/dam bind` again |
| An attached image was ignored | Telegram relays text only. Paste the content instead |
| Asked to accept Terms of Use, or silence in a group | Whoever bound the chat must accept the terms in the DAM UI before turns run. In a group nothing is posted — the bot just stays quiet |
