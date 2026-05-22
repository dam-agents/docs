# Connections

Connect your accounts and API keys so your agent can act on your behalf — without ever seeing your tokens.

## How It Works

All credentials (OAuth tokens and API keys) are stored in a secure vault and injected on the network wire by a paired gateway pod. The agent never has access to raw tokens — even if compromised, it can't leak your keys.

## OAuth Connections

OAuth connections auto-refresh and require no manual rotation.

### GitHub

DAM integrates with both GitHub.com and GitHub Enterprise (`github.ibm.com`):

1. Navigate to **Connections** in the DAM UI
2. Click **"Connect GitHub"** or **"Connect GitHub Enterprise"**
3. Complete the OAuth flow and authorize repository access

Once connected, your agent can clone private repos, push branches, create PRs, and use the `gh` CLI (via `GH_TOKEN` auto-set).

### Google Workspace

Connect Google services (Gmail, Calendar, Drive, Sheets, Docs, and more):

1. Navigate to **Connections**
2. Click **"Connect Google"**
3. Sign in and grant the requested scopes

### Generic OAuth

Connect any OAuth 2.1 provider (Linear, Jira, Notion, Spotify, etc.):

1. Navigate to **Connections** → **"Add Generic OAuth"**
2. Provide: display name, host, authorization URL, token endpoint, client ID/secret, scopes
3. Complete the OAuth flow

The credential gateway injects the token on matching requests automatically.

## API Key Secrets

For services that use static API keys rather than OAuth:

1. Navigate to **Connections** → **Secrets**
2. Click **"Add Secret"**
3. Choose the type and paste your key

| Type | Required By |
|------|-------------|
| Anthropic API Key | Claude Code |
| OpenAI API Key | Codex, Pi Agent |
| Mistral API Key | Pi Agent |
| Bob API Key | Bob Agent |
| Generic API Token | Any Bearer-token service |

### Secrets vs. OAuth Connections

| | Secrets | OAuth Connections |
|---|---------|-------------------|
| How you add them | Paste a key | OAuth flow |
| Token refresh | Manual (you rotate) | Automatic |
| Use case | API keys, static tokens | GitHub, Google, OAuth apps |
| Security | Same gateway injection | Same gateway injection |

## Managing Connections

- **View** — the Connections panel shows all active connections and their status
- **Revoke** — click "Disconnect" to immediately remove access
- **Per-user** — your connections are yours alone, never shared with teammates
- **All agents** — your connected credentials are available to all your agents

## Security

Tokens live in a separate gateway pod that the agent cannot access. Network policy enforces this boundary at the cluster level. See [Architecture](../reference/architecture.md) for the full model.
