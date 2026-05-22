# Codex Agent

Codex is an OpenAI-powered coding agent running the Zed Industries Codex CLI with ACP integration. It supports any OpenAI-compatible API endpoint.

## Use Cases

- Code generation and editing with OpenAI models
- Working with custom OpenAI-compatible endpoints (Azure OpenAI, vLLM, etc.)
- Teams standardized on OpenAI's model ecosystem

## Required Connections

| Type | Purpose |
|------|---------|
| OpenAI API Key | Authenticates with the OpenAI API (or compatible endpoint) |

The API key is injected by the platform's credential gateway — it never appears in the pod environment.

## Configuration

| Variable | Purpose | Default |
|----------|---------|---------|
| `OPENAI_BASE_URL` | Custom API endpoint (for Azure, vLLM, etc.) | `https://api.openai.com/v1` |
| `OPENAI_MODEL` | Model to use | Provider default |

## Session Modes

- **Chat mode** — runs via `codex-acp`, communicating over ACP stdin/stdout
- **Terminal mode** — launches the interactive Codex TUI; supports session resumption with `codex resume --last`

## Features

- Full coding agent capabilities (read, write, execute)
- Compatible with any OpenAI API-compatible provider
- Session persistence across terminal reconnections
- 5 Gi workspace storage
