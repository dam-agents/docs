# Quickstart

Get a Claude Code agent running in 5 minutes.

=== "Web UI"

    ## Step 1: Log In

    Navigate to [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"} and sign in with your IBM SSO credentials.

    ## Step 2: Create a Claude Code Agent

    1. Click **"New Agent"**
    2. Select the **Claude Code** template
    3. Give it a name (e.g., `my-coding-agent`)
    4. Click **Create**

    Your agent will provision in a few seconds — you'll see the status change to "Running."

    ## Step 3: Add Your Anthropic API Key

    Claude Code needs an Anthropic API key to function:

    1. Navigate to **Connections** → **Secrets**
    2. Click **"Add Secret"**
    3. Select type: **Anthropic API Key**
    4. Paste your key
    5. Click **Save**

    !!! tip "Don't have an API key?"
        Visit [console.anthropic.com](https://console.anthropic.com){:target="_blank"} to create one.

    !!! info "Your key is safe"
        The key is stored in a secure vault and injected on the network wire by a separate gateway. Your agent never sees the raw token. Even if compromised, it can't leak your key.

    ## Step 4: Connect GitHub (optional but recommended)

    1. Go to **Connections**
    2. Click **"Connect GitHub"**
    3. Complete the OAuth flow
    4. Authorize repository access

    This lets your agent clone private repos, push branches, and create PRs.

    ## Step 5: Start Working

    Open your agent and type your first message:

    ```
    Clone github.ibm.com/my-org/my-project and give me a summary
    of the codebase architecture.
    ```

    Or click **"Terminal"** to launch the full Claude Code interactive TUI.

=== "CLI"

    ## Step 1: Install the CLI

    ```bash
    npm install -g @dam-agents/cli@0.1.0-rc4
    ```

    Requires Node 20 or later.

    ## Step 2: Configure and Log In

    ```bash
    dam config set server https://ibm.biz/dam-agents
    dam auth login
    ```

    A browser window will open for IBM SSO authentication. Confirm the device code to complete login.

    ## Step 3: Create a Claude Code Agent

    Interactive setup (walks you through template, API key, and GitHub PAT):

    ```bash
    dam agent create-interactive
    ```

    Or scripted:

    ```bash
    dam agent create my-coding-agent --template claude-code
    ```

    Wait for it to reach running state:

    ```bash
    dam agent create my-coding-agent --template claude-code --wait
    ```

    ## Step 4: Upload Your Project (optional)

    Import a local project into the agent's workspace:

    ```bash
    dam import my-coding-agent ./my-project
    ```

    This uploads the folder to `work/my-project` on the agent.

    ## Step 5: Start Working

    Attach your terminal to the agent:

    ```bash
    dam chat my-coding-agent
    ```

    You're now in the Claude Code interactive TUI — the same experience as running `claude` locally, but persistent in the cloud. Use slash commands, approve tool calls, and work as normal.

    To reattach later:

    ```bash
    dam chat my-coding-agent --continue
    ```

    ### Other Useful Commands

    ```bash
    dam agent list                    # List your agents
    dam session list my-coding-agent  # List sessions
    dam file list my-coding-agent     # Browse workspace files
    dam file get my-coding-agent work/output.txt  # Download a file
    ```

## What Just Happened

You now have:

- **A persistent workspace** — anything you clone or create lives here permanently (survives restarts and hibernation)
- **Secure credential injection** — your tokens are safe in a separate gateway pod
- **Network isolation** — your agent can only reach services you explicitly allow
- **Resumable sessions** — close the tab and come back anytime, the conversation persists

## What to Try Next

- **[Set up Slack](../integrations/slack.md)** — chat with your agent from Slack threads
- **[Create a schedule](../features/schedules.md)** — run tasks automatically (e.g., daily PR reviews)
- **[Install skills](../features/skills-and-tools.md)** — add specialized capabilities to your agent
- **[Learn about other agents](../agents/overview.md)** — Code Guardian, Pi Agent, Google Workspace, Bob, Codex
- **[CLI Reference](../reference/cli.md)** — full command documentation
