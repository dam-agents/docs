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
