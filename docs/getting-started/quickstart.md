# Quickstart

Launch a Claude Code agent in the cloud in minutes.

Prefer the command line? [Jump to CLI quickstart](#cli).

## Web UI

### Step 1: Log In

Navigate to [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"} and sign in with your IBM SSO credentials.

### Step 2: Add a Model Provider

1. Go to **Providers**
2. Click **"Add Provider"**
3. Select **IBM LiteLLM ETE Proxy**
4. Paste your LiteLLM API key
5. Click **Save**

!!! note
    Don't have a LiteLLM key yet? See [How to Create a LiteLLM Key](../guides/litellm-key.md) for a walkthrough.

### Step 3: Add Connections (optional but recommended)

Connections let your agent access your repositories — clone, push branches, and create PRs.

1. Go to **Connections**
2. Click **"Add Connection"**
3. Choose the GitHub type based on where your project lives:
    - **GitHub Public** — for repos on `github.com`
    - **GitHub Enterprise** — for repos on your company's GitHub Enterprise instance (e.g., `github.ibm.com`)
4. Provide a Personal Access Token (PAT) with appropriate repo permissions
5. Click **Save**

### Step 4: Create a Sandbox

1. Click **"Create sandbox"**
2. Choose **Claude Code** as the coding agent
3. Give it a name (e.g., `my-agent`)
4. Click **Create**

Your sandbox will provision in a few seconds — you'll see the status change to "Running."

### Step 5: Start Working

Open your agent and type your first message:

```
Clone github.ibm.com/my-org/my-project and give me a summary
of the codebase architecture.
```

Or click **"Terminal"** to launch the full Claude Code interactive TUI.

---

## CLI

### Step 1: Install the CLI

```bash
npm install -g @dam-agents/cli
```

Requires Node 20 or later.

### Step 2: Configure and Log In

```bash
dam config set server https://dam-prod.apps.dam-cl.fmaas.res.ibm.com/
dam auth login
```

A browser window will open for IBM SSO authentication. Confirm the device code to complete login.

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

This gives you a Remote-SSH window into the running agent — edit files, run the terminal, and debug with the agent's full environment behind it. See the [VS Code/Zed (SSH) guide](../guides/vs-code-zed-ssh.md) for prerequisites and details.

### Other Useful Commands

```bash
dam agent list                 # List your agents
dam agent get my-agent         # Show agent details
dam session list my-agent      # List sessions
```

---

## Next Steps

- **[Run code reviews automatically](../use-cases/code-review.md)** — set up an agent that reviews every PR
- **[Build a web app from scratch](../use-cases/web-app-development.md)** — describe an idea, let the agent build and deploy it
- **[Add Slack](../integrations/slack.md)** — let your team interact with shared agents
- **[Schedule tasks](../core-concepts/schedules.md)** — run agents on a timer with no human in the loop
