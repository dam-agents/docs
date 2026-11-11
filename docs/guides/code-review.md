# Automate PR review

PR review is the kind of steady, unglamorous work a long-running agent is perfect for. This guide walks
through **Code Guardian** — the team's PR-review agent, built on the [code-guardian](https://github.com/dam-agents/code-guardian)
definition — end to end, so you can set it up on your own repo.

On a schedule, it watches for new and updated PRs and reviews each one for correctness, security,
performance, architecture, and documentation drift — posting straight to GitHub, with optional Slack
nudges when a PR waits too long for a human. No one triggers it, and it only ever reports and proposes:
a human still owns the merge.

## What It Does

Once configured, the agent runs on a schedule and for each open PR:

1. **Fetches the diff** — reads the PR's changes, body, comments, and prior reviews
2. **Clones the branch** — pulls the code locally for deeper analysis
3. **Runs configurable skill-based checks** — the skill set lives in `work/CONFIG.md` (each skill with its own source repo, trigger, and output section); the default set covers issue fit, documentation drift, TypeScript engineering, and React UI engineering
4. **Posts a structured review** — findings with severity levels, inline comments on specific lines
5. **Shepherds PRs toward human review** (optional, Slack) — watches how long each PR waits for an independent review and nudges the right reviewers (or the author, when changes are requested) in your team channel, escalating over time
6. **Generates visual PR artifacts on demand** — assign the bot account to a PR and the next heartbeat publishes a self-contained HTML review artifact and links it in a comment
7. **Tracks state** — knows which PRs it already reviewed at which commit. The first review of every open non-draft PR is automatic; after new commits a re-review runs **only** when it's asked for — add the re-review label (default `code-guardian-review`, chosen at onboarding; the agent removes it once posted), re-request the bot's review on GitHub, or ask in chat or a PR comment that @-mentions it
8. **Learns your preferences** — remembers what to ignore, what to emphasize, and adapts over time

Reviews are posted as proper GitHub PR reviews (with inline comments), not just top-level comments.

## How It Works

```
Scheduled run (every 10 min by default; configurable)
         │
         ├── Load work/CONFIG.md + install/refresh configured skills
         ├── Fetch open PRs
         ├── Skip PRs with no review trigger (same HEAD, or no re-review request)
         │
         For each new/updated PR:
         │
         ├── Fetch diff + PR context (body, comments, prior reviews)
         ├── Clone branch locally
         ├── Run every configured review skill
         │   (default: issue-fit, doc-drift, typescript-engineering, react-ui-engineering)
         ├── Generate structured review with findings
         ├── Post GitHub PR review (summary + inline comments)
         └── Update local tracking (REVIEWS.md)
         │
         ├── Artifact sweep — publish visual HTML artifact
         │   for every PR the bot account is assigned to
         └── PR Shepherd sweep (only when Slack is enabled) —
             nudge reviewers/authors on PRs waiting for review
```

Deduplication is built in — the agent embeds a SHA marker in every review and checks both local state and GitHub before posting. No duplicate reviews, even across restarts.

## Requirements

### Provider

- **Any provider that serves Claude**, such as **Anthropic** — the agent runs on Claude Code. On the IBM hosted deployment, the IBM ETE LiteLLM Proxy works too

### Connections

- **GitHub** or **GitHub Enterprise** — the agent reads PRs, posts reviews, and clones branches
- **A Slack channel binding** (optional, **opt-in at onboarding**) — powers the PR Shepherd: escalating reminders in your team channel for PRs that sit unreviewed. The agent asks during onboarding whether to enable it and, on yes, walks you through importing your team (GitHub org team or top contributors) plus their Slack member IDs into a reviewer roster — the only people it may ever @-mention. With Slack disabled the agent never touches it; reviews still land in the chat UI and on GitHub.

    This is a **channel binding**, not a Slack connection — nudges go out through the platform to the conversation bound to this agent. Bind it with `/dam bind` from inside Slack.

!!! tip "You can add Slack later"
    Skip Slack at onboarding and bind a channel whenever you're ready — the agent picks it up from
    then on.

!!! warning "Use a dedicated bot account for the GitHub connection"
    Reviews are posted as whatever identity the GitHub connection resolves to. Connect a **dedicated machine/bot account** — **not your personal account** — that is a collaborator on the target repo with permission to review PRs.

    This matters for two reasons:

    - **GitHub ignores self-reviews.** The reviewing account must not be the one that opens the PRs it reviews, or the reviews are dropped.
    - **Reviews are signed as that account.** If you connect a human developer's account, every automated review will appear to come from that person — on their colleagues' PRs and their own. This is the most common setup mistake.

    A dedicated account can also be shared across a team's agents. If you don't have one, create a new GitHub account for the bot and add it as a collaborator on the target repo before onboarding.

    If your organization would rather not manage a second human account, connecting **your own GitHub
    App** gives you a bot identity the org itself owns — see
    [Connecting your own GitHub App](../core-concepts/connections.md#connecting-your-own-github-app).

### Network Access

**Trusted defaults** (recommended) — covers GitHub and common package registries. Slack nudges need no egress rule: they leave through the platform itself, not the open internet, so no approval appears in Activity for them.

## Setup

1. **Create a Claude Code agent** — on **Home**, **Create agent** (on a brand-new account, choose **Start from scratch instead** in the starter kits it opens) and pick the **Claude** card in the Web UI, or `dam agent create <name> --provider <your-provider>` from the CLI
2. **Add your provider** — configure a provider that serves Claude, such as Anthropic, with your API key
3. **Add the GitHub connection** — connect the **dedicated bot account** described under [Connections](#connections) above, with review access to the target repo
4. **Set the target repo** — set the `GITHUB_REPO` environment variable to the repo you want reviewed (e.g., `my-org/my-project`), under **Environment** in the agent's **Agent Setup** section or with `--env GITHUB_REPO=my-org/my-project` at create time; if you skip it, the agent asks for the slug at the start of onboarding and stores it in its config. Optionally set `GITHUB_REPO_WORK` to a separate repo that backs the agent's persistent `work/` state.
5. **Onboard the agent** — send it the onboarding runbook as your **first message** and let it set itself up (see below)
6. **Connect Slack** (optional) — only needed if you want the PR Shepherd nudging; you opt in during onboarding

### Onboarding the agent

You don't clone the repo or run any setup by hand — **the agent onboards itself.** Send it the link to `ONBOARDING.md` as your first message:

```
Here is a file — read it and set yourself up according to it:
https://github.com/dam-agents/code-guardian/blob/main/ONBOARDING.md
```

Onboarding is a short **interactive dialog** — the agent checks out its own definition (derived from the URL you send, so forks stay pinned to the fork), wires up the `work/` directory, and asks a handful of configuration questions: the display name it signs reviews with, the dedup-marker prefix, which review skills to run (defaults offered, each skill with its own source repo), whether to enable Slack notifications (and if so, it imports your team into the reviewer roster), and the review cadence (default **every 10 minutes**). Everything lands in `work/CONFIG.md`; it then reconstructs review state, creates its recurring schedules (the review heartbeat, plus a Slack nudge sweep and a weekly audit where those apply), writes the `$HOME/.code-guardian-onboarded` sentinel, and finishes with a summary of what it configured and how to use it day-to-day.

!!! note "Its schedules run in UTC, and you change them by asking"
    The agent creates its schedules as cron expressions, and cron schedules run in **UTC** — so a
    weekly audit or a workday-windowed sweep won't follow your local clock. To get local wall-clock
    times (and quiet hours), you want a schedule with a frequency and timezone instead.

    A schedule the agent created isn't editable in the UI, though: because the agent owns it, its
    **Schedules** section says so and points you back to chat. So **ask the agent** to change when it
    runs — that's the intended route, and it can delete and recreate the schedule itself. You can still
    pause or delete it from the panel yourself, and **Run now** in its menu fires it once without moving
    its next run. See [Schedules](../core-concepts/schedules.md).

!!! tip "Files live in `work/`, not a subfolder"
    The agent's state files (`CONFIG.md`, `MEMORY.md`, `REVIEWS.md`, `reviews/`) live **directly in `work/`** — not in a `code-guardian/` subfolder. If you're upgrading from an older setup that nested the checkout under `work/code-guardian/`, re-onboard from a fresh agent rather than moving files by hand.

!!! tip "Customizing the agent"
    To change the agent's behavior, **fork** [dam-agents/code-guardian](https://github.com/dam-agents/code-guardian) and onboard from your fork's `ONBOARDING.md` URL instead.

## Review Output

Every review follows a structured format:

```
## PR #42: Add user authentication
Author: @dev | Branch: feature/auth → main | Changes: +230 −45 (8 files)

### Summary
Adds JWT-based authentication middleware with refresh token rotation.

### Findings
- 🔴 Critical: Token secret hardcoded in config (`src/auth.ts:12`)
- 🟡 Warning: Missing rate limiting on login endpoint (`src/routes/login.ts:28`)
- 🟢 Suggestion: Consider extracting token validation to a shared utility

### Documentation Check (doc-drift)
✅ No documentation drift detected.

### TypeScript Engineering Review
- 🟡 Warning: Unused import (`src/middleware.ts:3`)

### Verdict
REQUEST_CHANGES — hardcoded secret must be moved to environment variable
```

### Severity Levels

| Level | Meaning |
|-------|---------|
| 🔴 Critical | Logic errors, security issues, data loss risks |
| 🟡 Warning | Performance problems, missing error handling, maintainability |
| 🟢 Suggestion | Style improvements, minor refactors, nice-to-haves |
| ✅ Looks good | Positive callouts for well-written code |

## Preference Learning

The agent remembers your feedback and adapts:

- **"Don't flag missing comments"** — added to ignore list, never raised again
- **"Be stricter about error handling"** — adjusts review focus
- **"The null check on line 42 is intentional"** — suppressed for that specific PR only

Preferences are stored in `MEMORY.md` (global) and per-PR files (PR-specific overrides). Global preferences apply to all future reviews; PR-specific dismissals only affect that one PR.

## Key Workspace Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Agent operating instructions — the full review playbook |
| `work/CONFIG.md` | Instance configuration filled in at onboarding — bot identity, dedup marker, the review-skills table (each skill with its own source repo and trigger), artifact skill, Slack opt-in. Change any value later just by asking the agent in chat — except the dedup marker, which is fixed once the first review is posted (changing it would hide every past review from deduplication). |
| `work/MEMORY.md` | Review preferences and learned feedback |
| `work/REVIEWS.md` | Tracking index — one row per PR with latest review state |
| `work/reviews/pr-<n>.md` | Per-PR review history (append-only log) |
| `work/DEVELOPERS.md` | Reviewer roster — the only people the agent may @-mention; only present when Slack is enabled |
| `work/SHEPHERD.md` | PR Shepherd nudge ledger — only present when Slack is enabled |

## Reference Implementation

[github.com/dam-agents/code-guardian](https://github.com/dam-agents/code-guardian)
