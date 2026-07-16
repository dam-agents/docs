# Agents

An **agent** is a running AI assistant with its own persistent workspace, credential access, and configuration. It's the core thing you create and work with on the platform. Each agent runs in its own isolated **sandbox** — the dedicated environment (compute pod, workspace, and network boundary) that keeps it separate from every other agent.

## What You Get

When you create an agent, the platform provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — a `~/work` directory that survives restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the agent can only reach services you explicitly allow

Each agent is independent. You can run multiple agents at once, each with a different coding harness and configuration.

## Lifecycle

```
Starting → Preparing workspace → Running → Hibernating → Hibernated → Starting → ...
                                     │                                     ↑
                          Pause ─────┤                                     │
                          Stop ──────┴──────────────→ Stopped ─── Start ───┘
                                                                           ↓
                                                                         Error
```

| State | What's happening |
|-------|-----------------|
| **Starting** | Pod is scaling up from hibernated, stopped, or newly created. |
| **Preparing workspace** | Pod is setting up its persistent storage and workspace (typically on first start). |
| **Running** | Pod is active, ready to chat. Reserves compute against your budget. |
| **Hibernating** | Pod is scaling down after inactivity. |
| **Hibernated** | Pod scaled to zero after being idle. Workspace and configuration preserved. No compute cost. Wakes automatically on next use. |
| **Paused** | You put it to sleep now instead of waiting for the idle timeout. Frees its budget immediately, and wakes automatically on next use — like an on-demand hibernate. |
| **Stopped** | A hard stop. Stays down and **does not wake on its own** — you must **Start** it (a scheduled run will also start it). Frees its budget. |
| **Parked** | Can't start yet because your compute budget is full. It waits; **Start** is its retry once other agents free up room. See [Compute budget](#compute-budget-and-size). |
| **Error** | Pod failed to start or encountered a fatal issue. |

**Pause vs. Stop vs. Hibernate.** All three take an agent down and free its budget, but they differ in how it comes back: a **hibernated** or **paused** agent wakes automatically the next time you use it (hibernate happens on its own after the idle timeout; pause is you doing it now), while a **stopped** agent stays down until you explicitly Start it or a schedule fires.

### Automatic Hibernation

Agents automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the agent wakes automatically.

An agent that is **busy** — actively working a turn, or running a session — is never hibernated by the idle checker; it only scales down once it goes idle.

By default an agent hibernates after the platform-wide idle timeout. You can **override this per agent** from its settings (the idle timeout is set in minutes): raise or lower it to fit how the agent is used, or set it to **0 to never hibernate** so the agent stays running. Leaving the override unset inherits the platform default.

!!! warning "An agent that never hibernates keeps consuming compute (and billing) even while idle. Use `0` only when you genuinely need an always-on agent."

### Keep Awake

Some work keeps running with **no** active chat or terminal session — a long
background job, for example — and the idle checker would otherwise hibernate the
agent out from under it. **Keep awake** prevents that.

- **Manual toggle.** In the agent's settings, under **Lifecycle**, switch
  **Keep awake** on and the agent **never hibernates** — it keeps running, and
  consuming resources, until you turn the toggle off or stop the agent.
- **Agent-held leases.** Your agent can hold itself awake for the duration of
  a background workload using built-in tools — no manual toggle needed:

    | Tool | What it does |
    |------|--------------|
    | `keep_awake_acquire` | Take a keep-awake lease under an `id` you choose (typically a job/run id). Optional `value` = minutes of idleness to tolerate while held (`0`/omitted = never hibernate). |
    | `keep_awake_release` | Release a lease by `id`. Idempotent. When the last lease is released, normal idle-hibernation resumes. |
    | `keep_awake_purge` | Drop all leases at once. |

    !!! warning "Leases don't expire on their own"
        A lease keeps the agent awake until it is explicitly released (or purged).
        Always release a lease when its work finishes, or the agent may run
        indefinitely. While any lease is held, turning the manual toggle off may let
        the agent hibernate and interrupt that work — the UI warns you before
        doing so.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`~/work`) | Running processes |
| Home directory (`$HOME`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Temporary files outside the workspace |
| Connections and egress rules | |

## Compute budget and Size

Each agent has a **Size** — how much CPU and memory it can use while it's running. You choose the Size when you create an agent and can change it later in settings; each catalog image starts from a sensible default.

Your Sizes draw against a per-user **compute budget** — a ceiling on how much CPU and memory your *running* agents can reserve at once. An agent reserves its Size against the budget only while it's up; pausing, stopping, or hibernating it frees that room immediately. As you pick a Size, the create wizard shows how much of your budget is still free.

- If a new or resized agent would exceed your budget, it can't start and shows as **Parked** — it waits until other agents free up room, and **Start** is its retry.
- To make room, **pause** or **stop** an agent you aren't using, then start the parked one.
- Changing the Size of a running agent restarts it on save; if it's down, the new Size applies the next time it starts.

## Available Coding Harnesses

An agent runs one **coding harness** — the AI software inside the pod:

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring |
| Pi Agent | Multi-model coding (GPT, Mistral, Gemini, etc.) |
| Bob | General-purpose AI shell with tenant scoping |
| Codex | Coding with OpenAI models or compatible endpoints |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Ephemeral Runs (`dam-run`)

From inside an agent — a web terminal, an SSH session, or the agent itself — you can run a single command in a **fresh, separate pod** with `dam-run`:

```bash
dam-run <command> [args...]
```

The executor pod is a throwaway copy of your agent: it uses the **same image, configuration, and persistent `~/home/agent` workspace**, so the command sees your files and credentials exactly as the main pod does. Its input and output are streamed back over a PTY, so it behaves like a local invocation (including interactive TTY programs). The executor pod exists only for that one command — it is **torn down as soon as `dam-run` exits**, and disconnecting reaps it automatically.

Use it to run something heavy, long, or risky off to the side without tying up — or disturbing — your main agent.

!!! note "Limits"
    Each agent may have up to **16 concurrent `dam-run` executors** at once; beyond that, new runs are rejected until one finishes. A single run is also capped at roughly **one hour** — for longer-lived work, use a dedicated agent instead.

## Managing Agents

### Create

Click **"Create a sandbox"** → choose a starting image, name it, choose a provider, pick a **Size** (CPU/memory, within your budget), and set network permissions → create. The agent starts immediately.

The image step groups options into **Coding agents**, **Custom images**, and **Pre-configured images**. Each catalog entry shows a display name and tags (e.g. the providers it supports), and some include a **Learn more** link to their docs. A pre-configured image may surface a setup note callout on the setup step (e.g. a recommended model).

### Configure

From the agent's settings panel:

- **Environment variables** — set `GITHUB_REPO`, model preferences, etc.
- **Connections** — link GitHub, API keys, and other credentials
- **Egress rules** — control which external services the agent can reach
- **Skills** — install or remove capabilities
- **Channels** — connect Slack or Telegram, and control who's allowed to interact
- **Size** — adjust the CPU/memory this agent uses while running (within your [compute budget](#compute-budget-and-size)); changing it restarts a running agent on save
- **Hibernation timeout** — override how long this agent stays idle before hibernating (in minutes; `0` = never), or inherit the platform default

### Open in

Configure an agent once, then launch it however you like from the **Open in** menu on the agent's page:

| Option | What it does |
|--------|--------------|
| **Chat (browser)** | Opens the chat interface in the browser |
| **Terminal (browser)** | Opens the harness terminal (PTY) in the browser |
| **Terminal (local)** | Shows a copyable `dam` command to attach from your own terminal |
| **VS Code / Zed (local)** | Shows a copyable command to open the workspace over SSH — see the [VS Code/Zed (SSH) guide](../guides/vs-code-zed-ssh.md) |

### Pause, Stop & Start

From the agent's actions menu you can take it down without deleting it:

- **Pause** — puts it to sleep now and frees its budget; it wakes automatically the next time you use it.
- **Stop** — a hard stop; it stays down until you **Start** it (or a scheduled run fires). Use this to reliably free budget for other work.
- **Start** — brings a stopped or parked agent back up.

All of these preserve the workspace and configuration exactly like hibernation — see [What Survives Hibernation](#what-survives-hibernation).

### Delete

Deleting an agent permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting an agent destroys all workspace files. If you have important work, download it or push it to a remote repository first."
