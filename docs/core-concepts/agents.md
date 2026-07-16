# Agents Overview

An agent is a running AI assistant with its own persistent workspace, credential access, and configuration.

## What is an Agent Instance?

When you create an agent, the platform provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — `/home/agent/work` directory that survives restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the agent can only reach services you explicitly allow

Each agent is independent. You can run multiple agents simultaneously, each with different templates and configurations.

## Agent Lifecycle

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

Agents automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a Slack message, the agent wakes automatically.

### Keeping an Agent Awake

For long-running background workloads that must never be paused, you can disable automatic hibernation:

- **In the UI** — turn on the **Keep awake (never hibernate)** toggle when creating an agent (image step) or later from its settings.
- **In a custom image** — set the `PLATFORM_KEEP_AWAKE` environment variable (accepts `1`, `true`, `yes`, or `on`).

While keep-awake is enabled the agent always reports itself as busy, so it stays running and continues to use compute resources (and billing) even when idle.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`/home/agent/work`) | Running processes |
| Home directory (`/home/agent`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Temporary files outside workspace |
| Connections and egress rules | |

## Available Harnesses

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring |
| Pi Agent | Multi-model coding (GPT-4, Mistral, Gemini, etc.) |
| Bob | General-purpose AI shell with tenant scoping |
| Codex | Coding with OpenAI models or compatible endpoints |
| Nous | Hypothesis-driven experimentation campaigns (experimental — Alpha) |
| OpenEvolve | Evolutionary code optimization — runs the OpenEvolve loop on the Claude Code base (experimental — Alpha) |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Managing Agents

### Create

Click **"New Agent"** → pick a template → name it → create. The agent starts immediately.

### Configure

From the agent's settings panel:

- **Environment variables** — set `GITHUB_REPO`, model preferences, etc.
- **Connections** — link GitHub, Anthropic, Google credentials
- **Egress rules** — control which external services the agent can reach
- **Skills** — install or remove capabilities
- **Allowed users** — control who can interact via Slack

### Delete

Deleting an agent permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting an agent destroys all workspace files. If you have important work, download or push it to a remote repository first."

