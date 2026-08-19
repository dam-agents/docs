---
hide:
  - toc
---

<div class="hero" markdown>

# IBM Research DAM

Shared infrastructure for AI-native engineering and research — run agents continuously, connect them to your tools, and scale securely without exposing credentials.

[Launch DAM →](https://ibm.biz/dam-agents){:target="_blank" .md-button .md-button--primary}
[Join the waitlist](https://ibm.biz/dam-waitlist){:target="_blank" .md-button}

<small class="vpn-note">If you're outside an IBM office, VPN is required</small>

</div>

## Why DAM?

<div class="grid cards" markdown>

-   :material-cloud-outline:{ .lg .middle } **Runs in the cloud**

    ---

    Sandboxes live in the cloud, not on your laptop — schedules and channel messages keep them working while you're away.

-   :material-shield-lock:{ .lg .middle } **Isolated execution**

    ---

    Each sandbox runs in an isolated container, with all access routed through a policy-enforced gateway.

-   :material-key-variant:{ .lg .middle } **Zero trust credentials**

    ---

    Connect agents to your tools through a gateway that adds your credentials without ever revealing them.

-   :material-account-group:{ .lg .middle } **Built for teams**

    ---

    Run sandboxes on schedules for recurring work, and share what they produce in a Slack channel.

</div>

---

## Ways to Use DAM

<div class="grid cards" markdown>

-   :material-chat:{ .lg .middle } **Web UI**

    ---

    Chat with your agent, stream its terminal, and manage files — all from the browser.

-   :material-console-line:{ .lg .middle } **CLI**

    ---

    Create sandboxes, attach to live sessions, and manage them from your local terminal.

-   :material-slack:{ .lg .middle } **Slack**

    ---

    Invite the DAM bot, run `/dam bind`, and the whole channel can work with your agent.

-   :material-microsoft-visual-studio-code:{ .lg .middle } **VS Code/Zed**

    ---

    Open your sandbox's workspace in VS Code or Zed via the CLI's `dam ssh` editor modes.

-   :material-calendar-clock:{ .lg .middle } **Schedules**

    ---

    Run sandboxes on a recurring timer — daily code reviews, nightly audits, continuous monitoring.

</div>

---

## Available Harness Images

<div class="grid cards" markdown>

-   :material-code-braces:{ .lg .middle } **Claude Code**

    ---

    Reasoning-first assistant for complex coding tasks.

-   :material-robot:{ .lg .middle } **PI Agent**

    ---

    Multi-provider coding harness across leading LLMs.

-   :material-console:{ .lg .middle } **IBM Bob**

    ---

    Bob shell agent — Anthropic, Mistral, or Granite models.

-   :material-code-tags:{ .lg .middle } **Codex**

    ---

    Execution-first system for end-to-end coding tasks.

</div>

## Available Specialized Images

Ready-to-use agents built on top of the coding harnesses. Pick one under **Specialized sandbox** in the create wizard, connect a model provider, and launch. A couple need a little more — GEPA's optimization loop requires an API-key-style provider connection (an Anthropic OAuth token drives only its chat), and k-search evaluates on Modal by default.

<div class="grid cards" markdown>

-   :material-flask-outline:{ .lg .middle } **NOUS** <small>Alpha</small>

    ---

    Hypothesis-driven agent for long-running research.

-   :material-dna:{ .lg .middle } **OpenEvolve** <small>Alpha</small>

    ---

    Evolutionary agent that iteratively optimizes your code.

-   :material-chip:{ .lg .middle } **k-search** <small>Alpha</small>

    ---

    LLM-driven agent that optimizes GPU kernels.

-   :material-dna:{ .lg .middle } **ShinkaEvolve** <small>Alpha</small>

    ---

    Sample-efficient evolutionary program-optimization agent.

-   :material-text-box-search-outline:{ .lg .middle } **GEPA** <small>Alpha</small>

    ---

    Reflective prompt and text-optimization agent.

-   :material-dna:{ .lg .middle } **AdaEvolve** <small>Alpha</small>

    ---

    Multi-island adaptive code and algorithm optimizer.

-   :material-dna:{ .lg .middle } **EvoX** <small>Alpha</small>

    ---

    Self-evolving code and algorithm optimizer.

-   :material-creation:{ .lg .middle } **More soon…**

    ---

    More ready-to-use agents are on the way.

</div>

---

## Beyond Chat

<div class="grid cards" markdown>

-   :material-test-tube:{ .lg .middle } **[Experiments](core-concepts/experiments.md)**

    ---

    Give an agent something to optimize and watch it design and run the loop — every stage, score, and candidate live as it runs.

-   :material-book-open-variant:{ .lg .middle } **[Knowledge Bases](core-concepts/knowledge-bases.md)**

    ---

    Build an organized body of knowledge by conversation. Feed it sources; it keeps and organizes what it learns.

-   :material-package-variant-closed:{ .lg .middle } **[Artifacts](core-concepts/artifacts.md)**

    ---

    Durable, versioned outputs — reports, dashboards, generated pages — that outlive the sandbox and can be shared by link.

</div>

---

## Get Started

<div class="grid cards" markdown>

-   **[Quickstart →](getting-started/quickstart.md)**

    ---

    Create a Claude Code sandbox and start chatting in five minutes.

-   **[Use Cases →](use-cases/code-review.md)**

    ---

    See what teams are building — code review, app development, and more.

</div>
