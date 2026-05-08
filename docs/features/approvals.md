# Approvals

The approval system lets you control what external services your agent can access.

## How It Works

When your agent tries to reach an external service that isn't in its egress allowlist:

1. The request is **held** (paused, not rejected)
2. You see an approval prompt in the chat UI (or your Inbox)
3. You choose: **Allow once**, **Allow always**, or **Deny**
4. The agent continues (or aborts if denied)

## The Inbox

The **Inbox** shows all pending approval requests across your agents. Check it when:

- An agent seems stuck (it may be waiting for approval)
- You want to review and batch-approve multiple requests

## Verdicts

| Verdict | Effect |
|---------|--------|
| **Allow once** | This specific request proceeds; future requests to the same service will prompt again |
| **Allow always** | Creates a permanent [egress rule](egress-rules.md) — the agent never asks again for this service |
| **Deny** | The request is blocked; the agent is informed and can adjust |

## Timeout Behavior

If no one responds to an approval request within **10 minutes**, the request is automatically denied. The agent is informed and can retry or take an alternative approach.

!!! tip "Avoid timeouts"
    Keep the DAM UI or Inbox open during active agent work, or set up Slack notifications so you don't miss approval prompts.

## When Approvals Trigger

- First time reaching a new hostname (e.g., `api.example.com`)
- Requests to hosts without a matching egress rule
- Sensitive tool calls that the agent harness flags for permission
