# Code Guardian

Code Guardian is an automated PR code review agent that reviews pull requests and posts findings to chat or Slack.

## When to Use Code Guardian

- Automated code review on new pull requests
- Scheduled daily/weekly review sweeps across a repo
- Catching security issues, style violations, and logic errors
- Learning and enforcing your team's review preferences

## Required Connections

| Connection | Required? | Purpose |
|------------|-----------|---------|
| Anthropic API key | **Yes** | Powers the AI reviewer |
| GitHub | **Yes** | Access to read PRs and post comments |

## Key Features

- **Learns your preferences** — maintains a `MEMORY.md` with review guidelines you teach it over time
- **Review history** — keeps a `REVIEWS.md` log of all past reviews
- **Slack integration** — posts review findings to your connected Slack channel
- **Non-destructive** — never modifies code or merges PRs, only reviews

## Configuration

| Environment Variable | Purpose |
|---------------------|---------|
| `GITHUB_REPO` | Target repository (e.g., `my-org/my-project`) |

## Setting Up Scheduled Reviews

Code Guardian works best when scheduled to run automatically:

1. Create a Code Guardian agent
2. Set the `GITHUB_REPO` environment variable
3. Connect GitHub credentials
4. Create a [Schedule](../features/schedules.md) (e.g., "Weekdays at 9am")
5. Optionally connect a Slack channel for notifications

The agent will wake on schedule, check for new PRs, review them, and post findings.
