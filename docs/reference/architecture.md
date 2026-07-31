# Architecture

DAM is designed so that even a fully compromised agent cannot leak your credentials or exfiltrate your data.

## Credential Isolation

Your tokens and API keys are never visible to your agent — they're injected on the network wire by a separate secure gateway.

### The Gateway Model

Every agent has a **paired gateway pod** — a separate, hardened container that handles credential injection:

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

- **Per-user:** Your credentials are yours alone — no other team member can read them or reuse them directly. Note that binding an agent to a channel in **shared** mode lets everyone in that conversation drive the agent *using* those credentials; see [Agents in a Channel](#agents-in-a-channel).
- **Per-service:** Each connected service (GitHub, Anthropic, etc.) is stored separately.
- **All your agents:** Any agent you own can use your connected credentials.

### Token Management

| Type | Storage | Refresh |
|------|---------|---------|
| **OAuth** (GitHub, etc.) | Kubernetes Secret in gateway pod | Auto-refreshed by the platform |
| **API Key** (Anthropic, etc.) | Kubernetes Secret in gateway pod | No expiry unless you rotate manually |

---

## Network Isolation

Each agent is network-isolated by default and can only reach external services you explicitly permit.

### How It Works

Every agent pod runs with a Kubernetes NetworkPolicy that blocks **all** outbound traffic except to its paired gateway pod. The gateway acts as the sole exit point to the internet.

This means:

- An agent cannot reach arbitrary internet endpoints
- All outbound requests are mediated by the credential gateway
- New destinations require explicit approval (via [Network & Approvals](../core-concepts/network-access.md))

### What This Prevents

- **Data exfiltration** — a compromised agent cannot send your code or data to arbitrary servers
- **Unauthorized API calls** — the agent can't use services you haven't approved
- **Lateral movement** — agents are isolated from each other and from internal infrastructure

### The Approval Flow

When your agent tries to reach a service for the first time:

1. The request hits the gateway
2. The gateway checks if there's a matching egress rule
3. If no rule exists, the request is **held** and you see an approval prompt
4. You decide: **Allow once** (releases just this call), **Always allow this request** (a lasting rule for that host/method/path), **Always allow this host** (a host-level rule), **Deny once**, or **Always deny this request**. **Customize…** opens the sandbox's settings to write the rule yourself.

From the CLI the same choices are `dam approval approve` / `deny` — note that the **bare** verb is durable and writes a permanent rule, while `--once` actions only the held call. See the [CLI reference](cli.md#network-access-and-approvals).

---

## Agents in a Channel

Binding a conversation to an agent is what authorizes it. There is one access model, and it is the same on both messengers:

1. The binding itself is the authorization — anyone the messenger admits to the conversation drives the agent, with no login and no allow-list
2. Every turn relays to the agent's main pod under the **agent's own** credentials
3. The sender is recorded by Slack user id and labelled in the prompt, so a multi-person conversation stays attributable (a 1:1 DM isn't labelled — there's only one human in it)
4. The **binding owner's** Terms of Use acceptance gates every turn — the terms bind the party whose credentials run the work, not whoever typed

Linking your messenger identity (`/dam login`) is never needed to drive a turn. It authorizes only the in-chat management commands — `bind`, `unbind`, and `ambient`.

Telegram works the same way: with no workspace to anchor per-user identity, consent attaches to the conversation.

A Slack conversation and an agent are strictly one-to-one: a conversation binds to exactly one agent, and an agent to at most one Slack conversation.

### What This Means for Access

| Governed by the messenger | Governed by the agent |
|---------------------------|----------------------|
| Who can see and post in the conversation | Which credentials and connections the turn uses |
| Adding or removing participants | Which files and skills are available |
| — | Approval and egress rules |

### Consequences

- **Everyone in the conversation acts with the agent's authority.** A colleague who pushes to GitHub through it does so with the agent's token, attributed to the agent's identity — not their own. Treat a bound channel as equivalent to sharing the agent's credentials with the room.
- **Conversation membership is the access control.** The messenger owns who is in the room; the platform never resolves who is typing to decide whether a turn may run.
- **Ambient mode is a deliberate opt-in**, off by default on every connect path, because it has the agent read every message in the channel rather than only the ones aimed at it.

