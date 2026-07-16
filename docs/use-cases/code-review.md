# Code Review

Get automated PR reviews on every pull request — posted to GitHub, and optionally summarized in Slack.

This guide walks through setting up the [code-guardian](https://github.com/dam-agents/code-guardian) agent on DAM to do it yourself.

Once running, the agent works on a schedule, continuously watching for new or updated PRs and reviewing them for correctness, security, performance, architecture issues, and documentation drift. No one needs to trigger it — DAM's scheduled execution keeps the reviewer running around the clock with no human in the loop.

## What It Does

Once configured, the agent runs on a schedule and for each open PR:

1. **Fetches the diff** — reads the PR's changes, body, comments, and prior reviews
2. **Clones the branch** — pulls the code locally for deeper analysis
3. **Runs skill-based checks** — documentation drift, TypeScript engineering, and React UI engineering
4. **Posts a structured review** — findings with severity levels, inline comments on specific lines
5. **Sends a Slack summary** (if Slack is connected) — the full review lands in your team's channel
6. **Tracks state** — knows which PRs it already reviewed at which commit, only re-reviews when new commits are pushed
7. **Learns your preferences** — remembers what to ignore, what to emphasize, and adapts over time

Reviews are posted as proper GitHub PR reviews (with inline comments), not just top-level comments.

## How It Works

```
Scheduled Heartbeat (every 10 min by default; configurable)
         │
         ├── Fetch open PRs
         ├── Skip already-reviewed (same HEAD commit)
         │
         For each new/updated PR:
         │
         ├── Fetch diff + PR context (body, comments, prior reviews)
         ├── Clone branch locally
         ├── Run doc-drift skill
         ├── Run typescript-engineering skill
         ├── Run react-ui-engineering skill
         ├── Generate structured review with findings
         ├── Post GitHub PR review (summary + inline comments)
         ├── Send full review to Slack (if connected)
         └── Update local tracking (REVIEWS.md)
```

Deduplication is built in — the agent embeds a SHA marker in every review and checks both local state and GitHub before posting. No duplicate reviews, even across restarts.

## Requirements

### Provider

- **IBM LiteLLM ETE Proxy** — the agent needs model access to function

### Connections

- **GitHub** (Enterprise or Public) — the agent reads PRs, posts reviews, and clones branches
- **Slack** (optional but recommended) — sends review summaries to your team channel

!!! warning "Use a dedicated bot account for the GitHub connection"
    Reviews are posted as whatever identity the GitHub connection resolves to. Connect a **dedicated machine/bot account** — **not your personal account** — that is a collaborator on the target repo with permission to review PRs.

    This matters for two reasons:

    - **GitHub ignores self-reviews.** The reviewing account must not be the one that opens the PRs it reviews, or the reviews are dropped.
    - **Reviews are signed as that account.** If you connect a human developer's account, every automated review will appear to come from that person — on their colleagues' PRs and their own. This is the most common setup mistake.

    A dedicated account can also be shared across a team's agents. If you don't have one, create a new GitHub account for the bot and add it as a collaborator on the target repo before onboarding.

### Network Access

**Trusted defaults** (recommended) — covers GitHub and common package registries. Slack API access will surface in your inbox for approval on first use.

## Setup

1. **Create a Claude Code sandbox** — choose Claude Code as the coding agent in the Web UI, or `dam agent create --template claude-code` from the CLI
2. **Add your provider** — configure IBM LiteLLM with your API key
3. **Add the GitHub connection** — connect the **dedicated bot account** described under [Connections](#connections) above, with review access to the target repo
4. **Set the target repo** — set the `GITHUB_REPO` environment variable to the repo you want reviewed (e.g., `my-org/my-project`). Optionally set `GITHUB_REPO_WORK` to a separate repo that backs the agent's persistent `work/` state.
5. **Onboard the agent** — send it the onboarding runbook as your **first message** and let it set itself up (see below)
6. **Connect Slack** (optional) — link a Slack channel to receive review notifications

### Onboarding the agent

You don't clone the repo or run any setup by hand — **the agent onboards itself.** Send it the link to `ONBOARDING.md` as your first message:

```
Here is a file — read it and set yourself up according to it:
https://github.com/dam-agents/code-guardian/blob/main/ONBOARDING.md
```

The agent then checks out its own definition, wires up the `work/` directory, reconstructs review state, creates the recurring review schedule (it asks for the cadence, defaulting to **every 10 minutes**), and writes the `$HOME/.code-guardian-onboarded` sentinel so it won't repeat the process.

!!! info "\"This pod is not onboarded…\" is expected before you send that message"
    On a fresh sandbox the agent reports that it hasn't been onboarded yet (no `work/` runtime state, no scheduled heartbeat). That's normal — it's waiting for you to point it at `ONBOARDING.md`. **You** kick off onboarding by sending the message above; the agent does the actual work autonomously from there. You do not run the runbook yourself.

!!! tip "Files live in `work/`, not a subfolder"
    The agent's state files (`CONFIG.md`, `MEMORY.md`, `REVIEWS.md`, `reviews/`) live **directly in `work/`** — not in a `code-guardian/` subfolder. If you're upgrading from an older setup that nested the checkout under `work/code-guardian/`, re-onboard from a fresh sandbox rather than moving files by hand.

!!! tip "Customizing the agent"
    To change the agent's behavior, **fork** [dam-agents/code-guardian](https://github.com/dam-agents/code-guardian) and onboard from your fork's `ONBOARDING.md` URL instead.

!!! warning "IBM Enterprise GitHub (`github.ibm.com`)"
    Reachability of `github.ibm.com` from the DAM cluster isn't guaranteed for every install. If your target repo is on Enterprise GitHub, confirm the agent can reach it (and that the bot account is a collaborator there) before relying on the schedule.


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
| `work/CONFIG.md` | Instance configuration filled in at onboarding (target repo, options) |
| `work/MEMORY.md` | Review preferences and learned feedback |
| `work/REVIEWS.md` | Tracking index — one row per PR with latest review state |
| `work/reviews/pr-<n>.md` | Per-PR review history (append-only log) |
| `work/DEVELOPERS.md` | Reviewer roster — only present when Slack is enabled |

## Reference Implementation

[github.com/dam-agents/code-guardian](https://github.com/dam-agents/code-guardian)
