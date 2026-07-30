# Network Access & Approvals

Sandboxes can only reach external services you explicitly allow. Everything else is blocked or requires your approval.

## Egress Rules

Each sandbox has an allowlist of external services it can reach.

| Field | Description | Example |
|-------|-------------|---------|
| Host | Domain the sandbox can reach; `*.` wildcards allowed | `api.github.com`, `*.example.com` |
| Method | HTTP method (defaults to `*`, meaning all) | `GET`, `POST` |
| Path | URL path pattern (defaults to `*`) | `/repos/*` |
| Port | Upstream port, when the service isn't on 443 | `6443` |

!!! note "Narrowing a rule restarts the gateway"
    A host-only rule is cheap. Adding a method, path, or port means the sandbox's network gateway has
    to inspect that host's traffic more closely, which briefly restarts the gateway **for that
    sandbox only**. Removing the last narrowed rule for a host reverses it.

### How Rules Are Created

1. **From connections** — connecting GitHub automatically allows `api.github.com`, `github.com`, and `raw.githubusercontent.com`. Connecting GitHub Enterprise allows the host you specify plus its `api.<host>`.
2. **From approvals** — clicking "Allow permanently" on an approval prompt creates a rule
3. **Manually** — add rules directly in the Egress Rules panel

## Approvals

When a sandbox tries to reach a service not in its allowlist:

1. The request is **held** (paused, not rejected)
2. You see an approval prompt in the chat UI or Inbox
3. You choose a verdict:

| Verdict | Effect |
|---------|--------|
| **Allow once** | This request proceeds; future requests prompt again |
| **Always allow this request** | Creates a lasting egress rule for that host/method/path |
| **Always allow this host** | Creates a host-level rule (all methods/paths on that host). Offered when the prompt suits a host-level rule |
| **Deny once** (**Dismiss** in the Inbox) | Blocks just this request; future requests prompt again |
| **Always deny this request** | Creates a lasting deny rule; this and every future matching request fail |

**Customize…** opens the sandbox's settings so you can write the rule yourself.

!!! warning "From the CLI, the bare verb is durable"
    `dam approval approve <id>` **writes a permanent allow rule** for a network request — the
    equivalent of *Always allow this request*. Pass `--once` if you only mean to release the held
    call. Tool-call approvals never write an egress rule. See the
    [CLI reference](../reference/cli.md#approvals).

### Timeout

If no one responds within **30 minutes** (your operator can change this), the request is automatically denied. The agent can retry or take an alternative approach.

!!! tip "If a prompt timed out"
    The original request has already failed, so **Allow once** no longer does anything. Choosing
    **Always allow this request** still writes the rule, and the sandbox succeeds on its next attempt.

### When things fail

The default is always deny:

- A host that isn't allowed fails immediately — name resolution happens in the gateway, so it doesn't hang.
- If the approval system itself is unreachable, the request is refused **without** raising a prompt.
- An expired hold is denied, not allowed.

## The Inbox

The Inbox shows all pending approval requests across your sandboxes. Check it when a sandbox seems stuck — it may be waiting for your approval.
