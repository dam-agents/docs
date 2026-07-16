# Web App Development

Turn an idea into a deployed application — without writing a single line of code yourself.

This guide walks through setting up the [software-factory](https://github.com/dam-agents/software-factory) agent on DAM to do it yourself.

Once running, the agent interviews you about what to build, then runs autonomously on a schedule to write a PRD, break it into GitHub issues, implement each one, and set up CI/CD so the result ships on every merge. You describe the idea once, then walk away — DAM's scheduled execution keeps the agent working with no human in the loop.

## What It Does

Once configured, this agent runs an end-to-end product loop:

1. **Interviews you** — asks relentless questions until it fully understands the problem
2. **Writes a PRD** — synthesizes the conversation into a Product Requirements Document on GitHub
3. **Decomposes into issues** — splits the PRD into vertical-slice tickets with dependency ordering
4. **Implements autonomously** — picks up unblocked tickets, writes code, opens PRs, reviews, and merges
5. **Deploys continuously** — sets up CI/CD (tracked under the `area:ci` label) so every merge ships
6. **Stops when done** — detects when all tickets are closed, then disables itself
7. **Pauses when stuck** — if something fails repeatedly, it stops and asks for help

All of this runs on a 1-minute heartbeat schedule — no human in the loop after the initial interview.

## How It Works

```
You ←→ Agent (interactive session)
         │
         ├── /grill-me   → interviews you about the problem
         ├── /to-prd     → creates PRD issue on GitHub
         └── /to-issues  → decomposes into tickets
                              │
                              ▼
                    Scheduled Heartbeat (every 1 min)
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
               Pick next   Implement   Review & merge
              unblocked     ticket      PR, close ticket
               ticket                        │
                                             ▼
                                      All done? → disable schedule
```

State lives entirely in GitHub labels — no external database. A heartbeat lock ensures only one execution runs at a time.

## Requirements

### Provider

- **IBM LiteLLM ETE Proxy** — the agent needs model access to function

### Connections

- **GitHub** (Enterprise or Public) — the agent creates issues, opens PRs, and merges code. Choose based on where your target repo lives.

### Network Access

**Trusted defaults** (recommended) — covers GitHub, npm, PyPI, and package mirrors out of the box. Any additional hosts the agent needs will surface in your inbox for approval.

## Setup

1. **Create a Claude Code sandbox** — choose Claude Code as the coding agent in the Web UI, or `dam agent create --template claude-code` from the CLI
2. **Add your provider** — configure IBM LiteLLM with your API key
3. **Add a GitHub connection** — provide a PAT with repo access to your target repository, so `gh` can authenticate
4. **Start a chat session** — tell the agent what you want to build. On first run it reads its `work/config.json`; if that's missing it asks you for the configuration (the target `owner/repo`), then verifies the repo's default branch and required labels before starting.

!!! note "Deployed as a Custom Image"
    Software Factory isn't one of the ready-made images in the agent picker. Deploy it from the image published in [dam-agents/software-factory](https://github.com/dam-agents/software-factory) (see the repo's README) via the **Custom Image** option. Once running, the agent operates from the `work/CLAUDE.md` playbook and its bundled skills — you don't clone anything by hand.


## Using the Agent

### The Interview Phase

Start a chat and tell the agent what you want to build. It will invoke `/grill-me` — an exhaustive interview that walks down every branch of your idea's decision tree, one question at a time.

The agent provides its own recommended answer for each question. You can accept, override, or redirect.

### PRD Creation

Once the interview reaches shared understanding, the agent runs `/to-prd` — synthesizing everything into a structured PRD with:

- Problem statement
- Proposed solution
- Extensive user stories
- Implementation decisions
- Testing strategy
- Out of scope

The PRD is published as a GitHub issue with the `PRD` label.

### Issue Decomposition

Next, `/to-issues` breaks the PRD into independently-implementable tickets, each labeled `prd:<n>` to tie it back to its PRD. Each ticket is a **vertical slice** — a thin cut through all layers (UI, API, data, tests) that's demoable on its own. Tickets are ordered by dependency so the agent works on them in the right sequence.

### Autonomous Execution

After decomposition, the agent creates a heartbeat schedule. Every minute, it wakes up and:

- Checks if there's a ticket labeled `working` → continues implementing
- Checks for `needs review` → reviews and merges the PR
- Finds the next unblocked ticket → starts working on it
- No open work left → checks if the PRD is complete

You don't need to do anything — just watch PRs appear on GitHub.

### Stuck Detection

If the agent encounters repeated failures (CI red, permission errors, ambiguous requirements), it runs `/pause-and-notify`, which:

1. Labels the PRD with `paused`
2. Notifies via Slack (if connected)
3. Stops working until you intervene

To resume: tell the agent "resume" in a chat session (it clears the `paused` label via `/pause-and-notify` in resume mode).

## Built-in Skills

| Skill | What it does |
|-------|-------------|
| `/grill-me` | Interviews you relentlessly about your plan until shared understanding |
| `/to-prd` | Turns conversation context into a PRD published to GitHub |
| `/to-issues` | Decomposes a PRD into vertical-slice tickets with dependencies |
| `/pause-and-notify` | Pauses the loop (and clears the pause on resume) when the agent is stuck |
| `frontend-design` | Provides frontend/React engineering guidance for UI-heavy tickets |

## Key Files in the Workspace

| File | Purpose |
|------|---------|
| `work/CLAUDE.md` | The agent's operating manual — defines behavior for interactive sessions and heartbeat routing |
| `work/HEARTBEAT.md` | The execution plan for each scheduled wake — orchestration, implementation, review, done detection |
| `work/config.json` | Project state — target repo (`github.repo`), PRD number, stuck counters, thresholds |
| `work/DEVELOPMENT_GUIDELINES.md` | Code quality standards the agent follows during implementation |

## Reference Implementation

Full source: [github.com/dam-agents/software-factory](https://github.com/dam-agents/software-factory)
