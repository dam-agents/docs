# Agents

A **sandbox** is a running AI agent with its own persistent workspace, credential access, and configuration. It's the core thing you create and work with on the platform, and it's isolated from every other sandbox — its own compute pod, workspace, and network boundary.

## What You Get

When you create a sandbox, DAM provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — a `~/work` directory that survives restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the agent can only reach services you explicitly allow

Each sandbox is independent. You can run several at once, each with a different coding harness and configuration.

## Lifecycle

```
Starting → Running → Hibernating → Hibernated → Starting → ...
               │                                     ↑
    Pause ─────┤                                     │
    Stop ──────┴──────────────→ Stopped ─── Start ───┘
                                                     ↓
                                                   Error
```

| State | What's happening |
|-------|-----------------|
| **Starting** | Pod is scaling up from hibernated, stopped, or newly created. |
| **Preparing workspace** | A one-time workspace setup step is still running. The pod is already up — it just isn't usable until that finishes. |
| **Running** | Pod is active, ready to chat. Reserves compute against your budget. |
| **Hibernating** | Pod is scaling down — the short window after you pause or stop it, before it reports down. |
| **Hibernated** | Pod scaled to zero after being idle. Workspace and configuration preserved. No compute cost. Wakes automatically on next use. |
| **Paused** | You put it to sleep now instead of waiting for the idle timeout. Frees its budget immediately, and wakes automatically on next use — like an on-demand hibernate. |
| **Stopped** | A hard stop. Stays down and **does not wake on its own** — you must **Start** it (a scheduled run will also start it). Frees its budget. |
| **Over budget** | Can't start yet because your compute budget is full. It waits; a deliberate start — the **Start** button, opening the sandbox, or a schedule fire — retries the gate once other sandboxes free up room. A never-hibernate sandbox (timeout `0`) is the exception: it starts itself as soon as room frees. See [Compute budget](#compute-budget-and-size). |
| **Error** | Pod failed to start or encountered a fatal issue. |

!!! note "The status pill collapses some of these"
    Hibernating, hibernated, paused, and stopped all show as **Hibernating**, and a sandbox waiting
    on budget shows as **Over budget** (with the CPU/memory figures in its tooltip). The states above
    are the underlying model, not nine distinct badges. **Preparing workspace** sits inside Running
    rather than before it, which is why it isn't a step in the diagram above.

**Pause vs. Stop vs. Hibernate.** All three take a sandbox down and free its budget, but they differ in how it comes back: a **hibernated** or **paused** sandbox wakes automatically the next time you use it (hibernate happens on its own after the idle timeout; pause is you doing it now), while a **stopped** sandbox stays down until you explicitly start it or a schedule fires. One exception: pausing a sandbox set to never hibernate behaves like **Stop** — it stays down until you start it.

### Automatic Hibernation

Sandboxes automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the sandbox wakes automatically.

A sandbox stays awake while the agent is **busy** — working a turn, with prompts queued, or waiting on
an approval — and also while any chat, terminal, or SSH connection is simply **open**, even with
nothing running. It only scales down once it's both idle and unconnected, so closing the tab, terminal,
or editor is what lets it idle down.

By default a sandbox hibernates after the DAM-wide idle timeout. You can **override this per sandbox** from its settings (the idle timeout is set in minutes): raise or lower it to fit how the sandbox is used, or set it to **0 to never hibernate** so it stays running. The field always shows the value in force, so a sandbox with no override displays the inherited default rather than a blank.

!!! warning "A sandbox that never hibernates holds its reserved compute even while idle, keeping that room away from your other sandboxes. Use `0` only when you genuinely need one always on."

### Background work and hibernation

**Background work the agent is running holds the sandbox awake by itself.** When an agent starts a
job in the background, it reports that work, and while anything is outstanding the sandbox won't
hibernate underneath it — so closing the tab no longer kills what it left running. Nothing times that
hold out: a sandbox with reported work stays up until the work ends. A **Pause** or **Stop** still
takes it down regardless, so those remain the way to reclaim one deliberately.

That covers work the agent's harness supervises, which is the usual case. A job **detached** from the
harness (`setsid`, `nohup`) is invisible to it, so it isn't holding the sandbox awake — hibernation is
what will eventually end it. For those, either:

- **Keep a terminal or SSH session open** for the duration — an open connection pins the sandbox
  awake, and closing the tab or editor is what lets it idle down again.
- **Raise the hibernation timeout**, or set it to `0` to never hibernate, before starting the job.
  Raising it only defers hibernation rather than preventing it.

Closing a chat tab doesn't stop a turn either — a running turn keeps the sandbox awake until it
finishes.

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

Each sandbox has a **Size** — how much CPU and memory it can use while it's running. You choose the Size when you create a sandbox and can change it later in settings; each catalog image starts from a sensible default.

Your Sizes draw against a per-user **compute budget** — a ceiling on how much CPU and memory your *running* sandboxes can reserve at once. A sandbox reserves its Size against the budget only while it's up; pausing, stopping, or hibernating it frees that room immediately. If the Size you pick is larger than the room currently free, the wizard warns you and names the free CPU and memory — the sandbox still saves, it just waits parked until room frees.

- If a new or resized sandbox would exceed your budget, it can't start and shows as **Over budget** — it waits until other sandboxes free up room, and **Start** is its retry.
- To make room, **pause** or **stop** a sandbox you aren't using, then start the waiting one. DAM will also free room for you where it can: starting a sandbox that doesn't fit hibernates your own idle, unattended sandboxes — longest-idle first, and only enough to fit — once they've been idle a few minutes, without waiting for their own timeout. Sandboxes you have open, and any set to never hibernate, are never touched.
- Changing the Size of a running sandbox restarts it on save; if it's down, the new Size applies the next time it starts. Growing a **running** sandbox past your budget is refused when you save — free room first.

## Available Coding Harnesses

Each sandbox runs one **coding harness** — the AI software inside the pod:

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring (Anthropic) |
| Codex | OpenAI Codex coding agent |
| PI Agent | Pi coding agent with multi-LLM support |
| IBM Bob | Bob shell agent (Anthropic, Mistral, Granite) |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Managing sandboxes

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
**Learn more** link, and some surface a setup note on the next step (e.g. a connection they need).

!!! note "The Alpha badge"
    Specialized images are early and evolving — each is badged **Alpha**. Expect rough edges.

### Configure

From the sandbox's page, the left nav has these sections:

- **Sandbox Setup** — name, provider, **Model settings** (see
  [Sessions](sessions.md#model-settings)), **Network access** (which external services it may reach),
  **Lifecycle** (hibernation timeout — in minutes; `0` = never), **Environment** variables, and
  **Size** (CPU/memory while running, within your [compute budget](#compute-budget-and-size);
  changing it restarts a running sandbox on save). The image is fixed at creation — the only change
  to it is the upgrade below.
- **Connections** — link GitHub, API keys, and other credentials
- **Channels** — connect the sandbox to Slack channels or Telegram chats (see
  [Slack](../integrations/slack.md) and [Telegram](../integrations/telegram.md))
- **Skills** — install or remove capabilities
- **Schedules** — run recurring tasks
- **Artifacts** — files the sandbox has published
- **Usage** — this sandbox's model spend for a calendar month, broken down by model and by day

### Open in

Configure a sandbox once, then launch it however you like from the **Open in** menu on its page:

| Option | What it does |
|--------|--------------|
| **Chat (browser)** | Opens the chat interface in the browser |
| **Terminal (browser)** | Opens the harness terminal (PTY) in the browser |
| **Terminal (local)** | Shows a copyable `dam` command to attach from your own terminal |
| **VS Code / Zed (local)** | Shows a copyable command to open the workspace over SSH — see the [VS Code/Zed (SSH) guide](../guides/vs-code-zed-ssh.md) |

### Pause, Stop & Start

From the sandbox's actions menu you can take it down without deleting it:

- **Pause — wakes on next use** — puts it to sleep now and frees its budget.
- **Stop — until started again** — a hard stop; it stays down until you bring it back up (or a scheduled run fires). Use this to reliably free budget for other work.
- **Wake** / **Start** — brings a sandbox back up; the label depends on where you are. Opening a sandbox that's down offers **Start agent**. In the sandboxes list a parked sandbox offers *Start* (which retries the budget gate) and any other sandbox that's down offers *Wake*; the sandbox's own actions menu always reads *Wake*.
- **Restart** — offered for a running or errored sandbox, and the only power action for those states. It recreates the sandbox on its current settings; the workspace survives. Reach for it when one is wedged.

All of these preserve the workspace and configuration exactly like hibernation — see [What Survives Hibernation](#what-survives-hibernation).

### Updating the image

When the template a sandbox was created from moves on, its page shows an **Update available** badge
and an **Upgrade** button under the Image field. Applying it is always explicit, changes only the
image, and restarts the sandbox if it's running.

### Delete

Deleting a sandbox permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting a sandbox destroys all workspace files. If you have important work, download it or push it to a remote repository first."
