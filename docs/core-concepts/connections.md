# Connections

Connect your accounts so your agent can act on your behalf — without ever seeing your tokens.

## Providers, Connections & API Keys

Three of the tabs in DAM **Settings** are easy to confuse. They do different things:

| Tab | What it's for | Example |
|-----|---------------|---------|
| **Providers** | The **model** that powers a sandbox. Every sandbox needs one. | Anthropic, OpenAI, IBM LiteLLM ETE Proxy, Bob Shell |
| **Connections** | Credentials and tools that let the agent **act on external services** on your behalf. | GitHub, a custom-header API, an MCP server |
| **API keys** | Tokens that let **you or your automation** drive the DAM platform itself via the API/CLI. | A scoped key for a CI pipeline |

(Settings also has **Account**, **Appearance**, and **Usage** tabs.)

The rest of this page covers **Connections**. Providers are set when you create a sandbox (see [the LiteLLM key guide](../guides/litellm-key.md)); API keys are covered in [the CLI reference](../reference/cli.md).

## How It Works

Connection credentials (OAuth tokens and API keys) are stored outside your sandbox and injected onto outbound requests by a paired gateway that sits between the sandbox and the internet. The sandbox never holds the raw token — even if compromised, it can't leak your keys.

All credentials — including provider API keys and GitHub personal access tokens — are modeled and managed as Connections. There is no separate "secrets" concept; if you used one previously, it is migrated to a Connection automatically.

## OAuth Connections

OAuth connections auto-refresh and require no manual rotation.

### GitHub

DAM integrates with both GitHub.com and GitHub Enterprise (`github.ibm.com`):

1. Navigate to **Connections** in the DAM UI
2. Click **New** on the **GitHub** (or **GitHub Enterprise**) card
3. Choose **Sign in with Github** and complete the OAuth flow, or **Personal access token** and paste
   a token you create on GitHub — the token path is the one to pick when you want finer-grained repo
   access

Once connected, your agent can clone private repos, push branches, create PRs, and use the `gh` CLI (via `GH_TOKEN` auto-set).

When you sign in with **GitHub.com**, commits the agent makes are attributed to that account — DAM writes your GitHub display name and email into the agent's git config (`user.name` / `user.email`). If your GitHub profile email is private, a `users.noreply.github.com` address is used instead, so your primary email never lands in commit history. Other GitHub connection types, including GitHub Enterprise, don't set this — configure git yourself there.

## API Key Connections

For services that use a static API key rather than OAuth, connect a prebuilt template:

1. Navigate to **Connections** → **New** → under **Apps**, pick the service (e.g. **Modal**)
2. Paste your key and click **Create**

Model providers aren't here — they live on the **Providers** tab (see above).

### Kubernetes / OpenShift

Connect a cluster with its API endpoint and a service-account token (plus the cluster's CA, if its
certificate isn't publicly trusted). Your agent gets a working `kubectl` / `oc` — the token itself
never reaches the sandbox, so it can't leak from there. Connect several clusters and they compose.

For a service without a prebuilt template, use the **Custom Headers** tab → **New** to inject any header (e.g. `Authorization: Bearer …`) on requests to a host you specify. See [Calling an External Service](../guides/external.md).

## MCP Servers

Connect an external MCP server to give your agent **additional tools** beyond the built-in ones. Once a server is connected and granted to a sandbox, its tools are exposed to the agent automatically — no per-session setup.

Under **Connections → MCP servers**, click **New**, then give the server a name and its URL (e.g.
`https://mcp.acme.internal/sse`). DAM checks the URL and works out whether it needs OAuth — if it
does, the button becomes **Create + Authorize** and walks you through the server's sign-in. An
**Advanced configuration** section lets you pin your own OAuth client or a header credential instead.

!!! info "Built-in vs. connected tools"
    Every agent already has built-in tools for managing its own skills, channels, and schedules — those need no setup and are noted on each feature's page. MCP server connections are how you add *more* tools on top of those.

## Managing Connections

- **View** — the Connections panel lists everything you've connected, grouped by service.
- **Remove from one sandbox** — a connection's **⋯** menu → **Remove from this sandbox** revokes it there and leaves it available elsewhere.
- **Delete everywhere** — **⋯** → **Delete this connection** removes it for every sandbox. DAM confirms first and lists the sandboxes that will lose it.
- **Per-user** — your connections are yours alone, never shared with teammates.
- **Granted per sandbox** — a connection you own reaches a sandbox only once you grant it there. Connect once under Settings, then grant it to each sandbox that should use it.
- **Rotating a key** — when an API key rotates upstream, update the connection's value in place (`dam connection update`) rather than disconnecting and reconnecting. The connection keeps its name and all its sandbox grants, and nothing restarts. OAuth connections refresh themselves and need no rotation.

## Security

Tokens live in a separate gateway pod that the agent cannot access. Network policy enforces this boundary at the cluster level. See [Architecture](../reference/architecture.md) for the full model.
