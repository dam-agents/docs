# Sandboxes

A **sandbox** is a running AI assistant with its own persistent workspace, credential access, and configuration. It's the core thing you create and work with on the platform.

!!! note "Sandbox = agent"
    "Sandbox" is what the UI calls it. The CLI and API use the older term **agent** (e.g. `dam agent ...`, `agent-` IDs) — they're the same thing. You'll see both across the product.

## What You Get

When you create a sandbox, the platform provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — a `~/work` directory that survives restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the sandbox can only reach services you explicitly allow

Each sandbox is independent. You can run multiple sandboxes at once, each with a different coding agent and configuration.

## Lifecycle

```
Starting → Preparing workspace → Running → Hibernating → Hibernated → Starting → ...
                                                                          ↓
                                                                        Error
```

| State | What's happening |
|-------|-----------------|
| **Starting** | Pod is scaling up from hibernated or newly created. |
| **Preparing workspace** | Pod is setting up its persistent storage and workspace (typically on first start). |
| **Running** | Pod is active, ready to chat. Uses compute resources. |
| **Hibernating** | Pod is scaling down after inactivity. |
| **Hibernated** | Pod scaled to zero. Workspace and configuration preserved. No compute cost. |
| **Error** | Pod failed to start or encountered a fatal issue. |

### Automatic Hibernation

Sandboxes automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the sandbox wakes automatically.

A sandbox that is **busy** — actively working a turn, or running a session — is never hibernated by the idle checker; it only scales down once it goes idle.

By default a sandbox hibernates after the platform-wide idle timeout. You can **override this per sandbox** from its settings (the idle timeout is set in minutes): raise or lower it to fit how the sandbox is used, or set it to **0 to never hibernate** so the sandbox stays running. Leaving the override unset inherits the platform default.

!!! warning "A sandbox that never hibernates keeps consuming compute (and billing) even while idle. Use `0` only when you genuinely need an always-on sandbox."

### Keep Awake

Some work keeps running with **no** active chat or terminal session — a long
background job, for example — and the idle checker would otherwise hibernate the
sandbox out from under it. **Keep awake** prevents that.

- **Manual toggle.** In the sandbox's settings, under **Lifecycle**, switch
  **Keep awake** on and the sandbox **never hibernates** — it keeps running, and
  consuming resources, until you turn the toggle off or stop the sandbox.
- **Agent-held leases.** Your agent can hold the sandbox awake for the duration of
  a background workload using built-in tools — no manual toggle needed:

    | Tool | What it does |
    |------|--------------|
    | `keep_awake_acquire` | Take a keep-awake lease under an `id` you choose (typically a job/run id). Optional `value` = minutes of idleness to tolerate while held (`0`/omitted = never hibernate). |
    | `keep_awake_release` | Release a lease by `id`. Idempotent. When the last lease is released, normal idle-hibernation resumes. |
    | `keep_awake_purge` | Drop all leases at once. |

    !!! warning "Leases don't expire on their own"
        A lease keeps the sandbox awake until it is explicitly released (or purged).
        Always release a lease when its work finishes, or the sandbox may run
        indefinitely. While any lease is held, turning the manual toggle off may let
        the sandbox hibernate and interrupt that work — the UI warns you before
        doing so.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`~/work`) | Running processes |
| Home directory (`$HOME`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Temporary files outside the workspace |
| Connections and egress rules | |

## Available Coding Agents

A sandbox runs one **coding agent** — the AI software inside the pod:

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring |
| Pi Agent | Multi-model coding (GPT, Mistral, Gemini, etc.) |
| Bob | General-purpose AI shell with tenant scoping |
| Codex | Coding with OpenAI models or compatible endpoints |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Ephemeral Runs (`dam-run`)

From inside a sandbox — a web terminal, an SSH session, or the agent itself — you can run a single command in a **fresh, separate pod** with `dam-run`:

```bash
dam-run <command> [args...]
```

The executor pod is a throwaway copy of your sandbox: it uses the **same image, configuration, and persistent `~/home/agent` workspace**, so the command sees your files and credentials exactly as the main pod does. Its input and output are streamed back over a PTY, so it behaves like a local invocation (including interactive TTY programs). The executor pod exists only for that one command — it is **torn down as soon as `dam-run` exits**, and disconnecting reaps it automatically.

Use it to run something heavy, long, or risky off to the side without tying up — or disturbing — your main sandbox.

!!! note "Limits"
    Each sandbox may have up to **16 concurrent `dam-run` executors** at once; beyond that, new runs are rejected until one finishes. A single run is also capped at roughly **one hour** — for longer-lived work, use a dedicated sandbox instead.

## Managing Sandboxes

### Create

Click **"Create a sandbox"** → choose a starting image, name it, choose a provider, and set network permissions → create. The sandbox starts immediately.

The image step groups options into **Coding agents**, **Custom images**, and **Pre-configured images**. Each catalog entry shows a display name and tags (e.g. the providers it supports), and some include a **Learn more** link to their docs. A pre-configured image may surface a setup note callout on the setup step (e.g. a recommended model).

### Configure

From the sandbox's settings panel:

- **Environment variables** — set `GITHUB_REPO`, model preferences, etc.
- **Connections** — link GitHub, API keys, and other credentials
- **Egress rules** — control which external services the sandbox can reach
- **Skills** — install or remove capabilities
- **Channels** — connect Slack or Telegram, and control who's allowed to interact
- **Hibernation timeout** — override how long this sandbox stays idle before hibernating (in minutes; `0` = never), or inherit the platform default

### Delete

Deleting a sandbox permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting a sandbox destroys all workspace files. If you have important work, download it or push it to a remote repository first."
