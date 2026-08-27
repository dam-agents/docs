# Connections

Connect your accounts so your agent can act on your behalf — without ever seeing your tokens.

## Providers, Connections & API Keys

Three of the tabs in DAM **Settings** are easy to confuse. They do different things:

| Tab | What it's for | Example |
|-----|---------------|---------|
| **Providers** | The **model** that powers an agent. Every agent needs one. | Anthropic, OpenAI, IBM LiteLLM ETE Proxy, Bob Shell |
| **Connections** | Credentials and tools that let the agent **act on external services** on your behalf. | GitHub, a custom-header API, an MCP server |
| **API keys** | Tokens that let **you or your automation** drive the DAM platform itself via the API/CLI. | A scoped key for a CI pipeline |

(Settings also has **Account**, **Appearance**, and **Usage** tabs.)

The rest of this page covers **Connections**. Providers live on the **Providers** tab, where you can also update or remove a saved key; you pick one for an agent while creating it (see [Create your LiteLLM key](../reference/litellm-key.md)). API keys are covered in [the CLI reference](../reference/cli.md).

## How It Works

Connection credentials (OAuth tokens and API keys) are stored outside your agent and injected onto outbound requests by a paired gateway that sits between the agent and the internet. The agent never holds the raw token — even if compromised, it can't leak your keys.

Every credential your agent uses on the wire — provider API keys, GitHub tokens, custom headers — is a Connection. (Private-registry credentials are the one exception; you set those on the agent itself.)

## OAuth Connections

OAuth connections auto-refresh and require no manual rotation.

### GitHub

DAM integrates with GitHub.com and with any GitHub Enterprise host you supply:

1. Go to **Settings → Connections** and click **New** to open the **Connection catalogue**
2. On the **Apps** tab, click **New** on the **GitHub** card, then choose **Sign in with Github** and
   complete the OAuth flow, or **Personal access token** and paste a token you create on GitHub — the
   token path is the one to pick when you want finer-grained repo access
3. For **GitHub Enterprise**, **New** opens **Sign in with Github Enterprise** — supply the host
   (plus an OAuth client id and secret, unless your operator registered one for everyone) and
   complete the OAuth flow

Signing in confirms who you are — on its own it doesn't grant access to your repositories. For private
repos, use the row's **Install on GitHub** link to install the app on the organization that owns them,
choosing which of its repositories DAM may read and write. That's done once per organization, and if
you don't own the org an owner has to approve or complete it.

Once that's in place, your agent can clone private repos, push branches, create PRs, and use the `gh` CLI (via `GH_TOKEN` auto-set).

When you sign in with **GitHub.com**, commits the agent makes are attributed to that account — DAM writes your GitHub display name and email into the agent's git config (`user.name` / `user.email`). If your GitHub profile email is private, a `users.noreply.github.com` address is used instead, so your primary email never lands in commit history. Other GitHub connection types, including GitHub Enterprise, don't set this — configure git yourself there.

## API Key Connections

For services that use a static API key rather than OAuth, connect a prebuilt template:

1. Go to **Settings → Connections** → **New** to open the **Connection catalogue**, then on the **Apps** tab pick the service (e.g. **Modal**)
2. Fill in the fields the service asks for — the key, plus anything else on the form (Modal also wants its **Token ID**) — and click **Create**

Model providers aren't here — they live on the **Providers** tab (see above).

### Kubernetes / OpenShift

Connect a cluster with its **API server URL** and a service-account token (plus the **Cluster CA
certificate**, if its endpoint isn't publicly trusted). Your agent gets a working `kubectl` / `oc` —
the token itself never reaches the agent, so it can't leak from there. Connect several clusters and
they compose.

For a service without a prebuilt template, use the **Custom Headers** tab → **New** to inject any header (e.g. `Authorization: Bearer …`) on requests to a host you specify. See [Calling an External Service](../guides/external.md).

## MCP Servers

Connect an external MCP server to give your agent **additional tools** beyond the built-in ones. Once a server is connected and granted to an agent, its tools are exposed to the agent automatically — no per-session setup.

Under **Connections → MCP servers**, click **New**, then give the server a name and its URL (e.g.
`https://mcp.acme.internal/sse`). DAM checks the URL and works out whether it needs OAuth — if it
does, the button becomes **Create + Authorize** and walks you through the server's sign-in. An
**Advanced configuration** section lets you pin your own OAuth client or a header credential instead.

!!! info "Built-in vs. connected tools"
    Every agent already has built-in tools for managing its own skills, channels, and schedules — those need no setup and are noted on each feature's page. MCP server connections are how you add *more* tools on top of those.

## Managing Connections

- **View** — the Connections panel lists everything you've connected, grouped by service.
- **Remove from one agent** — a connection's **⋯** menu → **Remove from this agent** revokes it there and leaves it available elsewhere.
- **Delete everywhere** — **⋯** → **Delete this connection** removes it for every agent. DAM confirms first and lists the agents that will lose it.
- **Per-user** — your connections are yours alone, never shared with teammates.
- **Granted per agent** — a connection you own reaches an agent only once you grant it there. Connect once under Settings, then grant it to each agent that should use it.
- **Rotating a key** — when an API-key or custom-header connection's value rotates upstream, update it in place rather than disconnecting and reconnecting: the connection's **⋯** menu → **Update credential**, or `dam connection update`. The connection keeps its name and all its agent grants, and nothing restarts. OAuth connections refresh themselves through their own flow and need no rotation.
- **Re-authenticating** — a connection whose credential stopped working shows **Expired**, with a one-click **Re-authenticate** button on the row. It re-runs login and consent and replaces only the tokens: the connection keeps its name and every agent grant. `dam connection reauth` does the same from the CLI.

## Security

Tokens live in a separate gateway pod your agent cannot reach. Network policy enforces this boundary at the cluster level. See [Architecture](../reference/architecture.md) for the full model.
