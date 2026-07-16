# CLI

The `dam` CLI lets you manage agents, attach to terminal sessions, transfer files, wire up connections, and more — all from your local machine.

## Installation

```bash
npm install -g @dam-agents/cli
```

Requires Node 20 or later.

## Configuration

```bash
dam config set server https://dam-prod.apps.dam-cl.fmaas.res.ibm.com/
```

Config is stored at `~/.config/dam/config.toml`. Credentials are stored separately at `~/.local/state/dam/auth.toml` (mode 0600).

**Environment variables:**

| Variable | Purpose |
|----------|---------|
| `DAM_SERVER` | Override the configured server URL |
| `DAM_TOKEN` | Use a bearer token directly (bypasses login, useful for CI) |

Precedence: CLI flag > environment variable > config file.

**Global flags:**

- `--server <url>` — override the configured server URL for a single call. Available on essentially every command that talks to the platform.
- `--json` — emit machine-readable JSON instead of the default table/summary. Available on most read and mutate commands; handy for scripting.

## Authentication

```bash
dam auth login       # Authenticate via OAuth 2.0 Device Authorization Grant
dam auth logout      # Clear local credentials and best-effort revoke the refresh token
dam auth status      # List configured hosts, their credential source, and the active server
```

Login uses the OAuth 2.0 Device Authorization Grant — you'll see a URL and code to confirm in your browser. Tokens are stored locally and refreshed automatically.

```bash
dam auth login --server https://host   # set the active host while logging in
dam auth login --no-browser            # print the verification URL instead of opening a browser
dam auth login --force                 # skip the re-login confirm when already logged in
```

For CI/headless use, set `DAM_TOKEN=<bearer>` instead of logging in. There is no `--token` flag, to avoid leaking tokens into shell history.

### API keys (`auth token`)

For headless/CI use you can mint long-lived API keys:

```bash
dam auth token create --name ci-key                 # mint a key; plaintext printed ONCE on stderr
dam auth token create --name ci --scope agents:operate --agent agent-123 --expires 2026-12-31T00:00:00Z
dam auth token list                                 # list your keys (plaintext never shown again)
dam auth token revoke key-abcdef12                  # revoke a key; rejected on the next request
```

- `--name <name>` is required; `--scope <scope...>` and `--agent <agent-id...>` are repeatable.
- `--expires <iso>` takes an ISO 8601 timestamp.
- The plaintext token is shown only at creation — copy it immediately, it cannot be recovered.

## Commands

### Agent Management

```bash
dam agent list                                   # List your agents (default command)
dam agent get <name-or-id>                        # Show one agent's details
dam agent create <name> --template <id>           # Create an agent (--template required)
dam agent create <name> --template <id> --wait    # Block until the agent reaches running
dam agent create-interactive                     # Interactive setup: name + template + credentials (TTY-only)
dam agent delete <name-or-id>                     # Delete an agent and all its persistent data (confirms)
dam agent delete <name-or-id> --yes               # Delete without confirmation
dam agent restart <name-or-id>                    # Restart the pod (persistent volumes survive)
```

- `create` flags: `--template <id>` (required), `--description <text>`, `--env KEY=VAL` (repeatable), `--wait`, `--timeout <seconds>` (default 120), `--json`.
- `--wait` is available on `create` and `restart`; pair with `--timeout` to change the 120s default.
- `--json` on `list`/`get` for machine-readable output.

### Templates

```bash
dam template list    # List agent templates available on the active host
```

Templates are read-only from the CLI — operators manage them via Helm.

### Connections

Connections are reusable OAuth/credential bindings (GitHub, MCP servers, custom-header API keys) that you create once — they're yours alone — then grant to any of your agents.

```bash
dam connection templates                          # List provider templates you can create from
dam connection list                               # List team connections
dam connection list <agent>                       # List the connections granted to one agent
dam connection connect <provider-or-url>          # Create a connection from a template or MCP server URL
dam connection disconnect <id-or-name>            # Remove a team connection (deletes the credential)
dam connection grant <agent> --connection <id>    # Grant one or more connections to an agent (repeatable)
dam connection revoke <agent> --connection <id>   # Revoke connections from an agent (repeatable; idempotent)
```

`connection connect` takes either a provider template id (see `dam connection templates`) or an MCP server URL (`https://…`). Useful flags:

- `--name <name>` — connection name (defaults to a slug of the template name or the MCP host)
- `--auth oauth|none` — MCP auth mode (auto-detected by default)
- `--no-browser` — for MCP OAuth, print the consent URL instead of opening a browser
- `--timeout <seconds>` — MCP OAuth timeout (default 300)
- `-c, --config <key=value>` — set an optional template config input (repeatable), e.g. `--config model=premium-shell`; values are validated against the template's declared inputs.

### Network access

Per-agent rules that pre-approve the external hosts an agent may reach. (The command is `network`; the underlying module is the egress controller.)

```bash
dam network list <agent>                               # List network access rules
dam network create <agent> --host api.example.com      # Add a rule (default method/path '*', verdict allow)
dam network update <rule-id>                           # Update an existing rule
dam network revoke <rule-id>                           # Remove a rule
dam network preset <agent>                             # Show the agent's effective preset
dam network apply-preset <agent> --preset <none|trusted|all>   # Apply a network preset
dam network trusted-hosts                              # Platform-wide hosts seeded by the 'trusted' preset
```

`network create` flags: `--host <h>` (required), `--method <m>` (default `*`), `--path <p>` (default `*`), `--verdict allow|deny` (default `allow`), `-y, --yes` (skip the path-level restart confirmation), `--json`.

### Skills

Browse skill sources and manage the skills installed on an agent.

```bash
dam skill source list                                  # List connected skill sources
dam skill source add <git-url>                         # Register a skill source from a git URL
dam skill source remove <id>                           # Remove a source (installed skills remain)
dam skill source refresh <id>                          # Refresh a source
dam skill catalog <source>                             # Scan a source's available skills
dam skill catalog <source> --agent <ref>               # Annotate install status against an agent
dam skill list <agent>                                 # List the skills on an agent
dam skill install <agent> --source <id-or-url> --name <skill>   # Install (or update) a skill
dam skill uninstall <agent> --source <id-or-url> --name <skill>   # Remove a skill (both flags required)
dam skill publish <agent> --name <skill> --source <id-or-url> [--title <t>] [--body <b>]   # Publish a skill as a PR (--name/--source required; --title/--body optional)
```

### Channels

Bind an agent to a messenger channel (Slack, Telegram) and manage who is allowed to talk to it.

```bash
dam channel available                                       # List enabled messenger providers
dam channel list <agent>                                    # Channels bound to an agent (+ Slack allow-list)
dam channel slack connect <agent> --channel-id <C0…>        # Bind a Slack channel to the agent
dam channel slack disconnect <agent>                        # Unbind the Slack channel
dam channel telegram connect <agent> [--bot-token <token>]  # Bind a Telegram chat/bot
dam channel telegram disconnect <agent>                     # Unbind Telegram
dam channel allow <agent> --user <email>                    # Add allowed Slack user(s) (repeatable)
dam channel disallow <agent> --user <email>                 # Remove allowed Slack user(s) (repeatable)
```

- A Slack channel binds to **exactly one** agent. After connecting, allowed users run `/platform login` in Slack to link their own credentials.
- `--user <email>` is repeatable; `allow`/`disallow` rewrite the agent's allowed-user list.

### Approvals

The human-in-the-loop inbox — requests your agents are waiting on (held tool calls, network prompts).

```bash
dam approval list                                      # Pending requests across all your agents
dam approval list <agent> --status all                 # Include resolved and expired, for one agent
dam approval approve <id>                               # Approve a pending request
dam approval approve <id> --entire-host                 # Approve the whole host (network requests only)
dam approval deny <id>                                  # Deny a pending request
```

- `list` flags: `--status pending|resolved|expired|all` (default `pending`), `--limit <n>` (1–500).
- `approve`/`deny` take `--once` to allow/deny only this specific held call (the same request shape re-prompts next time). `approve --entire-host` allows any method and path on the host (network requests only).

### Schedules

Manage time-triggered tasks (schedules) attached to an agent — the CLI equivalent of the agent's **Schedules** panel.

```bash
dam schedule list <agent>              # List the schedules attached to an agent
dam schedule get <schedule-id>         # Show one schedule, expanded
dam schedule create <agent> --name <name> --task <task> --daily 22:00   # Create an RRULE schedule
dam schedule update <schedule-id> --task "New prompt"   # Update name/task/recurrence/timezone/etc.
dam schedule enable <schedule-id>      # Enable a schedule (idempotent)
dam schedule disable <schedule-id>     # Disable a schedule (idempotent)
dam schedule delete <schedule-id>      # Delete a schedule (idempotent — unknown ids exit 0)
dam schedule reset-session <schedule-id>   # Reset the session a continuous schedule reuses
```

**`create`** requires `--name <name>` and `--task <task>` (the prompt run each tick), plus one recurrence. Choose the recurrence with exactly one of:

- `--daily <HH:MM>` — run daily at a 24-hour time
- `--every <interval>` — run every N minutes (`Nm`) or hours (`Nh`)
- `--rrule <body>` — a raw RFC 5545 RRULE body (e.g. `FREQ=WEEKLY;BYDAY=MO;BYHOUR=7;BYMINUTE=0`)

Additional flags (on both `create` and `update`):

- `--weekdays <days>` — limit `--daily`/`--every` to specific days, e.g. `MO,WE,FR`
- `--timezone <tz>` — IANA timezone (defaults to the host's zone)
- `--quiet-window <HH:MM-HH:MM>` — suppress fires inside a window (repeatable)
- `--session-mode <fresh|continuous>` — new session per fire, or one shared session
- `--json` — emit the schedule as JSON (available on every subcommand)

`delete` also takes `-y, --yes` to skip the confirmation prompt. Schedule ids come from `dam schedule list`.

### Terminal Sessions

```bash
dam chat <agent>                       # New terminal session
dam chat <agent> --continue            # Reattach to the most recent terminal session
dam chat <agent> --resume <session-id> # Resume a specific session
dam chat <agent> --reset               # Kill the existing PTY and start fresh
dam session list <agent>               # List all sessions for an agent
```

`dam chat` connects your terminal directly to the agent's interactive TUI over WebSocket. Keystrokes (including Ctrl+C) pass through to the remote harness. On disconnect, the CLI prints a `dam chat --resume` command for easy reattach. Short flags: `-c` (`--continue`), `-r` (`--resume`).

### SSH access

```bash
dam ssh connect <agent>                # Open an SSH session to an agent
dam ssh connect <agent> -x code        # Launch VS Code Remote-SSH against the agent
dam ssh connect <agent> -x zed         # Launch Zed against the agent
dam ssh configure <agent>              # Write SSH host config for an agent (no launch)
dam ssh configure --all                # Reconcile config for every agent (prunes deleted ones)
dam ssh configure --clear              # Remove all dam-managed SSH host config
```

`-x, --exec <bin[:mode]>` selects the client to launch: an executable name or path, optionally suffixed with `:ssh`, `:code`, or `:zed` to force how it's invoked. `dam ssh configure` writes the same host config without launching a client.

### File Operations

```bash
dam import <agent> <path...>           # Upload files/folders to the agent workspace
dam file list <agent> [<path>]         # List files in the agent workspace
dam file get <agent> <remote-path>     # Download a single file from the agent
dam file put <agent> <local> <remote>  # Upload a single file to the agent
```

**Import** bundles local paths into a gzipped tar and uploads them. Each path lands at `work/<basename>` on the agent, replacing any existing entry at that path. Excluded: `node_modules`, `.venv`, `__pycache__`, `.DS_Store`. Use `-y, --yes` to skip the confirm (required on non-TTY).

**File get/put** operate on single files. `file get` writes to `basename(remote-path)` in the current directory by default; use `-o, --output <path>` to choose the local path, `--stdout` to stream to stdout instead of writing a file, and `--overwrite` to clobber an existing local file. `file put` takes `--overwrite` to clobber an existing remote file.

### Utility

```bash
dam ping             # Reach the configured server and check compatibility
dam version          # Print the local CLI version and (best-effort) the server's
dam --version        # Show CLI version
dam --help           # Show help
```

## Agent Addressing

Agents can be referenced by **name** (case-sensitive, exact match) or by **ID** (starts with `agent-`). If a name matches multiple agents, the CLI exits with an error.

## Compatibility

The CLI checks server compatibility before each command. If your CLI version is below the server's minimum, commands will fail with an upgrade prompt. Run `npm install -g @dam-agents/cli@latest` to update.
