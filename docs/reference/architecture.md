# Architecture

DAM is designed so that even a fully compromised agent cannot leak your credentials or reach services you haven't approved.

## Credential Isolation

Your tokens and API keys are never visible to your agent — they're injected on the network wire by a separate secure gateway.

### The Gateway Model

Every agent is paired with its own **credential gateway** — a separate, hardened environment that handles credential injection:

```
┌─────────────────────────────────────────────────────┐
│ Your agent                                        │
│                                                     │
│  Agent makes HTTPS request to api.github.com  ──────┼──┐
│  (no credentials attached)                          │  │
└─────────────────────────────────────────────────────┘  │
                                                         │
    ┌────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────┐
│ Credential gateway                                  │
│                                                     │
│  1. Intercepts the request                          │
│  2. Looks up your stored GitHub token               │
│  3. Injects "Authorization: Bearer <token>"         │
│  4. Forwards to api.github.com                      │
└─────────────────────────────────────────────────────┘
```

**Key properties:**

- Your agent has **no access** to the raw token — it never reaches the agent's disk, and where a tool expects an env var the agent gets a placeholder, not the real value
- Your agent has no route to the internet except through the gateway, and that's enforced at the network layer rather than by the agent's own good behavior
- Even if an agent is fully compromised (prompt injection, malicious code), it cannot exfiltrate your credentials

### Credential Scoping

- **Per-user:** Your credentials are yours alone — no other team member can read them or reuse them directly. Note that binding an agent to a channel lets everyone in that conversation drive the agent *using* those credentials; see [Agents in a Channel](#agents-in-a-channel).
- **Per-service:** Each connected service (GitHub, Anthropic, etc.) is stored separately.
- **All your agents:** Any agent you own can be granted your connected credentials.

### Token Management

| Type | Storage | Refresh |
|------|---------|---------|
| **OAuth** (GitHub, etc.) | Held by the gateway, never by your agent | Auto-refreshed for you |
| **API Key** (Anthropic, etc.) | Held by the gateway, never by your agent | No expiry unless you rotate manually |

---

## Network Isolation

Each agent is network-isolated by default and can only reach external services you explicitly permit.

### How It Works

Every agent has **all** outbound traffic blocked except to its own credential gateway, which is the sole exit point to the internet.

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
3. If no rule exists, the request is **held** and the approval appears in **Activity**, opened from the **Activity** button at the top right of every page — unless the turn came from a channel with no session of yours attached, in which case it's refused straight away and recorded there instead (see [Network & Approvals](../core-concepts/network-access.md#approvals))
4. You decide: **Allow once** (releases just this call), **Allow permanently** (a lasting rule for that host/method/path), **Allow all of &lt;host&gt;** (a host-level rule), **Deny this request**, or **Deny permanently**.

From the CLI the same choices are `dam approval approve` / `deny` — note that the **bare** verb is durable and writes a permanent rule, while `--once` actions only the held call. See the [CLI reference](cli.md#approvals).

---

## Agents in a Channel

Binding a conversation to an agent is what authorizes it. There is one access model, and it is the same on both messengers:

1. The binding itself is the authorization — anyone the messenger admits to the conversation drives the agent, with no login and no allow-list
2. Every turn runs on the agent under the **agent's own** credentials
3. The sender is recorded by Slack user id and labelled in the prompt, so a multi-person conversation stays attributable (a 1:1 DM isn't labelled — there's only one human in it)
4. The **binding owner's** Terms of Use acceptance gates every turn — the terms bind the party whose credentials run the work, not whoever typed

Linking your messenger identity (`/dam login`) is never needed to drive a turn, or to whisper to an agent with `/dam whisper`. It authorizes the in-chat management commands — `/dam unbind`, `/dam ambient on|off`, `/dam default <agent>`, and `/dam delete` — and an owner's link is also what lets their agents post into Slack channels beyond their own bindings, which they can do only where that owner is a member. The read-only forms, a bare `/dam ambient` or a bare `/dam default`, need no link either. (`/dam bind` needs no prior link; the bind flow signs you in as part of connecting the agent.)

Telegram works the same way: with no workspace to anchor per-user identity, consent attaches to the conversation.

Bindings are many-to-many on the Slack side: a conversation can hold several agents, and one agent can be connected to several conversations at once, in any mix of channels, group DMs, and DMs. Each pairing is its own binding, released on its own, and the same agent can't join one conversation twice. A conversation with more than one agent has a **default** — the one a mention with no name reaches. A Telegram chat still binds to a single agent.

### What This Means for Access

| Governed by the messenger | Governed by the agent |
|---------------------------|----------------------|
| Who can see and post in the conversation | Which credentials and connections the turn uses |
| Adding or removing participants | Which files and skills are available |
| — | Approval and egress rules |

### Consequences

- **Everyone in the conversation acts with the agent's authority.** A colleague who pushes to GitHub through it does so with the agent's token, attributed to the agent's identity — not their own. Treat a bound channel as equivalent to sharing the agent's credentials with the room.
- **Conversation membership is the access control.** The messenger owns who is in the room; DAM never resolves who is typing to decide whether a turn may run.
- **Ambient mode is a deliberate opt-in**, off by default on every connect path, because it has the agent read every message in the channel rather than only the ones aimed at it.

