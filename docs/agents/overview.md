# Agents Overview

An agent is a running AI assistant with its own persistent workspace, credential access, and configuration.

## What is an Agent Instance?

When you create an agent, the platform provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — `/workspace` and `/home/agent` directories that survive restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the agent can only reach services you explicitly allow

Each agent is independent. You can run multiple agents simultaneously, each with different templates and configurations.

## Agent Lifecycle

```
Created → Running → Hibernated → Running → ...
                                     ↓
                                  Deleted
```

| State | What's happening |
|-------|-----------------|
| **Running** | Pod is active, ready to chat. Uses compute resources. |
| **Hibernated** | Pod scaled to zero. Workspace and configuration preserved. No compute cost. |
| **Waking** | Transitioning from hibernated to running (takes ~20 seconds). |

### Automatic Hibernation

Agents automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a Slack message, the agent wakes automatically.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`/workspace`) | Running processes |
| Home directory (`/home/agent`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Temporary files outside workspace |
| Connections and egress rules | |

## Available Templates

| Template | Harness | Best For |
|----------|---------|----------|
| [Claude Code](claude-code.md) | Anthropic Claude Code | General-purpose coding, debugging, refactoring |
| [Pi Agent](pi-agent.md) | pi.dev | Multi-model coding (GPT-4, Mistral, Gemini, etc.) |
| [Google Workspace](google-workspace.md) | Custom | Email, calendar, drive, and sheets automation |
| [Code Guardian](code-guardian.md) | Claude Code | Automated PR code review with Slack notifications |

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


