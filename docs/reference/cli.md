# CLI

The `dam` CLI lets you manage agents, attach to terminal sessions, and transfer files — all from your local machine.

## Installation

```bash
npm install -g @dam-agents/cli
```

Requires Node 20 or later.

## Configuration

```bash
dam config set server https://your-platform-url.example.com
```

Config is stored at `~/.config/dam/config.toml`. Credentials are stored separately at `~/.local/state/dam/auth.toml` (mode 0600).

**Environment variables:**

| Variable | Purpose |
|----------|---------|
| `DAM_SERVER` | Override the configured server URL |
| `DAM_TOKEN` | Use a bearer token directly (bypasses login, useful for CI) |

Precedence: CLI flag > environment variable > config file.

## Authentication

```bash
dam auth login       # Opens browser for OAuth device flow
dam auth logout      # Revokes tokens and clears local credentials
dam auth status      # Shows auth state per configured host
```

Login uses the OAuth 2.0 Device Authorization Grant — you'll see a URL and code to confirm in your browser. Tokens are stored locally and refreshed automatically.

For CI/headless use, set `DAM_TOKEN=<bearer>` instead of logging in. There is no `--token` flag to avoid leaking tokens into shell history.

## Commands

### Agent Management

```bash
dam agent list                          # List all your agents (default command)
dam agent get <name-or-id>              # Get agent details
dam agent create <name> --template <id> # Create an agent (scripted)
dam agent create-interactive            # Interactive agent setup (TTY only)
dam agent delete <name-or-id>           # Delete an agent (confirms by default)
dam agent delete <name-or-id> --yes     # Delete without confirmation
dam agent restart <name-or-id>          # Restart agent pod (PVC survives)
```

Use `--wait` with `create` or `restart` to block until the agent reaches `running` state (120s timeout).

Use `--json` on `list` or `get` for machine-readable output.

### Templates

```bash
dam template list    # List available agent templates
```

Templates are read-only from the CLI — operators manage them via Helm.

### Terminal Sessions

```bash
dam chat <agent>                       # New terminal session
dam chat <agent> --continue            # Reattach to most recent terminal session
dam chat <agent> --resume <session-id> # Resume a specific session
dam chat <agent> --reset               # Kill PTY and start fresh
dam session list <agent>               # List all sessions for an agent
```

`dam chat` connects your terminal directly to the agent's interactive TUI over WebSocket. Keystrokes (including Ctrl+C) pass through to the remote harness. On disconnect, the CLI prints a `dam chat --resume` command for easy reattach.

### File Operations

```bash
dam import <agent> <path...>           # Upload files/folders to agent workspace
dam file list <agent> [<path>]         # List files in agent workspace
dam file get <agent> <remote-path>     # Download a file from agent
dam file put <agent> <local> <remote>  # Upload a single file to agent
```

**Import** bundles local paths into a gzipped tar and uploads them. Each path lands at `work/<basename>` on the agent, replacing any existing entry at that path. Excluded: `node_modules`, `.venv`, `__pycache__`, `.DS_Store`.

**File get/put** operate on single files. Use `--overwrite` to clobber existing files. Use `--stdout` with `get` to stream to stdout instead of writing a local file.

### Utility

```bash
dam ping             # Connectivity and compatibility check
dam version          # Show server version and CLI compatibility
dam --version        # Show CLI version
dam --help           # Show help
```

## Agent Addressing

Agents can be referenced by **name** (case-sensitive, exact match) or by **ID** (starts with `agent-`). If a name matches multiple agents, the CLI exits with an error.

## Compatibility

The CLI checks server compatibility before each command. If your CLI version is below the server's minimum, commands will fail with an upgrade prompt. Run `npm install -g @dam-agents/cli@latest` to update.
