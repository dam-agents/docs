# Welcome to DAM

Deploy Agents Massively. Run agent harnesses like Claude Code headless in the cloud, on a schedule, connected to your tools — without exposing your tokens.

[Launch DAM →](https://dam-dev.apps.dam-cl.fmaas.res.ibm.com/){:target="_blank" .md-button .md-button--primary}

Don't have access yet? [Join the waitlist](https://ibm.biz/dam-waitlist){:target="_blank"}.

---

## Why DAM?

Running AI agents on your laptop has limitations:

- **Sessions die** when you close the terminal or your laptop sleeps
- **Credentials are exposed** — your API keys sit in plain text where the agent can read (and leak) them
- **No team collaboration** — colleagues can't interact with your agent
- **No scheduling** — you can't tell an agent "review all new PRs every morning at 9am"

DAM solves all of these. Your agents run in isolated cloud environments with persistent workspaces, credential-safe networking, Slack integration, and scheduled execution.

---

## What You Get

| Capability | What it means for you |
|------------|----------------------|
| **Persistent workspaces** | Your agent's files, repos, and memory survive indefinitely |
| **Secure credentials** | Tokens are injected on the wire by a separate gateway — the agent never sees them |
| **Always-on** | Agents hibernate when idle and wake automatically (~20s) when you message them |
| **Scheduled execution** | Set up recurring tasks that run without you |
| **Slack-native** | Chat with your agent from Slack. Colleagues participate with their own credentials. |
| **Extensible** | Install and publish skills from any git repo |

---

## Modes of Interaction

| Mode | Description |
|------|-------------|
| **Web Chat** | Browser-based chat UI — quick tasks, file browsing, session history |
| **Terminal** | Full PTY shell that preserves native harness TUIs (e.g., Claude Code's interactive CLI with slash commands and tool approvals) |
| **Slack** | Message your agent from Slack threads — teammates can join with their own credentials |
| **Schedules** | Agents run autonomously on a timer — no human in the loop |

---

## Available Agent Harnesses

| Harness | Best For |
|---------|----------|
| **Claude Code** | General-purpose coding — write, debug, refactor, review |
| **Pi Agent** | Multi-model coding (GPT-4, Mistral, Gemini, etc.) |
| **Code Guardian** | Automated PR code review with Slack notifications |
| **Google Workspace** | Gmail triage, Calendar, Drive, Sheets automation |

---

## Get Started

| | |
|---|---|
| **[Quickstart →](getting-started/quickstart.md)** | Create a Claude Code agent and start chatting in 5 minutes. |
| **[Set up Code Guardian →](agents/code-guardian.md)** | Automated PR reviews posted to your Slack channel. |
| **[Chat from Slack →](integrations/slack.md)** | Message your agent directly from Slack threads. |
