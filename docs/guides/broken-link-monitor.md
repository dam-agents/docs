# Broken Link Monitor

Keep your documentation links healthy automatically.

This guide walks through setting up the [link-guardian](https://github.com/dam-agents/link-guardian) agent on DAM to do it yourself.

Once running, the agent works on a daily schedule, scanning markdown files across your repositories for broken links and maintaining a single tracking issue per repo. No one needs to remember to check — DAM's scheduled execution keeps the monitor running every day with no human in the loop.

## What It Does

Once configured, the agent runs daily and for each target repository:

1. **Scans all markdown** — READMEs, docs, any `.md` file in the repo
2. **Checks every link** — HTTP requests for external URLs, path resolution for relative links
3. **Classifies results** — 404s, dead domains, missing files, redirects
4. **Maintains one tracking issue** — opens a GitHub issue listing all broken links, updates it on subsequent runs, closes it when everything is fixed
5. **Learns ignore rules** — comment on the tracking issue to teach it which links to skip (e.g., flaky domains, intentionally dead links)

No PRs, no code changes — the agent only reads your code and writes issues.

## How It Works

```
Scheduled Run (daily)
     │
     ├── Read config (which repos to watch)
     ├── For each target repo:
     │     │
     │     ├── Clone or pull latest
     │     ├── Scan markdown for links
     │     ├── Check each link (HTTP + relative path resolution)
     │     ├── Reconcile with previous state
     │     │
     │     ├── No broken links? → Close tracking issue (if open)
     │     ├── New broken links? → Open or update tracking issue
     │     └── Same as before? → No action
     │
     └── Done
```

State is persisted on a volume between runs — the agent remembers what it found last time and only updates the issue when something changes.

## Requirements

### Provider

- **IBM LiteLLM ETE Proxy** — the agent needs model access to function

### Connections

- **GitHub** or **GitHub Enterprise** — the agent clones repos (read) and manages issues (write). The default GitHub connection uses OAuth (scopes `repo`, `read:user`, `user:email`). If you'd rather use a token, click **New** on the **GitHub** card and pick **Personal access token** instead of **Sign in with Github**.

### Network Access

**Allow everything** — this agent checks external URLs across the internet to verify links are alive, so it needs broad outbound access. Alternatively, use **Trusted defaults** and approve hosts as they surface in your inbox.

## Setup

1. **Create a Claude Code agent** — click **Create agent** and pick the **Claude Code** harness in the Web UI, or `dam agent create <name> --template claude-code` from the CLI
2. **Add your provider** — configure IBM LiteLLM with your API key
3. **Add a GitHub connection** — connect GitHub via OAuth (default), or pick **Personal access token** on the **GitHub** card if you prefer a token
4. **Clone the Link Guardian repo** — in your agent session, clone [github.com/dam-agents/link-guardian](https://github.com/dam-agents/link-guardian) into `work/`
5. **Run the first session** — the agent will ask you which org/repos to watch and save the configuration
6. **Create a schedule** — set it to run daily so the agent checks links automatically


## First Run

On the first run, the agent notices it has no configuration and asks you interactively:

- Which GitHub org to watch
- Which repos within that org (you can say "all" or pick specific ones)

Once you answer, it saves the target list to `state/MEMORY.md` and starts scanning immediately. Future scheduled runs use this saved config without asking again.

## Tracking Issues

The agent maintains exactly **one issue per repo** titled `[Bug]: broken links`. The issue body lists every broken link with its location and status code.

- **New broken links found** — issue is opened (or updated if already open)
- **Some links fixed** — issue body is updated to reflect current state
- **All links fixed** — issue is closed with a comment

### Teaching Ignore Rules

Comment on the tracking issue to teach the agent what to skip:

> "ignore flaky.example.com, it rotates IPs and always times out"

The agent picks up these comments on the next run and adds them to its ignore list. Useful for known-flaky domains or intentionally dead links.

## Key Workspace Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Agent operating instructions |
| `state/MEMORY.md` | Target repo list, configuration, and learned ignore rules |
| `state/repos/<owner>-<repo>.json` | Per-repo state (broken links, issue numbers) |
| `.agents/skills/check-broken-links/` | The scanning skill and TypeScript helpers (symlinked as `.claude/skills/…`) |
| `state/skip-patterns.txt` | Learned ignore rules, appended from tracking-issue comments |

## Security

- **Reads code, writes only issues** — it clones markdown and opens, updates, or closes tracking
  issues; it never pushes branches or PRs. Note this is the agent's own discipline rather than a
  platform-enforced scope: a GitHub connection carries the standard `repo` scope, so a modified
  agent definition *could* write code.
- **No token visibility** — credentials are injected at the network layer, never exposed to the agent

## Reference Implementation

[github.com/dam-agents/link-guardian](https://github.com/dam-agents/link-guardian)
