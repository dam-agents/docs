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

- **Per-user:** Your credentials are yours alone. No other team member can use or see them.
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
4. You decide: **Allow once** (releases just this call), **Allow permanently** (a lasting rule for that host/method/path), **Allow `<host>`** (a host-level rule), or **Deny forever**. You can also **Dismiss** (block this request; future ones prompt again) or **Customize** the rule directly.

---

## Shared Agents in a Channel

When you bind an agent to a Slack conversation, **the binding is the authorization**: anyone Slack admits to that conversation can drive the agent, and every turn runs on the agent's own pod under the **agent's own** credentials.

### How It Works

When someone messages an agent in a bound conversation:

1. The conversation resolves to exactly one agent
2. The turn runs on that agent's main pod, using the credentials and connections the agent already has
3. The sender is recorded by their Slack user id, so a multi-person conversation stays attributable. Each message is labelled in the prompt with who sent it (a 1:1 DM isn't labelled — there's only one human in it)
4. The **binding owner's** Terms of Use acceptance gates every turn — the terms bind the party whose credentials run the work, not whoever typed

There is no per-person access mode, no identity linking required to send a message, and no allow-list. Identity linking exists only to authorize the in-chat `unbind` and `ambient on|off` commands (and is established during `bind` so the binder can later release it) — never to admit a turn.

A conversation and an agent are strictly one-to-one: a conversation binds to exactly one agent, and an agent to at most one conversation.

### What This Means for Access

| Governed by the messenger | Governed by the agent |
|---------------------------|----------------------|
| Who can see and post in the conversation | Which credentials and connections the turn uses |
| Adding or removing participants | Which files and skills are available |
| — | Approval and egress rules |

### Consequences

- **Everyone in the conversation acts with the agent's authority.** A colleague who pushes to GitHub through a bound agent does so with the agent's token, attributed to the agent's identity — not their own.
- **Channel membership is the access control.** Excluding someone means removing them from the conversation, or unbinding it. Bind deliberately, and treat a bound channel as equivalent to sharing the agent's credentials with the room.
- **Someone who needs to act as themselves needs their own agent.** There is no per-person credential switching within a shared one.
- **Attribution within a binding is messenger-id based.** The security log records each turn's sender id; there is no per-turn platform identity.

!!! note "This replaces the earlier per-person model"
    Slack previously supported a person-scoped mode — identity linking, an allowed-users list, and a per-turn forked pod pair running under the replier's own credentials. It was removed for two reasons: the isolation it offered was incomplete (credentials were split per person, but the workspace was not, so any member could leave instructions the agent later ran under the owner's credentials), and supporting a second concurrent writer forced every agent's storage onto a shared-writable volume — a cost the whole platform paid for a mode almost nobody used.

    Bindings that used it were **deleted** rather than converted: silently switching them to the open model would have widened who could drive an agent without the owner deciding that. Re-binding is the owner's consent. If a channel stopped working after the upgrade, bind it again.

