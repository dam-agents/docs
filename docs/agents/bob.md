# Bob Agent

Bob is IBM's general-purpose AI shell. The DAM platform runs Bob Shell as a fully managed agent with ACP integration, auto-approval of tool requests, and persistent workspace.

## Use Cases

- General-purpose coding and file manipulation
- IBM-internal workflows and tooling
- Tasks requiring higher resource allocation (up to 2 CPU, 4 GB RAM)

## Required Connections

| Type | Purpose |
|------|---------|
| Bob API Key | Authenticates with the Bob Shell service (`BOBSHELL_API_KEY`) |

The API key is injected by the platform's credential gateway — it never appears in the pod environment.

## Configuration

Bob supports several environment variables for customization:

| Variable | Purpose |
|----------|---------|
| `BOB_SHELL_MODEL` | Default model for Bob to use |
| `BOB_INSTANCE_ID` | Instance identifier for tenant scoping |
| `BOB_TEAM_ID` | Team identifier for tenant scoping |
| `BOB_MAX_COINS` | Budget cap for the session |
| `BOB_CHAT_MODE` | Default persona/mode |

## Autonomy Model

Bob operates differently depending on session mode:

- **Chat mode (ACP)** — runs with `--yolo` flag, which auto-approves all tool permission requests. There is no per-tool human-in-the-loop in the platform UI for chat sessions.
- **Terminal mode (TUI)** — runs in `auto_edit` mode, which auto-approves file edits but prompts for other tool calls within the TUI.

## Session Behavior

Each terminal session starts a fresh Bob TUI session. Bob uses a numeric session index internally (`~/.bob/`), so sessions are not resumed by UUID — opening a new terminal always starts fresh.

The persistent volume at `/home/agent/` preserves Bob's session index and workspace files across hibernation.

## Resource Requirements

Bob requires more resources than other agent templates:

| Resource | Request | Limit |
|----------|---------|-------|
| CPU | 500m | 2 |
| Memory | 1 Gi | 4 Gi |
| Storage | 2 Gi | — |
