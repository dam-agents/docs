# Agents

An **agent** is a running AI assistant with its own persistent workspace, credential access, and configuration. It's the core thing you create and work with on the platform, and it's isolated from every other agent — its own compute pod, workspace, and network boundary.

## What You Get

When you create an agent, DAM provisions:

- **A compute pod** — an isolated Linux environment running the AI harness
- **A persistent workspace** — a `~/work` directory that survives restarts and hibernation
- **A credential gateway** — a paired security pod that injects your tokens on the wire
- **Network isolation** — the agent can only reach services you explicitly allow

Each agent is independent. You can run several at once, each with a different coding harness and configuration.

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
| **Over budget** | Can't start yet because your compute budget is full. It waits; a deliberate start — the **Start** button, opening the agent, or a schedule fire — retries the gate once other agents free up room. A never-hibernate agent (timeout `0`) is the exception: it starts itself as soon as room frees. See [Compute budget](#compute-budget-and-size). |
| **Error** | Pod failed to start or encountered a fatal issue. |

!!! note "The status pill collapses some of these"
    Hibernating, hibernated, paused, and stopped all show as **Hibernating**, and an agent waiting
    on budget shows as **Over budget** (with the CPU/memory figures in its tooltip). The states above
    are the underlying model, not nine distinct badges. **Preparing workspace** sits inside Running
    rather than before it, which is why it isn't a step in the diagram above.

**Pause vs. Stop vs. Hibernate.** All three take an agent down and free its budget, but they differ in how it comes back: a **hibernated** or **paused** agent wakes automatically the next time you use it (hibernate happens on its own after the idle timeout; pause is you doing it now), while a **stopped** agent stays down until you explicitly start it or a schedule fires. One exception: pausing an agent set to never hibernate behaves like **Stop** — it stays down until you start it.

### Automatic Hibernation

Agents automatically hibernate after a period of inactivity (no active sessions, no pending triggers). When you open a chat or send a message from a connected channel, the agent wakes automatically.

An agent stays awake while it is **busy** — working a turn, with prompts queued, or waiting on
an approval — and also while any chat, terminal, or SSH connection is simply **open**, even with
nothing running. It only scales down once it's both idle and unconnected, so closing the tab, terminal,
or editor is what lets it idle down.

By default an agent hibernates after the DAM-wide idle timeout. You can **override this per agent** from its settings (the idle timeout is set in minutes): raise or lower it to fit how the agent is used, or set it to **0 to never hibernate** so it stays running. The field always shows the value in force, so an agent with no override displays the inherited default rather than a blank.

!!! warning "An agent that never hibernates holds its reserved compute even while idle, keeping that room away from your other agents. Use `0` only when you genuinely need one always on."

### Background work and hibernation

**Background work holds the agent awake by itself.** When an agent starts a
job in the background, it reports that work, and while anything is outstanding the agent won't
hibernate underneath it — so closing the tab no longer kills what it left running. Nothing times that
hold out: an agent with reported work stays up until the work ends. A **Pause** or **Stop** still
takes it down regardless, so those remain the way to reclaim one deliberately.

That covers work the agent's harness supervises, which is the usual case. A job **detached** from the
harness (`setsid`, `nohup`) is invisible to it, so it isn't holding the agent awake — hibernation is
what will eventually end it. For those, either:

- **Keep a terminal or SSH session open** for the duration — an open connection pins the agent
  awake, and closing the tab or editor is what lets it idle down again.
- **Raise the hibernation timeout**, or set it to `0` to never hibernate, before starting the job.
  Raising it only defers hibernation rather than preventing it.

Closing a chat tab doesn't stop a turn either — a running turn keeps the agent awake until it
finishes.

Crossing the `0` boundary asks you to confirm in both directions, since it's the difference between
an always-on agent and one that may be interrupted.

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

Your Sizes draw against a per-user **compute budget** — a ceiling on how much CPU and memory your *running* agents can reserve at once. An agent reserves its Size against the budget only while it's up; pausing, stopping, or hibernating it frees that room immediately. If the Size you pick is larger than the room currently free, the wizard warns you and names the free CPU and memory — the agent still saves, it just waits parked until room frees.

- If a new or resized agent would exceed your budget, it can't start and shows as **Over budget** — it waits until other agents free up room, and **Start** is its retry.
- To make room, **pause** or **stop** an agent you aren't using, then start the waiting one. DAM will also free room for you where it can: starting an agent that doesn't fit hibernates your own idle, unattended agents — longest-idle first, and only enough to fit — once they've been idle a few minutes, without waiting for their own timeout. Agents you have open, and any set to never hibernate, are never touched.
- Changing the Size of a running agent restarts it on save; if it's down, the new Size applies the next time it starts. Growing a **running** agent past your budget is refused when you save — free room first.

**Home** shows where you stand, under **Compute allocated** — what your running agents currently reserve
against the ceiling, not what they're actually consuming, so it drops when you stop or pause one rather
than when an agent goes quiet. **Request more** beside it goes wherever your operator points it.

## Available Coding Harnesses

Each agent runs one **coding harness** — the AI software inside the pod:

| Harness | Best For |
|---------|----------|
| Claude Code | General-purpose coding, debugging, refactoring (Anthropic) |
| Codex | OpenAI Codex coding agent |
| PI Agent | Pi coding agent with multi-LLM support |
| IBM Bob | Bob shell agent (Anthropic, Mistral, Granite) |

Want to bring your own? See the [Building a Custom Image](../guides/custom-image.md) guide.

## Experiments, knowledge bases, and frameworks

Experiments and knowledge bases aren't separate kinds of agent — they're a base agent (today, Claude Code) with a capability layered on:

- An **experiment** is a Claude Code agent with the experiment-authoring skill installed. Describe the design → build → test → learn loop you want and it writes it as ordinary Python, while the platform charts every stage, score, and candidate live. See [Experiments](experiments.md) and the [Run experiments](../guides/run-experiments.md) walkthrough.
- A **knowledge base** is a Claude Code agent set up to build and maintain a body of knowledge you chat with. Feed it sources and it answers with citations back to where the answer came from. See [Knowledge bases](knowledge-bases.md) and the [Capture team knowledge](../guides/building-knowledge-bases.md) guide.

Rather than write an optimization loop from scratch, an experiment can run an established **framework**. Each isn't an agent in its own right — it's run by a Claude Code subagent the experiment spawns, pairing the base agent with that framework. Point it at a target and a measurable objective in chat — "evolve this function to run faster on this input set" — and it authors the run, drives the framework, and reports the winner. Just tell it which one to use.

<div class="rows" markdown>

- **NOUS** <small>Alpha</small> <span>Hypothesis-driven experimentation framework for long-running research.</span>
- **OpenEvolve** <small>Alpha</small> <span>Open-source AlphaEvolve — evolves code through scored variations.</span>
- **ShinkaEvolve** <small>Alpha</small> <span>Sample-efficient evolutionary program optimization from Sakana AI.</span>
- **GEPA** <small>Alpha</small> <span>Reflective text and prompt optimization (Genetic-Pareto).</span>
- **k-search** <small>Alpha</small> <span>LLM-driven search that optimizes GPU kernels.</span>
- **SkyDiscover** <small>Alpha</small> <span>Berkeley Sky lab's code and algorithm optimizer, with AdaEvolve and EvoX search strategies.</span>

</div>

## Managing agents

### Create

Each kind of agent has its own creation flow, reached from its own place in the app. Each one is a
**single page** — no steps to click through:

- **Coding agent** — from **Coding agents**, click **Create coding agent**. Name it, pick an
  **image** (the AI software inside the pod — Claude Code, Codex, Pi Agent, or Bob), and choose a
  model **provider**. To bring your own image instead of a listed one, see
  [Building a Custom Image](../guides/custom-image.md).
- **Experiment agent** — from **Experiments**, click **Create experiment**. Name it, choose a
  provider, and add connections. It's a Claude Code agent with the experiment authoring skill
  installed. See [Experiments](experiments.md).
- **Knowledge base agent** — from **Knowledge bases**, click **Create knowledge base**. Name it,
  pick a **template** (Plain Wiki or LLM Wiki), choose a provider, and add connections. See
  [Knowledge Bases](knowledge-bases.md).

New agents start with **Trusted defaults** for network access; you tune network permissions and
**Size** (CPU/memory, within your budget) afterward from the agent's page — see [Configure](#configure).

### Configure

From the agent's page, the left nav has these sections:

- **Agent Setup** — name, provider, **Model settings** (see
  [Sessions](sessions.md#model-settings)), **Network access** (which external services it may reach),
  **Lifecycle** (hibernation timeout — in minutes; `0` = never), **Environment** variables, and
  **Size** (CPU/memory while running, within your [compute budget](#compute-budget-and-size);
  changing it restarts a running agent on save). The image is fixed at creation — the only change
  to it is the [update](#updating-the-image) below.
- **Connections** — link GitHub, API keys, and other credentials
- **Channels** — connect the agent to Slack channels or Telegram chats (see
  [Slack](../integrations/slack.md) and [Telegram](../integrations/telegram.md))
- **Skills** — install or remove capabilities
- **Schedules** — run recurring tasks
- **Artifacts** — files the agent has published
- **Usage** — this agent's model spend for a calendar month, broken down by model and by day. It includes work the agent delegated to other agents.

### Ways in

**Chat is the default way in, and it's one click.** Clicking an agent in the list opens its chat — and
starts it first if it's asleep, so there's no config page to pass through on the way. From chat, the
**⋮** menu carries **Configure agent** to reach the page described above; the agents list has the same
item in each row's **⋯** menu, and the agent's page has an **Open chat** button back the other way.

The other ways in sit **beside the message box on a new session** — where a session has no messages
yet, under *Start a new session*:

| Option | What it does |
|--------|--------------|
| **Terminal (browser)** | Opens the harness terminal (PTY) in the browser |
| **Terminal (local)** | Shows a copyable `dam chat` command to attach from your own terminal |
| **VS Code / Zed (local)** | Shows copyable `dam ssh connect` commands for either editor — see the [VS Code/Zed (SSH) guide](../integrations/vs-code-zed-ssh.md) |

Both dialogs link the CLI quickstart, so a first-time setup is one hop away. **New** in the sessions
sidebar opens a chat session directly rather than asking which kind you want — start a terminal from
the launch options above instead.

### Pause, Stop & Start

From the agent's actions menu you can take it down without deleting it:

- **Pause — wakes on next use** — puts it to sleep now and frees its budget.
- **Stop — until started again** — a hard stop; it stays down until you bring it back up (or a scheduled run fires). Use this to reliably free budget for other work.
- **Wake** / **Start** — brings an agent back up; the label depends on where you are. Opening an agent that's down offers **Start agent**. In the agents list a parked agent offers *Start* (which retries the budget gate) and any other agent that's down offers *Wake*; on the agent's own actions menu an agent that's down reads *Wake* whatever the reason it went down.
- **Restart** — offered for a running or errored agent, and the only power action for those states. It recreates the agent on its current settings; the workspace survives. Reach for it when one is wedged.

All of these preserve the workspace and configuration exactly like hibernation — see [What Survives Hibernation](#what-survives-hibernation).

### Updating the image

When the template an agent was created from moves on, an **Update** button appears — in the agent's
page header, and on its row in the agents list. Hover it to see which image version the template now
ships, and a **See what changed** link where the harness publishes release notes; click it to confirm
and apply.

Applying is always explicit and changes only the image. An agent that's **hibernating**, or parked
over budget, applies it the next time it starts; anything else restarts to apply it, so in-flight work
is interrupted. The confirmation tells you which case you're in.

Where **more than one** agent is behind, the list carries a banner counting them, with **Update all** to
catch up in one action rather than a visit per agent. The experiment and knowledge base lists carry the
same banner over their own agents.

### Delete

Deleting an agent permanently removes its workspace, sessions, and configuration. This cannot be undone.

!!! warning "Deleting an agent destroys all workspace files. If you have important work, download it or push it to a remote repository first."
