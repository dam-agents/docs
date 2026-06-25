# Network Access & Approvals

Agents can only reach external services you explicitly allow. Everything else is blocked or requires your approval.

## Egress Rules

Each agent has an allowlist of external services it can reach.

| Field | Description | Example |
|-------|-------------|---------|
| Host | Domain the agent can reach | `api.github.com` |
| Method | HTTP method (optional, defaults to all) | `GET`, `POST` |
| Path | URL path pattern (optional) | `/repos/*` |

### How Rules Are Created

1. **From connections** — connecting GitHub automatically allows `api.github.com`, `github.com`, and `raw.githubusercontent.com`. Connecting GitHub Enterprise allows the host you specify plus its `api.<host>`.
2. **From approvals** — clicking "Allow permanently" on an approval prompt creates a rule
3. **Manually** — add rules directly in the Egress Rules panel

## Approvals

When your agent tries to reach a service not in its allowlist:

1. The request is **held** (paused, not rejected)
2. You see an approval prompt in the chat UI or Inbox
3. You choose a verdict:

| Verdict | Effect |
|---------|--------|
| **Allow once** | This request proceeds; future requests prompt again |
| **Allow permanently** | Creates a lasting egress rule for that host/method/path |
| **Allow `<host>`** | Creates a host-level rule (all methods/paths on that host) |
| **Deny forever** | Creates a lasting deny rule; the agent is informed |
| **Dismiss** | Blocks just this request; future requests prompt again |

### Timeout

If no one responds within **30 minutes**, the request is automatically denied. The agent can retry or take an alternative approach.

!!! tip "Avoid timeouts"
    Keep the DAM UI open during active agent work, or set up Slack notifications for approval prompts.

## The Inbox

The Inbox shows all pending approval requests across your agents. Check it when an agent seems stuck — it may be waiting for your approval.
