# Telegram

Chat with your agent directly from Telegram.

Unlike Slack — where one DAM app serves the whole workspace — Telegram works **one bot per agent**. You create a bot with Telegram's `@BotFather`, paste its token into DAM, and that bot becomes the front door to that one agent.

!!! info "This is a channel, not a Connection"
    Telegram is set up in your agent's chat **Configuration** (below), not under **Settings → [Connections](../core-concepts/connections.md)**. The one thing you provide is the **bot token** — pasted into the channel config, not created as a Connection. Once paired, the agent can receive messages and post back (proactively or when you ask), all through that bot.

## Overview

Connect an agent to Telegram so you can:

- Message your agent from a Telegram DM or group
- Get proactive notifications (e.g., results after a scheduled run)
- Keep a persistent session per chat — come back later and the agent remembers

## Create Your Bot

Telegram bots are created through Telegram's own bot, **@BotFather**:

1. Open Telegram and start a chat with [`@BotFather`](https://t.me/BotFather).
2. Send `/newbot` and follow the prompts (pick a name and a username ending in `bot`).
3. BotFather replies with a **bot token** that looks like `123456:ABC-DEF...`. Copy it.

!!! info "One bot per agent"
    Each agent has its own Telegram bot. If you want two agents on Telegram, create two bots with `@BotFather`.

## Connect It in DAM

1. Open your agent in the DAM UI.
2. Open the **Configuration** panel (right sidebar).
3. Expand the **Channels** section.
4. Under **Telegram**, check **Enabled**.
5. Paste your bot token into the **Bot token** field.
6. Click **Save**.

!!! tip "Changing the token later"
    Once saved, the token is hidden behind a **Change** button. Click it to paste a new token (for example, if you regenerated it with `@BotFather`).

## Authorize a Chat (`/login`)

Telegram has no shared workspace to tie accounts to, so authorization happens **per conversation** rather than per user. A chat stays unauthorized — the bot won't act — until someone logs in from inside it:

1. Start a chat with your bot (a direct message, or add it to a group).
2. Send `/login` in that chat.
3. Follow the link to complete the sign-in flow. The chat is now authorized.

Send `/logout` in a chat to revoke its access.

!!! warning "Groups: only admins can log in"
    In a group chat, only **group admins** can run `/login`. Until an admin does, the bot stays silent — so it won't spam every group it's added to. Once authorized, anyone in the group can talk to the agent in that chat.

## What to Expect

- **Each chat is a persistent session.** A DM or group chat maps to its own session — return days later and the agent picks up where you left off.
- **The first message wakes a sleeping agent.** If your agent is hibernated, your first message wakes it. There's a brief delay while it starts up (typically under a minute), then it responds normally.
- **Agents can message you first.** A connected agent can post to Telegram on its own — for example, posting results when a scheduled run finishes.

!!! tip "How the agent posts"
    This works through two built-in tools the agent calls itself: `describe_channel` (list the chats it can post to) and `send_channel_message` (post text, optionally with a file attached). They're available automatically whenever a channel is connected — no per-session setup.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot doesn't respond in a DM | Send `/login` in the chat to authorize it |
| Bot silent in a group | A **group admin** must send `/login` — non-admins can't authorize a group |
| Slow to respond | It may be waking from hibernation — can take up to a minute |
| "Unauthorized" after it worked before | Someone may have run `/logout` — send `/login` again |
| Nothing works after connecting | Re-check the **Bot token** in the UI — paste a fresh token from `@BotFather` if needed |
