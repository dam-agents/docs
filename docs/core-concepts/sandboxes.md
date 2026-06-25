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

Each sandbox is independent. You can run multiple sandboxes at once, each with different templates and configurations.

## Lifecycle

```
Created → Starting → Running → Hibernating → Hibernated → Starting → Running → ...
                                                                        ↓
                                                                     Deleted
```

| State | What's happening |
|-------|-----------------|
| **Starting** | Pod is scaling up from hibernated or newly created. |
| **Running** | Pod is active, ready to chat. Uses compute resources. |
| **Hibernating** | Pod is scaling down after inactivity. |
| **Hibernated** | Pod scaled to zero. Workspace and configuration preserved. No compute cost. |
| **Error** | Pod failed to start or encountered a fatal issue. |

### Automatic Hibernation

Sandboxes automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the sandbox wakes automatically.

### Keeping a Sandbox Awake

For long-running background workloads that must never be paused, you can disable automatic hibernation:

- **In the UI** — turn on the **Keep awake (never hibernate)** toggle when creating a sandbox (image step) or later from its settings.
- **In a custom image** — set the `PLATFORM_KEEP_AWAKE` environment variable (accepts `1`, `true`, `yes`, or `on`).

While keep-awake is enabled the sandbox always reports itself as busy, so it stays running and continues to use compute resources (and billing) even when idle.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`~/work`) | Running processes |
| Home directory (`$HOME`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Temporary files outside the workspace |
| Connections and egress rules | |

## Available Harnesses

A sandbox runs one **harness** — the AI agent software inside the pod:

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring |
| Pi Agent | Multi-model coding (GPT, Mistral, Gemini, etc.) |
| Bob | General-purpose AI shell with tenant scoping |
| Codex | Coding with OpenAI models or compatible endpoints |
| Nous | Knowledge and campaign workflows |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Managing Sandboxes

### Create

Click **"Create a sandbox"** → name it, choose a provider, and set network permissions → create. The sandbox starts immediately.

### Configure

From the sandbox's settings panel:

- **Environment variables** — set `GITHUB_REPO`, model preferences, etc.
- **Connections** — link GitHub, API keys, and other credentials
- **Egress rules** — control which external services the sandbox can reach
- **Skills** — install or remove capabilities
- **Channels** — connect Slack or Telegram, and control who's allowed to interact

### Delete

Deleting a sandbox permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting a sandbox destroys all workspace files. If you have important work, download it or push it to a remote repository first."
