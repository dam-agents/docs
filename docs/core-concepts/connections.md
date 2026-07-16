# Connections

Connect your accounts so your agent can act on your behalf — without ever seeing your tokens.

## Providers, Connections & API Keys

DAM **Settings** has three separate tabs that are easy to confuse. They do different things:

| Tab | What it's for | Example |
|-----|---------------|---------|
| **Providers** | The **model** that powers a sandbox. Every sandbox needs one. | Anthropic, IBM LiteLLM, Bob |
| **Connections** | Credentials and tools that let the agent **act on external services** on your behalf. | GitHub, a custom-header API, an MCP server |
| **API keys** | Tokens that let **you or your automation** drive the DAM platform itself via the API/CLI. | A scoped key for a CI pipeline |

The rest of this page covers **Connections**. Providers are set when you create a sandbox (see [the LiteLLM key guide](../guides/litellm-key.md)); API keys are covered in [the CLI reference](../reference/cli.md).

## How It Works

Connection credentials (OAuth tokens and API keys) are stored in a secure vault and injected on the network wire by a paired gateway pod. The agent never has access to raw tokens — even if compromised, it can't leak your keys.

All credentials — including provider API keys and GitHub personal access tokens — are modeled and managed as Connections. There is no separate "secrets" concept; if you used one previously, it is migrated to a Connection automatically.

## OAuth Connections

OAuth connections auto-refresh and require no manual rotation.

### GitHub

DAM integrates with both GitHub.com and GitHub Enterprise (`github.ibm.com`):

1. Navigate to **Connections** in the DAM UI
2. Click **"Connect GitHub"** or **"Connect GitHub Enterprise"**
3. Complete the OAuth flow and authorize repository access

Once connected, your agent can clone private repos, push branches, create PRs, and use the `gh` CLI (via `GH_TOKEN` auto-set). The connection also seeds the agent's git identity (`~/.gitconfig` `user.name`/`user.email`) from your GitHub account, so commits the agent makes are attributed to you. If your GitHub email is private, a `@users.noreply.github.com` address is used instead.

## API Key Connections

For services that use a static API key rather than OAuth, connect a prebuilt template:

1. Navigate to **Connections** → under **Apps**, pick the service (e.g. **Anthropic (API Key)**)
2. Click **Connect** and paste your key

For a service without a prebuilt template, use **Other** → **Custom header credential** to inject any header (e.g. `Authorization: Bearer …`) on requests to a host you specify. See [Calling an External Service](../guides/external.md).

## MCP Servers

Connect an external MCP server to give your agent **additional tools** beyond the built-in ones. Once a server is connected and granted to an agent, its tools are exposed to the agent automatically — no per-session setup.

Under **Connections → MCP servers**:

- **Custom MCP server (OAuth)** — for a server that authenticates via OAuth.
- **Custom MCP server (no auth)** — for a server that needs no credentials.

Provide the server's URL (e.g. `https://mcp.acme.internal/sse`); the OAuth variant walks you through its auth flow.

!!! info "Built-in vs. connected tools"
    Every agent already has built-in tools for managing its own skills, channels, and schedules — those need no setup and are noted on each feature's page. MCP server connections are how you add *more* tools on top of those.

## Managing Connections

- **View** — the Connections panel shows all your connections; **Connect** / **Disconnect** toggles each one.
- **Revoke** — **Disconnect** immediately removes access.
- **Per-user** — your connections are yours alone, never shared with teammates.
- **All agents** — your connected credentials are available to all your agents.

## Security

Tokens live in a separate gateway pod that the agent cannot access. Network policy enforces this boundary at the cluster level. See [Architecture](../reference/architecture.md) for the full model.
