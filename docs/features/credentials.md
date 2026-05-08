# Credentials

Connect your accounts via OAuth so your agent can act on your behalf — without ever seeing your token.

## How It Works

When you connect an OAuth credential:

1. You complete a standard OAuth flow (e.g., GitHub Enterprise)
2. The platform stores your access and refresh tokens in a secure vault
3. A paired gateway pod injects the token on the network wire when your agent makes requests
4. The agent never has access to the raw token

Tokens are auto-refreshed — you don't need to re-authenticate unless you explicitly revoke access.

## Available Connections

| Connection | What It Enables |
|------------|-----------------|
| **GitHub.com** | Clone repos, push branches, create PRs, read issues on github.com |
| **GitHub Enterprise** | Same capabilities for your enterprise GitHub instance |
| **Google** | Gmail, Calendar, Drive, Sheets, Analytics, and Admin access |
| **Generic OAuth** | Connect **any** OAuth 2.1 provider — supply the auth URL, token endpoint, and your client credentials |

## Connecting GitHub

1. Navigate to **Connections** in the DAM UI
2. Click **"Connect GitHub"** (or **"Connect GitHub Enterprise"** for your org's instance)
3. Complete the OAuth flow
4. Authorize the requested permissions

Once connected, your agent can interact with your repos immediately. See [GitHub Integration](../integrations/github.md) for details.

## Connecting Google

1. Navigate to **Connections**
2. Click **"Connect Google"**
3. Sign in and grant the requested scopes (Gmail, Calendar, Drive, Sheets)

## Connecting Any OAuth Provider

DAM supports connecting arbitrary OAuth 2.1 services:

1. Navigate to **Connections**
2. Click **"Add Generic OAuth"**
3. Provide:
    - **Display name** (e.g., "Linear")
    - **Host** the credential should inject on (e.g., `api.linear.app`)
    - **Authorization URL** and **Token endpoint**
    - **Client ID** and **Client Secret**
    - **Scopes** needed
4. Complete the OAuth flow

The credential gateway will inject the resulting token on matching requests, just like built-in connections.

## Managing Connections

- **View** — the Connections panel shows all active connections and their status
- **Revoke** — click "Disconnect" to immediately remove access
- **All your agents** can use your connected credentials
- **Per-user** — your connections are yours alone, never shared with teammates

## Security

Your tokens live in a separate gateway pod that the agent cannot access. Network policy enforces this boundary at the cluster level — even a compromised agent cannot reach the gateway's storage.

See [How Credentials Work](../security/index.md) for the full security model.
