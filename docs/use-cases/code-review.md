# Code Review

Get automated PR reviews on every pull request — posted to GitHub and summarized in Slack.

This Claude Code configuration runs on a schedule, continuously watching for new or updated PRs and reviewing them for correctness, security, performance, architecture issues, and documentation drift. No one needs to trigger it — DAM's scheduled execution keeps the reviewer running around the clock with no human in the loop.

## What It Does

Once configured, the agent runs on a schedule and for each open PR:

1. **Fetches the diff** — reads the PR's changes, body, comments, and prior reviews
2. **Clones the branch** — pulls the code locally for deeper analysis
3. **Runs skill-based checks** — documentation drift, TypeScript engineering, and React UI engineering
4. **Posts a structured review** — findings with severity levels, inline comments on specific lines
5. **Sends a Slack summary** — the full review lands in your team's channel
6. **Tracks state** — knows which PRs it already reviewed at which commit, only re-reviews when new commits are pushed
7. **Learns your preferences** — remembers what to ignore, what to emphasize, and adapts over time

Reviews are posted as proper GitHub PR reviews (with inline comments), not just top-level comments.

## How It Works

```
Scheduled Heartbeat (configurable interval)
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
         ├── Send full review to Slack
         └── Update local tracking (REVIEWS.md)
```

Deduplication is built in — the agent embeds a SHA marker in every review and checks both local state and GitHub before posting. No duplicate reviews, even across restarts.

## Requirements

### Provider

- **IBM LiteLLM ETE Proxy** — the agent needs model access to function

### Connections

- **GitHub** (Enterprise or Public) — the agent reads PRs, posts reviews, and clones branches
- **Slack** (optional but recommended) — sends review summaries to your team channel

### Network Access

**Trusted defaults** (recommended) — covers GitHub and common package registries. Slack API access will surface in your inbox for approval on first use.

## Setup

1. **Create a Claude Code agent** — use the standard Claude Code template from the Web UI or CLI
2. **Add your provider** — configure IBM LiteLLM with your API key
3. **Add a GitHub connection** — provide a PAT with repo access to the repository you want reviewed
4. **Clone the Code Guardian repo** — in your agent session, clone [github.com/dam-agents/code-guardian](https://github.com/dam-agents/code-guardian) into the workspace
5. **Set the target repo** — set the `GITHUB_REPO` environment variable to the repo you want reviewed (e.g., `my-org/my-project`)
6. **Create a schedule** — set a heartbeat interval (e.g., every 5 minutes) so the agent checks for new PRs automatically
7. **Connect Slack** (optional) — link a Slack channel to receive review notifications


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
| `work/MEMORY.md` | Review preferences and learned feedback |
| `work/REVIEWS.md` | Tracking index — one row per PR with latest review state |
| `work/reviews/pr-<n>.md` | Per-PR review history (append-only log) |

## Reference Implementation

[github.com/dam-agents/code-guardian](https://github.com/dam-agents/code-guardian)
