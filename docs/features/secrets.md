# Secrets

Store API keys and tokens that your agent needs to function — securely, without exposing them to the agent itself.

## How It Works

When you add a secret (like an Anthropic API key):

1. You paste the key in the Secrets UI
2. It's stored as a Kubernetes Secret in a secure vault
3. The paired gateway pod injects it on the wire when the agent makes requests to the relevant service
4. The agent sees requests succeed but never has access to the raw key

## Adding a Secret

1. Navigate to **Connections** → **Secrets**
2. Click **"Add Secret"**
3. Choose the secret type (Anthropic API Key, generic token, etc.)
4. Paste your key
5. Click **Save**

## Secret Types

| Type | Purpose | Required By |
|------|---------|-------------|
| **Anthropic API Key** | Powers Claude Code agents | Claude Code, Code Guardian |
| **OpenAI API Key** | GPT-4 access | Pi Agent |
| **Mistral API Key** | Mistral access | Pi Agent |
| **Generic API Token** | Any service that uses Bearer token auth | Custom integrations |

## Managing Secrets

- **View** — the Secrets panel shows which secrets are configured (values are hidden)
- **Update** — paste a new value to rotate a key
- **Delete** — remove a secret to immediately revoke agent access to that service

## Secrets vs. Credentials

| | Secrets | Credentials |
|---|---------|-------------|
| **How you add them** | Paste a key manually | OAuth flow |
| **Token refresh** | Manual (you rotate it yourself) | Automatic (platform refreshes) |
| **Use case** | API keys, static tokens | GitHub, Google, OAuth apps |
| **Security** | Same gateway injection model | Same gateway injection model |

Both are stored identically in the secure vault and injected the same way — the difference is just how you obtain and manage the token.
