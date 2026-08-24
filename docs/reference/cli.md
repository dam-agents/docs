# CLI

The `dam` CLI lets you manage sandboxes, attach to terminal sessions, transfer files, wire up connections, and more — all from your local machine.

## Installation

```bash
npm install -g @dam-agents/cli
```

Requires Node 20 or later.

## Configuration

```bash
dam config set server https://dam-prod.apps.dam-cl.fmaas.res.ibm.com/
```

`server` is the only key `dam config set` accepts. Config is stored at `~/.config/dam/config.toml` (or `$XDG_CONFIG_HOME/dam/`); credentials are stored separately at `~/.local/state/dam/auth.toml` (mode 0600).

**Environment variables:**

| Variable | Purpose |
|----------|---------|
| `DAM_SERVER` | Override the configured server URL |
| `DAM_TOKEN` | Use a bearer token or `pk_…` API key directly (bypasses login, useful for CI). Used **verbatim and never refreshed** |

Precedence: CLI flag > environment variable > config file.

**Global flags:**

- `--server <url>` — override the configured server URL for a single call. Available on every networked command **except `dam auth status` and the `dam auth token` subcommands**, which use the configured active host.
- `--json` — emit machine-readable JSON instead of the default table/summary. Available on most read and mutate commands; handy for scripting. Not available on `auth login`, `auth logout`, `auth status`, `auth token revoke`, `ping`, `version`, `dam ssh`, `chat`, `file get`, `file put`, `config set`, or `agent create-interactive`. (`file list` does have it.)

Precedence for the server URL is flag > `DAM_SERVER` > config file > **error** — there is no default server.

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
dam auth logout --server https://host  # log out of a specific host (defaults to the active one)
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

- `--name <name>` is required. `--scope <scope...>` and `--agent <agent-id...>` each take several values after one flag (`--scope agents:read agents:operate`), and repeating the flag works too — both forms add up. Omit `--scope` and you get `agents:operate` alone; a key always carries at least one scope. The valid values are `agents:read`, `agents:operate`, `agents:manage`, `credentials:read`, and `credentials:manage`. `--agent` defaults to every sandbox you own.
- `--expires <iso>` takes an ISO 8601 timestamp.
- The plaintext token is shown only at creation — copy it immediately, it cannot be recovered.
- Minted keys carry a **`pk_`** prefix and can be used directly as `DAM_TOKEN`. API-key principals cannot mint further keys.

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
- `--json` on `list`, `get`, `delete`, and `restart` for machine-readable output.

### Templates

```bash
dam template list    # List agent templates available on the active host
```

Templates are read-only from the CLI — your operator decides which ones the host offers.

### Connections

Connections are reusable OAuth/credential bindings (GitHub, MCP servers, custom-header API keys) that you create once — they're yours alone — then grant to any of your agents.

```bash
dam connection templates                          # List provider templates you can create from
dam connection list                               # List your connections
dam connection list <agent>                       # List the connections granted to one agent
dam connection connect <provider-or-url>          # Create a connection from a template or MCP server URL
dam connection update <id-or-name>                # Rotate a header connection's stored secret (prompts securely)
dam connection reauth <id-or-name>                # Re-run login and consent for an OAuth connection
dam connection disconnect <id-or-name>            # Remove a connection (deletes the stored credential)
dam connection grant <agent> --connection <id>    # Grant one or more connections to an agent (repeatable; at least one)
dam connection revoke <agent> --connection <id>   # Revoke connections from an agent (repeatable; at least one; idempotent)
```

`connection connect` takes either a provider template id (see `dam connection templates`) or an MCP server URL (`https://…`). `dam connection templates` lists only the ids `connect` accepts directly — MCP-category templates are left out on purpose, since MCP servers are added by URL instead. Useful flags:

- `--name <name>` — connection name (defaults to a slug of the template name or the MCP host)
- `--auth oauth|none` — MCP auth mode (auto-detected by default)
- `--no-browser` — for any OAuth connection, print the consent URL instead of opening a browser
- `--timeout <seconds>` — how long to wait for OAuth authorization (default 300)
- `-c, --config <key=value>` — set an optional template config input (repeatable), e.g. `--config model=premium-shell`; values are validated against the template's declared inputs.

Provider templates also declare their own required inputs, each exposed as its own flag — e.g. `--host`/`--url`, `--client-id`/`--client-secret`, `--header-name`/`--value`/`--value-format`/`--env-name`, `--ca-data`. Run `dam connection connect <provider> --help` for the set that applies; omitted secrets are prompted for securely.

`connection update` replaces a connection's **stored secret** in place. What that means depends on the connection: the injected value for an API key or custom-header integration, or an OAuth connection's own client secret where it carries one. A connection that stores no credential has nothing to update. It prompts securely on a TTY; pass `--value <secret>` when running non-interactively — a multi-line secret can't be typed at the prompt, so use `--value "$(cat secret.txt)"`.

Use `update` to replace a *secret you hold*, and `reauth` to replace *expired OAuth tokens*.

`connection reauth` re-runs login and consent for an **OAuth** connection, replacing only its tokens: the connection keeps its identity and every sandbox grant it already has, so nothing needs re-granting afterwards. Reach for it when a connection shows expired, or to re-consent after its template gained scopes. It opens a browser and waits for you to authorize:

```bash
dam connection reauth github
dam connection reauth conn-61cc7b9137b0 --no-browser   # print the URL instead of opening it
```

Flags: `--no-browser`, `--timeout <seconds>` (default 300), `--json`, `--server <url>`. A connection that stores its own secret has no login flow — `reauth` refuses it and points you at `connection update` instead.

### Network access

Per-agent rules that pre-approve the external hosts an agent may reach.

```bash
dam network list <agent>                               # List network access rules
dam network create <agent> --host api.example.com      # Add a rule (default method/path '*', verdict allow)
dam network update <rule-id>                           # Update an existing rule
dam network revoke <rule-id>                           # Remove a rule
dam network preset <agent>                             # Show the agent's effective preset
dam network apply-preset <agent> --preset <none|trusted|all> [-y]  # Apply a network preset ('all' needs -y when scripting)
dam network trusted-hosts                              # Platform-wide hosts seeded by the 'trusted' preset
```

`network create` flags: `--host <h>` (required), `--method <m>` (default `*`), `--path <p>` (default `*`), `--verdict allow|deny` (default `allow`), `-y, --yes` (skip the path-level restart confirmation), `--json`. A rule for a service that isn't on port 443 can't be created from the CLI — add it under **Network access** in the sandbox, where the host field takes `host:port`.

`network update` flags: `--method <m>`, `--path <p>`, `--verdict allow|deny`, `-y, --yes`, `--json` — partial, so pass only the fields you want to change. Updating a rule marks it manual.

`apply-preset` replaces only the rules the **preset** put there — rules you created by hand and rules derived from connections are preserved. The CLI prints that reminder before applying.

!!! note "A narrowed rule restarts the gateway"
    A rule that specifies a non-wildcard method or path needs path-level enforcement, which
    **restarts the sandbox's network gateway** (~5–15s) — the sandbox keeps running, but its outbound
    requests are briefly interrupted. The CLI confirms first; `-y` skips the prompt. Host-only rules
    don't restart anything. See [Network & Approvals](../core-concepts/network-access.md).

### Skills

Browse skill sources and manage the skills installed on an agent.

```bash
dam skill source list                                  # List connected skill sources
dam skill source add <git-url>                         # Register a skill source from a git URL
dam skill source remove <id>                           # Remove a source you added (installed skills remain)
dam skill source refresh <id>                          # Drop a source's scan cache so the next catalog re-scans upstream
dam skill catalog <source>                             # Scan a source's available skills
dam skill catalog <source> --agent <ref>               # Annotate install status against an agent
dam skill list <agent>                                 # List the skills on an agent
dam skill install <agent> --source <id-or-url> --name <skill>   # Install (or update) a skill (both flags required)
dam skill uninstall <agent> --source <id-or-url> --name <skill>   # Remove a skill (both flags required)
dam skill publish <agent> --name <skill> --source <id-or-url> [--title <t>] [--body <b>]   # Publish a skill as a PR (--name/--source required; --title/--body optional)
```

- `source add` also takes `--name <name>` and `--path <subdir>` to scan only a subdirectory of the repo. The name defaults to the git URL's path with a leading `/`, a trailing `/`, and a trailing `.git` removed — `https://github.com/acme/skills` becomes `acme/skills`. Slashes are kept; unlike connection names, skill-source names aren't slugified. On a TTY you're prompted with that value pre-filled.
- `source list --agent <ref>` additionally includes sources declared by the agent's template. `source remove` confirms on a TTY — pass `-y, --yes` non-interactively.
- `catalog --agent <ref>` annotates each skill as not-installed / installed / update-available **and** scans private sources through that agent's pod.
- `install` always installs the source's **current HEAD** — there is no `--version` flag and no separate `update` verb; re-running `install` updates in place.
- `publish` works only on **standalone** (locally-authored) skills, and only against **GitHub** sources. Its `--source` is the repo to open the pull request *against* — not where the skill came from, as it means on `install`/`uninstall`, since a standalone skill has no origin source.

### Channels

Bind an agent to a Slack conversation.

```bash
dam channel available                                       # List enabled messenger providers
dam channel list <agent>                                    # Bindings, with each Slack binding's ambient state
dam channel slack connect <agent> --channel-id <C0…>            # Bind the channel (mentions-only)
dam channel slack connect <agent> --channel-id <C0…> --ambient  # Bind with ambient mode on
dam channel slack disconnect <agent>                        # Unbind (when only one channel is connected)
dam channel slack disconnect <agent> --channel-id <C0…>     # Unbind one of several connected channels
```

- A Slack conversation binds to **exactly one** agent, but an agent may hold **several** Slack bindings at once. `disconnect` takes `--channel-id` to say which to release; it is required once the agent has more than one connected, and optional when it has just one. `disconnect` is idempotent — an agent that isn't connected exits 0.
- **The binding is the authorization**: anyone Slack admits to the conversation drives the agent, and every turn runs under the **agent's own** credentials. There is no per-person mode and no allow-list. See [Who can drive the agent](../integrations/slack.md#who-can-drive-the-agent).
- `--ambient` starts the binding in [ambient mode](../integrations/slack.md#ambient-mode) — the agent reads along and may reply without being mentioned. Omit it and ambient is **off**. Against a server too old to understand ambient, the connect **fails loudly** rather than silently landing mentions-only.
- **Re-running `connect` for the same agent and channel updates that binding in place**, which is how you change ambient mode from the CLI — no disconnect needed. Connecting a *further* channel to the same agent adds a binding instead of replacing one. Pointing a bound channel at a *different* agent fails with `Slack channel already bound to another agent`; `disconnect` first. (The in-chat `/dam bind` is stricter — it refuses any already-connected conversation, even for the same agent.)
- `dam channel list` prints `TYPE`, `IDENTIFIER`, and `AMBIENT`. Telegram rows show no identifier or ambient state — the bot token is write-only and ambient is Slack-only.
- **Binding from the CLI doesn't link your Slack identity**, which the in-chat `/dam unbind` and `/dam ambient on|off` require. Run `/dam login` once in the channel. Driving the agent itself never needs a link.
- [Telegram](../integrations/telegram.md) chats bind in-chat via `/dam bind` rather than through this command group.

### Approvals

The human-in-the-loop inbox — requests your agents are waiting on (held tool calls, network prompts).

```bash
dam approval list                                      # Pending requests across all your agents
dam approval list <agent> --status all                 # Include resolved and expired, for one agent
dam approval approve <id>                               # Approve — DURABLE: writes a permanent allow rule
dam approval approve <id> --once                        # Allow only this held call
dam approval approve <id> --entire-host                 # Allow the whole host (network requests only)
dam approval deny <id>                                  # Deny — DURABLE: writes a permanent deny rule
dam approval deny <id> --once                           # Deny only this held call
```

- `list` flags: `--status pending|resolved|expired|all` (default `pending`), `--limit <n>` (1–500).

!!! warning "Bare `approve` and `deny` are durable"
    For a **network** request, `dam approval approve <id>` writes a **permanent allow rule** — egress
    to that host/method/path stays open, and the rule then appears in `dam network list <agent>`.
    `deny` is the mirror. Use `--once` when you only mean to release (or refuse) the held call; the
    same request shape re-prompts next time.

    **Tool-call approvals never write an egress rule** — the harness owns its own permission
    persistence, and `--entire-host` falls back to the durable approve there.

- `--once` and `--entire-host` are mutually exclusive.

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

- `--weekdays <days>` — limit `--daily`/`--every` to specific days, e.g. `MO,WE,FR`. Can't combine with `--rrule`, whose body carries its own `BYDAY`. On `update` it only applies while you're rebuilding the recurrence, so pass it together with `--daily` or `--every` — you can't narrow the days of an existing schedule on its own
- `--timezone <tz>` — IANA timezone (defaults to the host's zone)
- `--quiet-window <HH:MM-HH:MM>` — suppress fires inside a window (repeatable)
- `--session-mode <fresh|continuous>` — new session per fire, or one shared session. Defaults to `fresh` on `create`; on `update`, omitting it leaves the stored value unchanged
- `--json` — emit the schedule as JSON (available on every subcommand)

On **`update`**, `--quiet-window` **replaces all** existing windows rather than adding to them, and at least one field must be passed. Legacy cron schedules are **read-only** — recreate them as RRULE schedules to edit.

`delete` takes `-y, --yes` to skip the confirmation prompt and **requires it on a non-TTY** — without it the command refuses and exits non-zero whatever the id. Past the confirmation it's idempotent: an unknown id exits 0. `enable`/`disable` are idempotent too and need no confirmation. Schedule ids come from `dam schedule list`.

### Terminal Sessions

```bash
dam chat <agent>                       # New terminal session
dam chat <agent> --continue            # Reattach to the most recent terminal session
dam chat <agent> --resume <session-id> # Resume a specific session
dam chat <agent> --reset               # Kill the existing PTY and start fresh
dam session list <agent>               # List all sessions for an agent (`dam session <agent>` works too)
```

`dam chat` connects your terminal directly to the agent's interactive TUI over WebSocket. Keystrokes (including Ctrl+C) pass through to the remote harness. On disconnect, the CLI prints a `dam chat --resume` command for easy reattach. Short flags: `-c` (`--continue`), `-r` (`--resume`).

### SSH access

```bash
dam ssh connect <agent>                # Open an SSH session to an agent
dam ssh connect <agent> -x code        # Launch VS Code Remote-SSH against the agent
dam ssh connect <agent> -x zed         # Launch Zed against the agent
dam ssh configure <agent>              # Write SSH host config for an agent (no launch)
dam ssh configure -a, --all            # Reconcile the active host's agents (prunes every other entry)
dam ssh configure --clear              # Remove all dam-managed SSH host config
```

`-x, --exec <bin[:mode]>` selects the client to launch: an executable name or path, optionally suffixed with `:ssh`, `:code`, or `:zed` to force how it's invoked. `dam ssh configure` writes the same host config without launching a client, and takes exactly one of an agent, `--all`, or `--clear`. Because there is a single managed config for all hosts, `--all` prunes entries for agents on your other servers too — see the [SSH guide](../guides/vs-code-zed-ssh.md#wiring-up-other-tools-scp-sftp-tunnels).

### File Operations

```bash
dam import <agent> <path...>           # Upload files/folders to the agent workspace
dam file list <agent> [<path>]         # List a workspace directory's immediate children
dam file get <agent> <remote-path>     # Download a single file from the agent
dam file put <agent> <local> <remote>  # Upload a single file to the agent
```

**Import** bundles local paths into a gzipped tar and uploads them. Each path lands at `work/<basename>` on the agent, replacing any existing entry at that path. Excluded by basename at every level: `node_modules`, `.venv`, `__pycache__`, `.DS_Store`. A symlink passed **as a `<path>` argument** is rejected and the import fails; symlinks found **inside** an imported directory are **skipped, not followed**. Use `-y, --yes` to skip the confirm (required on non-TTY).

**File list** shows a directory's immediate children and is **not recursive**; the path defaults to the workspace root. The default output lists only files — pass `--json` to include directories.

**File get/put** operate on single files. `file get` writes to `basename(remote-path)` in the current directory by default; use `-o, --output <path>` to choose the local path, `--stdout` to stream to stdout instead of writing a file, and `--overwrite` to clobber an existing local file. If `-o` names an existing **directory**, the file lands inside it under its remote basename, and `--overwrite` is ignored with `--stdout`. `file put` takes `--overwrite` to clobber an existing remote file.

### Usage metrics

```bash
dam metrics <agent>                     # Token spend, runtime, and context usage
dam metrics <agent> --session <id>      # Narrow to one session
dam metrics <agent> --since 168 --json  # Last 7 days, machine-readable
```

- `--since <hours>` is the lookback window (max 720; defaults to all time), `--limit <n>` caps how many recent calls are included (default 100, max 1000).

### Terms of Use

```bash
dam terms show                          # Current terms text, version, and your acceptance state
dam terms status                        # Exits non-zero if unaccepted — usable as a CI gate
dam terms accept                        # Interactive on a TTY
dam terms accept --yes                  # Accept without the prompt (required on non-TTY)
dam terms accept --expect-version <v>   # Fail if the server's version differs from the one you reviewed
```

!!! tip "If a command fails with `Terms of Use acceptance required`"
    Run `dam terms accept`. The CLI prints this hint itself, and `--yes` clears the gate headlessly
    so CI never needs a browser. `--expect-version` is deliberately not called `--version`, which is
    reserved for the CLI's own version.

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

The CLI probes the server's version before each networked command, and there are three outcomes:

- **Current enough** — nothing to see.
- **Behind the server, but at or above its advertised minimum** — a warning on stderr, and the command proceeds.
- **Below the minimum** — networked commands hard-fail with an upgrade prompt. Informational commands still run: `dam version` reports the same verdict but always exits 0, so you can diagnose the mismatch.

If the server advertises no minimum, the hard-fail never triggers. Run `npm install -g @dam-agents/cli@latest` to update.
