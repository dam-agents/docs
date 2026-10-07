# Knowledge Bases

A **knowledge base** is an agent whose job is to build and maintain a body of knowledge you work
with by chatting. You feed it sources; it organizes what it learns and keeps it.

The platform deliberately owns very little here — there's no ingestion pipeline, no query language,
no fixed schema. The knowledge lives as files in the agent's workspace, and the agent does the
organizing. That means you shape it by talking to it, and you can always open the files yourself.

## Choosing a kit

A knowledge base is created from a [starter kit](agents.md#starter-kits) on the **Knowledge** shelf. The
kit decides what tooling gets installed in the workspace:

| Kit | What it does |
|----------|-------------|
| **LLM Wiki** | Installs the llm-wiki toolkit and builds a structured wiki with a knowledge graph as you feed it sources. |
| **Plain Wiki** | No toolkit — the agent reads your files as knowledge and keeps what it learns as plain markdown notes. |

Choose **Plain Wiki** if you want something durable and dependency-free that you could read straight
out of the workspace — once it's set up, it works offline. Choose **LLM Wiki** for a structured wiki
with a knowledge graph over what it has learned. Both kits fetch their setup from GitHub when the
knowledge base is created, so neither avoids the network requirement below; the difference is whether
you still need the network afterwards.

The kit is recorded on the agent and shown as a badge on its row, so you can see which one it came from.

!!! warning "The kit can't be changed later"
    Kit and harness are both create-time choices. Switching from Plain Wiki to LLM Wiki means
    creating a new knowledge base — there's no in-place conversion, and deleting the old one destroys
    its workspace.

## Creating one

Open **Starter Kits**, pick **Knowledge**, and open **LLM Wiki** or **Plain Wiki**. The kit's page shows
what it brings; **Use this Starter Kit** takes you to **Create an agent** with most of it already filled
in.

Name it, pick a **Harness**, and choose a **Provider**. The harness list is the same one any agent
chooses from and Claude Code is preselected, but a **custom image** isn't offered for a kit — so a
knowledge base runs one of the listed harnesses. **Repository** shows what the kit seeds your workspace
from, and **Connections** lists anything it needs. You don't choose network access here: the read-only
**Network access** box shows the preset the agent starts on — the trusted defaults unless the kit sets
its own — and you change it afterwards from its **Agent Setup** page.

After it's created, the platform runs the kit's install command **once** in the workspace, so the tooling
is in place before you say anything. No conversation happens during that step — it's a workspace change,
not a turn.

Then the kit opens a **first session** itself. Rather than dropping you into an empty chat, the agent
introduces itself, asks you for the few things only you can tell it, and explains how to feed it. An
**Onboarding** badge on the agent, and a bar above the message box, track what's still outstanding; any
schedules the kit registered are held until the agent marks setup complete. Open it the moment it's
created and you'll wait through **Preparing workspace** first, so nothing arrives half-configured.

!!! warning "Setup needs network access"
    The kit's install step downloads its toolkit from GitHub, through the agent's egress
    gateway. If you switch to the **Strict default-deny** network preset, that download can be blocked
    and the knowledge base comes up without its tooling. **Trusted defaults** — the preset a new agent
    starts on unless its kit sets another — is the safer choice here; if setup seems not to have run, check
    [network access](network-access.md) first.

## Adding sources and building knowledge

It's a conversation. Point the agent at what you want it to learn from:

- **Paste or describe** material directly in chat.
- **Upload files** into the workspace with the [File Manager](file-manager.md) or `dam import`, then
  ask the agent to read them.
- **Grant a connection** and have it pull from a source it can reach — a GitHub repo, for example.
  Reaching anything on the network needs the host allowed under
  [network access](network-access.md).

Ask it to reorganize, summarize, cross-link, or correct what it holds. Because the knowledge is just
files in the workspace, you can browse or download it at any time, and it survives hibernation like
any other agent content.

## Sharing one with your team

A knowledge base is normally yours alone. **Sharing** publishes a read-only copy of it that other
people's agents can read — so a base one person feeds can answer questions for the whole team without
anyone else being able to change it, or reach the agent behind it.

Open the base's **Agent Setup** → **Knowledge**, then **Share this knowledge base**. Pick which of its
workspace folders to publish — you're choosing folders, not the whole workspace — and DAM mints a
**share link** starting `kbshare_`. Hand that to a teammate and they paste it under the same
**Knowledge** section on *their* agent, under **Connected knowledge bases**, and that agent can list,
search, and read what you published.

What a consumer gets is deliberately narrow: **raw files only**. Skills, slash commands, and the way
your base behaves aren't shared and won't work for them — they're reading your notes, not borrowing
your agent.

Once shared, the published copy **updates itself** as the knowledge changes; **Refresh now** republishes
immediately if you don't want to wait. The section shows how many documents are published, when the copy
was last updated, and how many queries it has served.

Because consumers read the published copy rather than the agent, **a shared base answers while it's
hibernated** — nobody wakes your agent, or spends its compute, by reading it.

!!! warning "The link is the access control"
    Anyone holding a share link can read everything you published, with no DAM account needed for the
    agent they paste it into. **Rotate link…** issues a new one and everyone on the old link loses
    access until you hand out the replacement; **Stop sharing…** deletes the published copy and kills
    the link at once. Neither touches the knowledge base itself.

**Reveal share link** shows it again later — it's hidden until you ask, and it's never given to the
agent, so you can safely tell a base to share itself and still be the only one who can hand the link
out. **Public name** is what consumers see the base called.

!!! note "Sharing needs an object store"
    Publishing writes the read-only copy to object storage, which your operator configures. Where none
    is, sharing isn't available and says so — ask them rather than retrying.

## Managing one

Wake, restart, pause, stop, and delete behave exactly as they do for [any
agent](agents.md#lifecycle). A knowledge base lists on **Home** with your other agents.

It gets the same **Agent Setup** page as any other agent too: the name, compute resources, its harness
(read-only), the provider and model settings, **Knowledge**
([sharing](#sharing-one-with-your-team)), network access, **Lifecycle**, and environment
variables — alongside the usual Connections, Channels, Skills,
Schedules, Artifacts, and Usage sections.

When its kit has a newer version, the base is offered the update like any agent from a kit — a
**Kit update** tag in the list and a bar above the message box; see
[Starter kits](agents.md#starter-kits). That updates the same kit; it doesn't switch you to the other one.

There's no CLI command for creating a knowledge base, but `dam agent list` / `get` / `delete` all
work on one.

**Deleting a knowledge base deletes the agent and its workspace** — the knowledge with it. Export
anything you want to keep first.

Knowledge bases are owner-scoped: you see and manage your own.
