# Network Access & Approvals

Sandboxes can only reach external services you explicitly allow. Everything else is blocked or requires your approval.

## Egress Rules

Each sandbox has an allowlist of external services it can reach.

| Field | Description | Example |
|-------|-------------|---------|
| Host | Domain the sandbox can reach, matched exactly — a subdomain needs its own rule. Add `:port` for a service that isn't on 443 | `api.github.com`, `api.prod.example:6443` |
| Method | HTTP method (defaults to `*`, meaning all) | `GET`, `POST` |
| Path | URL path pattern (defaults to `*`) | `/repos/*` |
| Verdict | `allow` or `deny`. The most specific rule wins, so a deny rule can carve an exception out of a broad allow | `allow` |

!!! note "Narrowing a rule restarts the gateway"
    A host-only rule is cheap. Adding a method, path, or port means the sandbox's network gateway has
    to inspect that host's traffic more closely, which briefly restarts the gateway **for that
    sandbox only**. Removing the last narrowed rule for a host reverses it.

### How Rules Are Created

1. **From connections** — connecting GitHub automatically allows `api.github.com`, `github.com`, and `raw.githubusercontent.com`. Connecting GitHub Enterprise allows the host you specify plus its `api.<host>`.
2. **From approvals** — clicking "Allow permanently" on an approval prompt creates a rule
3. **Manually** — add rules directly under **Network access** in the sandbox's settings

### Presets

Rather than writing rules one at a time, pick a preset:

| Preset | What it does |
|--------|--------------|
| **Trusted defaults** | Allows GitHub, npm, PyPI, Anthropic and other common hosts. The usual starting point. |
| **Strict default-deny** | Adds nothing — every request surfaces in your Inbox until you allow it. |
| **Allow everything** | A development escape hatch. Narrow it back with deny rules. |

Switching presets replaces the previous preset's rules. Rules you added by hand, and rules that came
from a connection, are left alone.

## Approvals

When a sandbox tries to reach a service not in its allowlist:

1. The request is **held** (paused, not rejected)
2. You see an approval prompt in the chat UI or Inbox
3. You choose a verdict:

| Verdict | Effect |
|---------|--------|
| **Allow once** | This request proceeds; future requests prompt again |
| **Always allow this request** (**Allow permanently** in the Inbox) | Creates a lasting egress rule for that host/method/path |
| **Always allow this host** (**Allow &lt;host&gt;** in the Inbox) | Creates a host-level rule (all methods/paths on that host). Offered on network prompts only — tool-call prompts have no host |
| **Deny once** (**Dismiss** in the Inbox) | Blocks just this request; future requests prompt again |
| **Always deny this request** (**Deny forever** in the Inbox) | Creates a lasting deny rule; this and every future matching request fail |

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

- A host you've explicitly denied fails right away. A host with no rule at all doesn't fail — it
  waits for you (see [Approvals](#approvals)).
- If the approval system itself is unreachable, the request is refused **without** raising a prompt.
- An expired hold is denied, not allowed.

## The Inbox

The Inbox shows all pending approval requests across your sandboxes. Check it when a sandbox seems stuck — it may be waiting for your approval.

!!! note
    In the left sidebar this is currently the **Email** icon — the badge on it counts pending
    approvals.
