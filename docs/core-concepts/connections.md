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

### Generic OAuth

Connect any OAuth 2.1 provider (Linear, Jira, Notion, Spotify, etc.):

1. Navigate to **Connections** → **"Add Generic OAuth"**
2. Provide: display name, host, authorization URL, token endpoint, client ID/secret, scopes
3. Complete the OAuth flow

The credential gateway injects the token on matching requests automatically.

## MCP Servers

Connect an external MCP server to give your agent **additional tools** beyond the built-in ones. Once a server is connected and granted to an agent, its tools are exposed to the agent automatically — no per-session setup.

- **Prebuilt** — pick a ready-made MCP connection (e.g. Linear MCP, Atlassian MCP) and complete its auth flow.
- **Custom** — under **Connections → MCP servers**, choose **Add MCP server**, then provide the server's URL (e.g. `https://mcp.acme.internal/sse`) and its auth mode.

!!! info "Built-in vs. connected tools"
    Every agent already has built-in tools for managing its own skills, channels, and schedules — those need no setup and are noted on each feature's page. MCP server connections are how you add *more* tools on top of those.

## API Key Secrets

For services that use static API keys rather than OAuth:

1. Navigate to **Connections** → **Secrets**
2. Click **"Add Secret"**
3. Choose the type and paste your key

### Secrets vs. OAuth Connections

| | Secrets | OAuth Connections |
|---|---------|-------------------|
| How you add them | Paste a key | OAuth flow |
| Token refresh | Manual (you rotate) | Automatic |
| Use case | API keys, static tokens | GitHub, OAuth apps |
| Security | Same gateway injection | Same gateway injection |

## Managing Connections

- **View** — the Connections panel shows all active connections and their status
- **Replace key** — for API-key/secret connections you can rotate the stored value in place (UI: "Replace key") without dropping the connection's existing agent grants. OAuth connections refresh automatically and don't need this.
- **Revoke** — click "Disconnect" to immediately remove access
- **Per-user** — your connections are yours alone, never shared with teammates
- **All agents** — your connected credentials are available to all your agents

## Security

Tokens live in a separate gateway pod that the agent cannot access. Network policy enforces this boundary at the cluster level. See [Architecture](../reference/architecture.md) for the full model.
