# Slack

Chat with your agent directly from Slack — no need to switch to the DAM UI.

## Overview

DAM integrates with Slack so you can:

- Message your agent from any connected Slack channel
- Get proactive notifications (e.g., Code Guardian review results)
- Collaborate with teammates in shared agent threads
- Maintain persistent sessions per Slack thread

## Setting Up Slack

### Step 1: Link Your Slack Identity

Before you can interact with agents from Slack, link your IBM Slack account to DAM:

1. In any Slack channel where the DAM bot is present, type `/dam login`
2. Follow the link to complete the identity linking
3. Your Slack user is now associated with your DAM account

!!! info "One-time setup"
    You only need to link your identity once. It persists across all channels and agents.


### Step 2: Connect an Agent to a Channel

1. Open your agent in the DAM UI
2. Navigate to **Settings** → **Channels**
3. Click **"Connect Slack Channel"**
4. Select the channel where you want to interact with this agent
5. The agent is now listening in that channel

### Step 3: Start Chatting

In the connected Slack channel, start a new thread and mention your agent. The agent will respond in the thread.

## How Slack Sessions Work

- **One thread = one session** — each Slack thread maps to a persistent DAM session
- **Sessions resume automatically** — if you come back to a thread days later, the agent picks up where you left off
- **Multiple threads** — you can have multiple concurrent conversations with the same agent across different threads

## Multiplayer in Slack

When your agent is connected to a team channel:

- **You** can interact as the agent owner (using your credentials)
- **Colleagues** can reply in the same thread — DAM spawns an isolated fork pod that uses their credentials
- **Access control** — only users on the agent's "allowed users" list can interact

### How Fork Pods Work

When a colleague replies in your agent's thread, DAM doesn't reuse your credentials. Instead:

1. A **separate pod** is spun up for that turn, paired with a gateway loaded with the colleague's credentials
2. The agent processes the reply using the colleague's GitHub token, API keys, etc.
3. The fork pod is torn down after the response completes

This means colleagues can interact with your agent safely — their tokens are never mixed with yours, and the agent can only access repos/services the replier has permission to reach.

!!! tip "Adding allowed users"
    Edit your agent's settings to add colleagues by their linked Slack identity. This prevents unauthorized users from interacting with your agent.


## Proactive Notifications

Some agents (like Code Guardian) can post messages to Slack without being prompted:

- Code review results posted after a scheduled run
- Alerts when a scheduled task completes
- Status updates on long-running operations

## Wake-on-Message

If your agent is hibernated and someone messages it in Slack, the platform automatically wakes the agent. There's a brief delay (~20 seconds) while the pod starts, then the agent responds normally.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot not responding | Check that you've linked your identity (`/dam login`) |
| "Not authorized" error | Ask the agent owner to add you to the allowed users list |
| Agent slow to respond | It may be waking from hibernation — wait ~20 seconds |
| Messages not appearing | Verify the agent is connected to this specific channel |
