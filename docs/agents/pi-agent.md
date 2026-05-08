# Pi Agent

Pi Agent is a multi-model coding agent powered by [pi.dev](https://pi.dev) that supports multiple LLM providers.

## When to Use Pi Agent

- When you want to choose between different models (GPT-4, Mistral, Gemini, Claude, Groq)
- When you need a specific model's strengths for a particular task
- For comparison testing across different LLM providers

## Required Connections

| Connection | Required? | Purpose |
|------------|-----------|---------|
| Model API key | **Yes** | At least one provider key (OpenAI, Anthropic, Mistral, Groq, or Google) |
| GitHub | Recommended | Clone repos, push branches, create PRs |

## Key Features

- **Multi-model support** — switch between providers with the `/model` command
- **Cross-session memory** — remembers your preferences, project context, and decisions across restarts
- **Persistent workspace** — same as all DAM agents

## Configuration

| Environment Variable | Purpose |
|---------------------|---------|
| `OPENAI_API_KEY` | OpenAI/GPT-4 access |
| `ANTHROPIC_API_KEY` | Claude access |
| `MISTRAL_API_KEY` | Mistral access |
| `GROQ_API_KEY` | Groq access |
| `GOOGLE_API_KEY` | Gemini access |

Credentials are injected via the credential gateway — the agent never sees raw keys.
