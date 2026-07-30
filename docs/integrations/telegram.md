# Telegram

Chat with your agent from a Telegram DM or group.

## How Telegram differs from Slack

Telegram has no workspace to anchor per-person identity against, so consent attaches to the
**conversation**: whoever binds a chat lends their agent to everyone in it. Every turn runs under the
**agent's own** credentials, and members never log in. That makes Telegram equivalent to a
[shared Slack binding](slack.md#access-modes) — there is no person-scoped option and no allow-list.

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
| `/dam unbind` | Release the binding |
| `/dam` | Show the available commands |

!!! note "One agent per chat"
    A chat binds to one agent. Binding an already-connected chat is refused with *"This chat is
    already connected to an agent. Send `/dam unbind` first to reconnect."* Unlike Slack, the same
    agent **can** be bound to several Telegram chats.

The login link expires after about 10 minutes — send `/dam bind` again if you miss the window.

You can also release a bound chat from the DAM UI; the bot posts a farewell in the chat before the
binding goes away.

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

Each thread maps to its own persistent session, the same as Slack. Ambient mode is Slack-only.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Only group admins can `/dam bind`" | Ask a chat admin to run it, or bind from the DAM UI |
| "This chat is already connected to an agent" | Send `/dam unbind` first, then bind again |
| "This chat isn't connected to an agent" | Run `/dam bind` to connect one of your agents |
| Bot silent in a group | Make sure you `@mention` it, and that the chat is bound |
| The login link didn't work | It expires after ~10 minutes — send `/dam bind` again |
| An attached image was ignored | Telegram relays text only. Paste the content instead |
| Asked to accept Terms of Use | The agent's owner must accept the terms in the DAM UI before turns run |
