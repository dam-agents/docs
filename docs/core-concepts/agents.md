# Agents

A **sandbox** is a running AI agent with its own persistent workspace, credential access, and configuration. It's the core thing you create and work with on the platform, and it's isolated from every other sandbox — its own compute pod, workspace, and network boundary.

!!! note "Sandbox and agent are the same thing"
    **Sandbox** is the name you'll see in the UI. **Agent** is the same object's name in the API and
    the CLI (`dam agent …`), and this page uses both.

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
| **Parked** (shown as **Over budget**) | Can't start yet because your compute budget is full. It waits; a deliberate start — the **Start** button, opening the sandbox, or a schedule fire — retries the gate once other agents free up room. A never-hibernate agent (timeout `0`) is the exception: it starts itself as soon as room frees. See [Compute budget](#compute-budget-and-size). |
| **Error** | Pod failed to start or encountered a fatal issue. |

!!! note "The status pill collapses some of these"
    Hibernating, hibernated, paused, and stopped all show as **Hibernating**, and a parked sandbox
    shows as **Over budget** (with the CPU/memory figures in its tooltip). The states above are the
    underlying model, not nine distinct badges.

**Pause vs. Stop vs. Hibernate.** All three take an agent down and free its budget, but they differ in how it comes back: a **hibernated** or **paused** agent wakes automatically the next time you use it (hibernate happens on its own after the idle timeout; pause is you doing it now), while a **stopped** agent stays down until you explicitly start it or a schedule fires.

### Automatic Hibernation

Agents automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the agent wakes automatically.

An agent stays awake while it's **busy** — working a turn, with prompts queued, or waiting on an
approval — and also while any chat, terminal, or SSH connection is simply **open**, even with nothing
running. It only scales down once it's both idle and unconnected, so closing the tab, terminal, or
editor is what lets it idle down.

By default an agent hibernates after the platform-wide idle timeout. You can **override this per agent** from its settings (the idle timeout is set in minutes): raise or lower it to fit how the agent is used, or set it to **0 to never hibernate** so the agent stays running. The field always shows the value in force, so an agent with no override displays the inherited platform default rather than a blank.

!!! warning "An agent that never hibernates keeps consuming compute (and billing) even while idle. Use `0` only when you genuinely need an always-on agent."

### Background work and hibernation

The idle checker only sees **sessions and connections** — chat, terminal, and SSH. Work detached
from a session (a background job that outlives the terminal which started it, an async task) is
invisible to it, so an agent can hibernate mid-job and kill that work.

Two things keep such work alive:

- **Keep a terminal or SSH session open** to the sandbox for the duration — an open connection
  pins it awake.
- **Raise the hibernation timeout**, or set it to `0` to never hibernate, before starting the job.

Crossing the `0` boundary asks you to confirm in both directions, since it's the difference between
an always-on sandbox and one that may be interrupted.

### What Survives Hibernation

| Persists | Lost |
|----------|------|
| Workspace files (`~/work`) | Running processes |
| Home directory (`$HOME`, except `~/.cache`) | In-memory state |
| Installed skills | Open terminal sessions |
| Session history | Files outside the persisted mounts (e.g. `/tmp`) |
| Connections and egress rules | Packages installed outside `$HOME` (apt, system-level) |

!!! tip "Bake long-term dependencies into an image"
    Anything your agent needs permanently — system packages, tools installed at runtime — should go
    into a [custom image](../guides/custom-image.md), or it will vanish on the next hibernation.

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
| Claude Code | General-purpose coding, debugging, refactoring (Anthropic) |
| Codex | OpenAI Codex coding agent |
| PI Agent | Pi coding agent with multi-LLM support |
| IBM Bob | Bob shell agent (Anthropic, Mistral, Granite) |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## VM sandboxes

Some work a container simply can't do — systemd, Docker or k3s, nested containers, kernel-adjacent
tooling. For that, a sandbox can boot as a **full virtual machine** instead of a container, with
those tools available inside it.

This is a choice you make when you create the sandbox, not a command you run inside one: turn on
**Run as a virtual machine** in the create-sandbox wizard and pick one of the images it reveals —
for example **Claude Code VM**, tagged **VM**.

!!! note "Off until you opt in"
    VM sandboxes ride the **VM sandboxes** toggle under **Experimental features** (see
    [Configure](#configure)), which is off by default — until you enable it, the wizard switch and
    the VM images stay hidden entirely. Your operator must also enable virtualization on the
    install, so the option can be unavailable even with the toggle on. These images are **Alpha**.

## Managing Agents

### Create

Click **Create sandbox**. Step 1 asks what the sandbox is *for* rather than which image to boot:

| Starting point | What it gives you |
|----------------|-------------------|
| **Experiment sandbox** | Claude Code with the experiment authoring skill — runs one goal across several variants and charts each result live |
| **Knowledge base sandbox** | Claude Code that builds and maintains a body of knowledge you can chat with |
| **Specialized sandbox** | An image already set up for one task |
| **General-purpose sandbox** | Pick a coding harness and customize it yourself |
| **Custom image** (*Advanced*) | Bring your own image address |

Experiment and knowledge-base sandboxes need an image your operator has to enable — the wizard says
so if it's missing. Step 2 (**Setup your sandbox**) names it, picks a provider and a **Size**
(CPU/memory, within your budget), and sets network permissions; step 3 grants connections.

Catalog entries show a display name and tags (e.g. the providers they support); some carry a
**Learn more** link, and some surface a setup note on the next step (e.g. a recommended model).

!!! note "The Alpha badge"
    Some images are marked **Alpha** — early and evolving, and sorted to the bottom of the list.
    Expect rough edges.

### Configure

From the sandbox's page, the left nav has six sections:

- **Sandbox Setup** — name, image, provider, **Network access** (which external services it may
  reach), **Lifecycle** (hibernation timeout — in minutes; `0` = never), **Environment** variables,
  and **Size** (CPU/memory while running, within your [compute budget](#compute-budget-and-size);
  changing it restarts a running sandbox on save)
- **Connections** — link GitHub, API keys, and other credentials
- **Channels** — connect the sandbox to a [Slack](../integrations/slack.md) conversation or
  [Telegram](../integrations/telegram.md) chat, and set ambient mode
- **Skills** — install or remove capabilities
- **Schedules** — run recurring tasks
- **Artifacts** — files the sandbox has published

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

- **Pause — wakes on next use** — puts it to sleep now and frees its budget.
- **Stop — until started again** — a hard stop; it stays down until you bring it back up (or a scheduled run fires). Use this to reliably free budget for other work.
- **Wake** / **Start** — brings a sandbox back up. The menu says *Wake* for a hibernated or paused sandbox, and *Start* for a parked one, where it retries the budget gate.

All of these preserve the workspace and configuration exactly like hibernation — see [What Survives Hibernation](#what-survives-hibernation).

### Delete

Deleting an agent permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting an agent destroys all workspace files. If you have important work, download it or push it to a remote repository first."
