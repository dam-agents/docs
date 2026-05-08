# Quickstart

Get a Claude Code agent running in 5 minutes.

## Step 1: Log In

Navigate to [dam-dev.apps.dam-cl.fmaas.res.ibm.com](https://dam-dev.apps.dam-cl.fmaas.res.ibm.com/){:target="_blank"} and sign in with your IBM SSO credentials.

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

=== "Web Chat"

    ### Chat UI

    1. Open your agent
    2. Type your first message:

    ```
    Clone github.ibm.com/my-org/my-project into /workspace and give me
    a summary of the codebase architecture.
    ```

    3. The agent will respond in the chat interface — you're up and running.

    **Best for:** Quick tasks, file browsing, session history, team collaboration via Slack.

=== "Terminal"

    ### Terminal Session

    1. Open your agent
    2. Click **"Terminal"** to launch a PTY session
    3. You're now in the Claude Code interactive TUI — the same experience as running `claude` locally, but persistent in the cloud:

    ```
    $ claude
    ╭────────────────────────────────────────╮
    │ Claude Code                            │
    │ /workspace                             │
    ╰────────────────────────────────────────╯
    > Clone my-org/my-project and summarize the architecture
    ```

    4. Use slash commands (`/help`, `/clear`, `/compact`), approve tool calls interactively, and work just like you would locally.

    **Best for:** Power users, native Claude Code TUI experience, interactive debugging, full control over the agent.

## What Just Happened

You now have:

- **A persistent workspace** — anything you clone or create lives here permanently (survives restarts and hibernation)
- **Secure credential injection** — your tokens are safe in a separate gateway pod
- **Network isolation** — your agent can only reach services you explicitly allow
- **Resumable sessions** — close the tab and come back anytime, the conversation persists

## What to Try Next

- **[Set up Slack](../integrations/slack.md)** — chat with your agent from Slack threads
- **[Create a schedule](../features/schedules.md)** — run tasks automatically (e.g., daily PR reviews)
- **[Install skills](../features/skills.md)** — add specialized capabilities to your agent
- **[Learn about other agents](../agents/overview.md)** — Code Guardian, Pi Agent, Google Workspace
