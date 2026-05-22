# Custom Harness Image

DAM supports any AI agent that speaks the Agent Client Protocol (ACP). This guide explains how to package your own agent harness as a container image compatible with the platform.

## Architecture

Every DAM agent runs as a container built on top of `platform-base` — a shared base image that provides:

- Node 22 runtime, `git`, `gh` CLI
- The **agent-runtime** process (ACP WebSocket server, file service, trigger watcher)
- Default harness scripts for Claude Code
- Non-root user (UID 65532)

Your custom image extends `platform-base`, installs your harness, and optionally overrides the two entrypoint scripts.

## Harness Script Contract

The agent-runtime spawns two scripts at fixed paths depending on session mode:

| Path | Mode | Purpose |
|------|------|---------|
| `/usr/local/bin/harness-chat` | Chat | ACP communication over stdin/stdout |
| `/usr/local/bin/harness-terminal` | Terminal | Interactive TUI attached to a PTY |

**Chat mode** — your script must speak ACP over stdio. The platform routes messages between the UI/CLI and your harness via this protocol.

**Terminal mode** — your script receives a `HARNESS_SESSION_ID` environment variable. Use it to implement session resumption if your harness supports it.

## Dockerfile Template

```dockerfile
ARG BASE_IMAGE=platform-base
FROM ${BASE_IMAGE}

USER root

# 1. Install your harness
RUN npm install -g my-harness-package

# 2. Install ACP bridge (if your harness needs one)
RUN npm install -g my-harness-acp-bridge

# 3. Override harness scripts
COPY harness-chat.sh /usr/local/bin/harness-chat
COPY harness-terminal.sh /usr/local/bin/harness-terminal
RUN chmod +x /usr/local/bin/harness-chat /usr/local/bin/harness-terminal

# 4. Seed workspace (optional — copied to /home/agent/ on first pod boot)
COPY --chown=65532:0 workspace/ /app/working-dir/

# 5. Placeholder env vars for credentials (real values injected by Envoy sidecar)
ENV MY_API_KEY=dummy-placeholder

USER 65532
```

## Harness Script Examples

### Chat mode (`harness-chat.sh`)

```sh
#!/bin/sh
exec my-harness-acp "$@"
```

### Terminal mode (`harness-terminal.sh`)

```sh
#!/bin/sh
SESSION_MARKER="$HOME/.my-harness/session-started"
mkdir -p "$HOME/.my-harness" 2>/dev/null

if [ -f "$SESSION_MARKER" ]; then
  exec my-harness --resume "$HARNESS_SESSION_ID" "$@"
else
  touch "$SESSION_MARKER"
  exec my-harness "$@"
fi
```

## Workspace Seeding

Files placed in `/app/working-dir/` in your Dockerfile are copied to `/home/agent/` on first pod boot. Use this to provide:

- Default configuration files for your harness
- System prompts or instructions
- Pre-installed skills or extensions

Example layout:

```
workspace/
  .my-harness/
    config.json          # Harness config (→ ~/.my-harness/)
    SYSTEM.md            # System prompt
  work/
    README.md            # Initial files in the user's workspace
```

## Credentials

DAM never exposes real API keys inside the agent pod. Instead:

1. You declare placeholder environment variables in your Dockerfile (e.g., `ENV MY_API_KEY=dummy-placeholder`)
2. The platform's Envoy sidecar intercepts outbound requests and injects real credentials on the wire
3. Your harness starts normally with the placeholder — it only needs the env var to exist

This means your harness should check for the presence of credential env vars at startup but should not validate their values.

## Registering Your Template

Once your image is built and pushed to a registry, register it as a template in the platform's Helm values:

```yaml
agentTemplates:
  - name: my-custom-agent
    enabled: true
    description: "My custom AI agent"
    image:
      repository: quay.io/my-org/my-custom-agent
      tag: "1.0.0"
    storageSize: "5Gi"
    skillPaths:
      - $HOME/.my-harness/skills/
    env:
      - name: MY_API_KEY
        value: dummy-placeholder
```

After deploying the updated Helm chart, the template appears in the UI and CLI (`dam template list`).

## Key Constraints

- **ACP is required** — chat-mode harnesses must speak ACP over stdio
- **Two scripts only** — the platform spawns exactly `harness-chat` or `harness-terminal`; wrap any multi-process setup inside one of these
- **Non-root** — the container runs as UID 65532
- **PVC survives hibernation** — `/home/agent/` is persistent; ephemeral filesystem is lost when the pod scales down
- **No direct network access** — egress is controlled by NetworkPolicy; outbound traffic routes through the credential gateway

## Existing Examples

Refer to these built-in agent templates for real-world patterns:

| Agent | Key Pattern |
|-------|-------------|
| Claude Code | Minimal — uses default harness scripts, just installs the CLI |
| Pi Agent | Custom harness scripts for different chat/terminal modes |
| Codex | Custom OpenAI endpoint configuration via env vars |
| Bob | Heavy customization — Node 24, tenant scoping, budget caps |
