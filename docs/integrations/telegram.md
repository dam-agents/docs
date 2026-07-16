# Telegram

Chat with your agent directly from Telegram.

DAM uses a **single, platform-wide Telegram bot** for the whole installation — you don't create a bot or manage any tokens. You add the shared bot to a chat, then connect that chat to one of your agents from inside the chat with `/login`. Each chat is bound to a single agent.

## Overview

Connect a chat to an agent so you can:

- Message your agent from a Telegram DM or group
- Get proactive notifications (e.g., results after a scheduled run)
- Keep a persistent session per chat — come back later and the agent remembers

## Find the Bot

There's nothing to create. Every DAM installation exposes one shared Telegram bot, and its handle is shown in the DAM UI:

1. Open an agent in the DAM UI.
2. Open the **Configuration** panel (right sidebar) and expand the **Channels** section.
3. Under **Telegram**, click the bot's `@handle` link to open it in Telegram — or search for that handle in Telegram directly.

Start a DM with the bot, or add it to a group.

## Connect a Chat (`/login`)

A chat does nothing until you bind it to one of your agents. Binding happens **per conversation**, from inside the chat:

1. In the chat (DM or group), send **`/login`**.
2. The bot replies with a link. Open it — it lands on a DAM page in your browser, signed in as you.
3. The page lists **your agents**. Pick the one this chat should talk to.
4. The bot posts a confirmation in the chat. It's now connected.

Everyone in the chat can then talk to the agent you picked, using **the agent's own credentials**.

Send **`/logout`** in a chat to disconnect it.

!!! info "One chat, one agent"
    A chat binds to exactly one agent. If you try to `/login` a chat that's already connected elsewhere, you'll be told the chat is already connected — send `/logout` first, then `/login` again to rebind.

!!! warning "Groups: only admins can log in"
    In a group chat, only **group admins** can run `/login`. Until an admin does, the bot stays silent — so it won't spam every group it's added to. Once connected, anyone in the group can talk to the agent in that chat.

## Manage Connected Chats

The agent's **Channels** panel (Configuration → Channels → Telegram) lists the chats currently connected to that agent. You can **Disconnect** any of them from there — the same effect as sending `/logout` in the chat.

## What to Expect

- **Each chat is a persistent session.** A DM or group chat maps to its own session — return days later and the agent picks up where you left off.
- **The first message wakes a sleeping agent.** If your agent is hibernated, your first message wakes it. There's a brief delay while it starts up (typically under a minute), then it responds normally.
- **Agents can message you first.** A connected agent can post to Telegram on its own — for example, posting results when a scheduled run finishes.

!!! tip "How the agent posts"
    This works through two built-in tools the agent calls itself: `describe_channel` (list the chats it can post to) and `send_channel_message` (post text, optionally with a file attached). They're available automatically whenever a chat is connected — no per-session setup.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't respond in a DM | Send `/login` in the chat to connect it to an agent |
| Bot silent in a group | A **group admin** must send `/login` — non-admins can't connect a group |
| "This chat is already connected to another agent" | Send `/logout` in the chat, then `/login` again to rebind |
| Login link says it expired | Send `/login` again for a fresh link |
| "You don't own any agents yet" on the login page | Create an agent first, then send `/login` in the chat again |
| Slow to respond | It may be waking from hibernation — can take up to a minute |
| "Unauthorized" after it worked before | Someone may have run `/logout` — send `/login` again |
