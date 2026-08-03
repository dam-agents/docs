# Quickstart

Launch a Claude Code agent in the cloud in minutes.

Prefer the command line? [Jump to CLI quickstart](#cli).

## Web UI

### Step 1: Log In

Navigate to [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"} and sign in with your IBM SSO credentials.

### Step 2: Add a Model Provider

1. Go to **Settings → Providers**
2. Find **IBM LiteLLM ETE Proxy** and click **Connect**
3. Paste your LiteLLM API key
4. Click **Save**

!!! note
    Don't have a LiteLLM key yet? See [How to Create a LiteLLM Key](../guides/litellm-key.md) for a walkthrough.

### Step 3: Add Connections (optional but recommended)

Connections let your agent access your repositories — clone, push branches, and create PRs.

1. Go to **Connections** (in Settings, alongside **Providers**)
2. Choose the GitHub connection that matches where your project lives:
    - **GitHub** — for repos on `github.com`
    - **GitHub Enterprise** — for repos on your company's GitHub Enterprise instance (e.g., `github.ibm.com`)
3. Complete the OAuth flow, or use **GitHub (Personal Access Token)** and paste a PAT with the repo permissions you need

Connecting doesn't yet give a sandbox access — **grant** the connection to each sandbox that should use it.

### Step 4: Create a Sandbox

1. Click **"Create sandbox"**
2. Choose **General-purpose sandbox**, then **Claude Code**
3. Name it (e.g., `my-agent`), pick your provider and a **Size**
4. Grant the connections it should use, then create

Your sandbox will provision in a few seconds — you'll see the status change to "Running."

### Step 5: Start Working

Open your agent and type your first message:

```
Clone github.ibm.com/my-org/my-project and give me a summary
of the codebase architecture.
```

Or use **Open in → Terminal (browser)** to launch the full Claude Code interactive TUI.

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
