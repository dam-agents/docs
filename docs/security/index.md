# Security

DAM is designed so that even a fully compromised agent cannot leak your credentials or exfiltrate your data.

## Credential Isolation

Your tokens and API keys are never visible to your agent — they're injected on the network wire by a separate secure gateway.

### The Gateway Model

Every agent instance has a **paired gateway pod** — a separate, hardened container that handles credential injection:

```
┌─────────────────────────────────────────────────────┐
│ Your Agent Pod                                      │
│                                                     │
│  Agent makes HTTPS request to api.github.com  ──────┼──┐
│  (no credentials attached)                          │  │
└─────────────────────────────────────────────────────┘  │
                                                         │
    ┌────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────┐
│ Gateway Pod (Envoy Proxy)                           │
│                                                     │
│  1. Intercepts the request                          │
│  2. Looks up your stored GitHub token               │
│  3. Injects "Authorization: Bearer <token>"         │
│  4. Forwards to api.github.com                      │
└─────────────────────────────────────────────────────┘
```

**Key properties:**

- The agent pod has **no access** to the raw token — it's not in environment variables, not on disk, not in memory
- Kubernetes NetworkPolicy enforces that the agent can **only** reach the internet through the gateway — it cannot bypass this
- Even if an agent is fully compromised (prompt injection, malicious code), it cannot exfiltrate your credentials

### Credential Scoping

- **Per-user:** Your credentials are yours alone. No other team member can use or see them.
- **Per-service:** Each connected service (GitHub, Anthropic, Google) is stored separately.
- **All your agents:** Any agent you own can use your connected credentials.

### Token Management

| Type | Storage | Refresh |
|------|---------|---------|
| **OAuth** (GitHub, Google) | Kubernetes Secret in gateway pod | Auto-refreshed by the platform |
| **API Key** (Anthropic, etc.) | Kubernetes Secret in gateway pod | No expiry unless you rotate manually |

---

## Network Isolation

Each agent is network-isolated by default and can only reach external services you explicitly permit.

### How It Works

Every agent pod runs with a Kubernetes NetworkPolicy that blocks **all** outbound traffic except to its paired gateway pod. The gateway acts as the sole exit point to the internet.

This means:

- An agent cannot reach arbitrary internet endpoints
- All outbound requests are mediated by the credential gateway
- New destinations require explicit approval (via [Egress Rules](../features/egress-rules.md))

### What This Prevents

- **Data exfiltration** — a compromised agent cannot send your code or data to arbitrary servers
- **Unauthorized API calls** — the agent can't use services you haven't approved
- **Lateral movement** — agents are isolated from each other and from internal infrastructure

### The Approval Flow

When your agent tries to reach a service for the first time:

1. The request hits the gateway
2. The gateway checks if there's a matching egress rule
3. If no rule exists, the request is **held** and you see an approval prompt
4. You decide: **Allow once**, **Allow always** (creates a permanent rule), or **Deny**

---

## Multiplayer Security

When colleagues interact with your agent, they act as themselves — with their own credentials and identity.

### How It Works

When a colleague messages your agent via Slack or Telegram:

1. The platform identifies the colleague by their linked identity
2. A short-lived **fork pod** is spawned for that colleague's turn
3. The fork mounts the **colleague's** credentials (not yours)
4. The fork reads/writes the same workspace files (shared context)
5. When the turn completes, the fork terminates

### What's Shared vs. Private

| Shared (workspace) | Private (per-user) |
|-------------------|-------------------|
| Files in `/workspace` | API tokens and credentials |
| Conversation history | OAuth connections |
| Installed skills | Egress rule approvals |
| Agent memory | |

### Guarantees

- **Your credentials are never exposed** to other users, even when they interact with your agent
- **Their actions use their identity** — if a colleague pushes to GitHub via your agent, the commit is attributed to them
- **Workspace access is full** — colleagues can read all files in the shared workspace (this is by design for collaboration)

---

## FAQ

**Can I see which credentials are connected?**
Yes — the Connections panel shows all active connections and their status.

**What if my token expires?**
OAuth tokens are auto-refreshed. API keys don't expire unless you rotate them manually.

**Can I revoke a credential?**
Yes — disconnect it in the Connections panel. The agent immediately loses access.

**Can a compromised agent steal my tokens?**
No. The token is never present in the agent pod — not in env vars, not on disk, not in memory. NetworkPolicy enforcement at the Kubernetes level makes bypass impossible without cluster-level access.
