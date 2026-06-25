# Slack

!!! warning "IBM Enterprise Slack — not yet available"
    DAM is currently going through the IBM Slack enterprise approval process. For now, Slack integration only works with **non-enterprise Slack workspaces**. Enterprise workspace support will be available once org-level approval is granted.

Chat with your agent directly from Slack.

## Overview

DAM integrates with Slack so you can:

- Message your agent from any connected Slack channel
- Get proactive notifications (e.g., Code Guardian review results)
- Collaborate with teammates in shared agent threads
- Maintain persistent sessions per Slack thread

## Link Your Slack Identity

Before you can interact with any agent from Slack, link your Slack account to DAM:

1. In any Slack channel where the DAM bot is present, type `/dam login`
2. Follow the link to complete the identity linking
3. Your Slack user is now associated with your DAM account

!!! info "One-time setup"
    You only need to link your identity once. It persists across all channels and agents.

---

## As an Agent Owner

You own an agent and want to make it available in Slack.

### Connect Your Agent to a Channel

1. Create or pick a Slack channel for your agent
2. Invite the DAM bot to the channel (if not already present)
3. Open your agent in the DAM UI
4. Open the **Configuration** panel (right sidebar)
5. Expand the **Channels** section
6. Check **Enabled** under Slack
7. Paste the Slack **Channel ID** (starts with `C0...`)
8. Click **Save**

!!! tip "Finding a Slack Channel ID"
    Right-click a channel name in Slack → **View channel details** → scroll to the bottom to find the Channel ID.

!!! warning "One agent per channel"
    A Slack channel can only be connected to **one** agent. If you need multiple agents, use separate channels.

### Control Who Can Use Your Agent

In the **Channels** section you can add an **Allowed users** list (by email address):

- **List is empty** — any user who has linked their Slack identity (`/dam login`) can message your agent in that channel.
- **List has entries** — only those users can interact. Everyone else gets an ephemeral "not authorized" message.

### What to Expect

- Mention your agent (`@DAM`) in the channel to start a conversation — it replies in a thread.
- Each thread is its own persistent session. Come back days later and the agent remembers.
- You can have multiple threads (sessions) running at the same time.
- If your agent is hibernated, the first message wakes it (takes up to a minute).

### Inviting Colleagues

When colleagues reply in a thread with your agent:

1. They must have linked their identity (`/dam login`) — this is a one-time step.
2. They must be on your allowed users list (or the list must be empty).
3. DAM spins up an **isolated fork pod** using the colleague's own credentials — your tokens are never shared.
4. The fork pod is torn down after each response.

This means teammates can safely interact with your agent without ever seeing or using your credentials.

---

## As an Invited User

Someone else set up an agent in a Slack channel and invited you (or you joined the channel).

### What You Need to Do

1. **Link your identity** (once) — type `/dam login` in any channel where the DAM bot is present and complete the OAuth flow.
2. **Make sure you're on the allowed list** — if you get a "not authorized" error, ask the agent owner to add your email in the DAM UI.
3. **Start chatting** — mention the agent (`@DAM`) in the channel or reply in an existing thread.

### What to Expect

- The agent responds in a thread. Each thread is a separate conversation.
- Your messages run using **your own credentials** (GitHub tokens, API keys, etc.) via an isolated fork pod — the agent owner's credentials are never involved in your turns.
- If the agent is asleep, your first message wakes it (can take up to a minute).
- You can have multiple concurrent threads with the same agent.

---

## How Sessions Work

- **One thread = one session** — each Slack thread maps to a persistent DAM session
- **Sessions resume automatically** — if you come back to a thread days later, the agent picks up where you left off
- **Multiple threads** — you can have multiple concurrent conversations with the same agent across different threads


## Proactive Notifications

Agents can post messages to Slack without being prompted:

- Results posted after a scheduled run completes
- Alerts when a scheduled task finishes
- Status updates on long-running operations

!!! tip "How the agent posts"
    This works through two built-in tools the agent calls itself: `describe_channel` (list the chats it can post to) and `send_channel_message` (post text, optionally with a file attached). They're available automatically whenever a channel is connected — no per-session setup.

## Wake-on-Message

If your agent is hibernated and someone messages it in Slack, the platform automatically wakes the agent. There's a brief delay while the pod starts (typically under a minute), then the agent responds normally.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot not responding | Check that you've linked your identity (`/dam login`) |
| "Not authorized" error | Ask the agent owner to add you to the allowed users list |
| Agent slow to respond | It may be waking from hibernation — can take up to a minute |
| Messages not appearing | Verify the agent is connected to this specific channel |
