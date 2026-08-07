# Knowledge Bases

A **knowledge base** is a sandbox whose job is to build and maintain a body of knowledge you work
with by chatting. You feed it sources; it organizes what it learns and keeps it.

The platform deliberately owns very little here — there's no ingestion pipeline, no query language,
no fixed schema. The knowledge lives as files in the sandbox's workspace, and the agent does the
organizing. That means you shape it by talking to it, and you can always open the files yourself.

## Templates

Pick a template at create time. It decides what tooling gets installed in the workspace:

| Template | What it does |
|----------|-------------|
| **LLM Wiki** (default) | Installs the llm-wiki toolkit and builds a structured wiki with a knowledge graph as you feed it sources. |
| **Plain Wiki** | No toolkit — the agent reads your files as knowledge and keeps what it learns as plain markdown notes. |

Choose **Plain Wiki** if you want something durable and dependency-free that you could read straight
out of the workspace — once it's set up, it works offline. Choose **LLM Wiki** for a structured wiki
with a knowledge graph over what it has learned. Both templates fetch their setup from GitHub when the
knowledge base is created, so neither avoids the network requirement below; the difference is whether
you still need the network afterwards.

The template is recorded on the knowledge base and shown in the list, so you can see which one it
came from.

!!! warning "The template can't be changed later"
    Template and harness are both create-time choices. Switching from Plain Wiki to LLM Wiki means
    creating a new knowledge base — there's no in-place conversion, and deleting the old one destroys
    its workspace.

!!! info "Needs an image your operator enables"
    The knowledge-base starting point needs the `claude-code` agent image. If it isn't installed, the
    wizard hides it and tells you to ask your operator to enable it.

## Creating one

From **Knowledge bases**, click **Create knowledge base** — that's the sandbox wizard with the
**Knowledge base sandbox** starting point already selected.

You'll pick the template, then name it, choose a **Size** and network access preset, a provider, and
any connections to grant. Two things are decided for you: the harness is pinned to Claude Code (no
image to choose), and the provider list is limited to the two that reach Claude — **IBM LiteLLM ETE
Proxy** (recommended) or **Anthropic**.

After it's created, the platform runs the template's setup command **once** in the workspace and
wakes the sandbox, so the tooling is in place before you say anything. No conversation happens during
setup — it's a workspace change, not a turn.

Opening a brand-new knowledge base **greets you**: the agent introduces itself and explains how to
feed it, rather than dropping you into an empty chat.

!!! tip "No greeting? Setup was still landing"
    The greeting can fire before setup has finished, in which case it silently does nothing and you
    get an empty chat. It won't retry on its own — just send a message, or run `/wiki-onboard`
    yourself. Setup itself keeps retrying on each wake until it succeeds.

!!! warning "Setup needs network access"
    The template's setup step downloads its toolkit from GitHub, through the sandbox's egress
    gateway. If you switch to the **Strict default-deny** network preset, that download can be blocked
    and the knowledge base comes up without its tooling. **Trusted defaults** — the default — is the
    safer choice here; if setup seems not to have run, check
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
any other sandbox content.

## Managing one

Wake, restart, pause, stop, and delete behave exactly as they do for [any
sandbox](agents.md#lifecycle). Opening a knowledge base from the list keeps you in that context, so
closing it returns you to the list rather than to Sandboxes.

Its settings page covers the harness (read-only), network access, and connections. **Size isn't
editable there** — a knowledge base runs on its defaults to keep the surface conversational.

There's no CLI command for creating a knowledge base, but `dam agent list` / `get` / `delete` all
work on one.

**Deleting a knowledge base deletes the sandbox and its workspace** — the knowledge with it. Export
anything you want to keep first.

Knowledge bases are owner-scoped: you see and manage your own.
