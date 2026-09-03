# Quickstart

Launch a Claude Code agent in the cloud in minutes.

Prefer the command line? [Jump to CLI quickstart](#cli).

!!! note "Prerequisite: VPN"
    DAM requires VPN access. If you're outside an IBM office, connect to VPN before starting. If the
    site won't load, check your VPN connection first.

## Web UI

### Step 1: Log In

Navigate to [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"} and sign in with your IBM SSO credentials.

### Step 2: Create a Coding Agent

From the left sidebar, open **Coding agents** and click **Create coding agent**. Everything is on one
page — **Setup your coding agent** — filled top to bottom:

1. **Name** — what you'll call it.
2. **Image** — the harness that runs inside the pod. Pick the **Claude Code** card.
3. **Provider** — the model provider. Not connected yet? Connect it **right here** — paste your key
   inline, no trip to Settings.
4. **Connections** *(optional)* — grant credentials the agent may use. For working against your repos,
   add **GitHub** (for `github.com`) or **GitHub Enterprise** (for `github.ibm.com`) so it can clone,
   push branches, and open PRs.

!!! note
    No provider key yet? See [Create your LiteLLM key](../reference/litellm-key.md) — it covers which proxy
    to use and how to scope the key.

Click **Create coding agent**. Your agent provisions in a few seconds and you land straight in its chat.

### Step 3: Start Working

Type your first message:

```
Clone github.ibm.com/my-org/my-project and give me a summary
of the codebase architecture.
```

Or, on a session with no messages yet, use **Terminal (browser)** beside the message box to launch the
full Claude Code interactive TUI.

!!! tip "Want the full tour?"
    This gets you one working agent. To see the rest of the platform — connections, skills, schedules,
    Slack, artifacts, and more — built up one capability at a time, take the **[DAM new user tour](tour.md)**.

---

## CLI

### Step 1: Install the CLI

```bash
npm install -g @dam-agents/cli
```

Requires Node 20 or later.

### Step 2: Configure and Log In

```bash
dam config set server https://dam.res.ibm.com/
dam auth login
```

A browser window opens to authenticate. The CLI prints a user code — confirm it in the browser to
complete login. Pass `--no-browser` if you'd rather open the URL yourself.

### Step 3: Create a Claude Code Agent

Scripted (`--template` is required — list available templates with `dam template list`):

```bash
dam agent create my-agent --template claude-code
```

Wait for it to reach running state:

```bash
dam agent create my-agent --template claude-code --wait
```

Or run the interactive setup, which walks you through the name, template selection, and credentials:

```bash
dam agent create-interactive
```

### Step 4: Start Working

Attach your terminal to the agent:

```bash
dam chat my-agent
```

You're now in the Claude Code interactive TUI — the same experience as running `claude` locally, but persistent in the cloud. Use slash commands, approve tool calls, and work as normal.

To reattach later:

```bash
dam chat my-agent --continue
```

### Step 5: Open in Your IDE (optional)

Prefer to work in your editor? Open the agent's workspace in **VS Code** or **Zed** over SSH:

```bash
dam ssh connect my-agent -x code    # or: -x zed
```

This gives you a Remote-SSH window into the running agent — edit files, run the terminal, and debug with the agent's full environment behind it. See the [VS Code/Zed (SSH) guide](../integrations/vs-code-zed-ssh.md) for prerequisites and details.

### Other Useful Commands

```bash
dam agent list                 # List your agents
dam agent get my-agent         # Show agent details
dam session list my-agent      # List sessions
```

!!! tip "Want the full tour?"
    This gets you one working agent. To see the rest of the platform — connections, skills, schedules,
    Slack, artifacts, and more — built up one capability at a time, take the **[DAM new user tour](tour.md)**.

---

## Next Steps

- **[Build a knowledge base](../guides/building-knowledge-bases.md)** — a personal brain, a project base, or a team's memory you can ask
- **[Run your first experiment](../guides/run-experiments.md)** — walk a small optimization loop end to end
- **[Run code reviews automatically](../guides/code-review.md)** — set up an agent that reviews every PR
- **[Add Slack](../integrations/slack.md)** — let your team interact with shared agents
- **[Schedule tasks](../core-concepts/schedules.md)** — run agents on a timer with no human in the loop
