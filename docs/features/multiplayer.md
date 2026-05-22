# Multiplayer

Share your agent with teammates — they interact using their own credentials, not yours.

## Overview

DAM agents support multi-user collaboration. You can grant colleagues access to your agent, and when they interact:

- Their messages run with **their** credentials (GitHub token, API keys, etc.)
- **Your** credentials are never exposed to them
- The workspace (files, repos) is shared for collaboration
- Actions are attributed to the correct user (e.g., git commits)

## Adding Collaborators

1. Open your agent's **Settings**
2. Go to **Allowed Users**
3. Add colleagues by their linked identity (IBM SSO or Slack handle)

## Where Multiplayer Works

| Channel | How It Works |
|---------|--------------|
| **Slack** | Colleagues reply in the agent's thread — a fork pod handles their turn |
| **Telegram** | Same per-turn isolation model |
| **DAM UI** | Allowed users can open the agent directly |

## How It's Secured

Each collaborator's turn runs in an isolated fork pod with only their credentials loaded. See [Architecture](../reference/architecture.md#multiplayer-security) for the full model.
