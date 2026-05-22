# Pi Agent

Pi Agent is a multi-model coding agent powered by [pi.dev](https://pi.dev) that supports 20+ AI providers.

## When to Use Pi Agent

- When you want to choose between different models (GPT-4, Mistral, Gemini, Claude, Groq, DeepSeek, etc.)
- When you need a specific model's strengths for a particular task
- For comparison testing across different LLM providers
- When connecting to self-hosted models (vLLM, Ollama, LM Studio)

## Required Connections

| Connection | Required? | Purpose |
|------------|-----------|---------|
| Model API key | **Yes** | At least one provider key (OpenAI, Anthropic, Mistral, Groq, Google, etc.) |
| GitHub | Recommended | Clone repos, push branches, create PRs |

## Supported Providers

Pi Agent supports a wide range of providers out of the box:

- OpenAI, Anthropic, Google (Gemini), Mistral, Groq, DeepSeek
- Azure OpenAI, AWS Bedrock, Google Vertex AI
- GitHub Copilot, Cohere, Together AI, Fireworks
- Self-hosted: vLLM, Ollama, LM Studio (via custom model config)

Switch between providers at any time with the `/model` command.

## Key Features

- **Multi-model support** — switch between 20+ providers with the `/model` command
- **Cross-session memory** — remembers your preferences, project context, and decisions across restarts
- **Custom system prompts** — override behavior via `SYSTEM.md` (replace) or `APPEND_SYSTEM.md` (append)
- **Persistent workspace** — same as all DAM agents
- **Skill support** — install skills at `~/.pi/agent/skills/`

## Memory System

Pi Agent includes a built-in memory system that persists across sessions:

- **Personal memory** stored at `~/.pi/agent/memory/`
- Includes `MEMORY.md`, `IDENTITY.md`, `USER.md`, and daily logs
- First-run bootstrap interview populates initial memory
- Use the `memory` tool to read, write, search, and list memory entries

## Configuration

| Environment Variable | Purpose |
|---------------------|---------|
| `OPENAI_API_KEY` | OpenAI/GPT-4 access |
| `ANTHROPIC_API_KEY` | Claude access |
| `MISTRAL_API_KEY` | Mistral access |
| `GROQ_API_KEY` | Groq access |
| `GOOGLE_API_KEY` | Gemini access |
| `DEEPSEEK_API_KEY` | DeepSeek access |

Credentials are injected via the credential gateway — the agent never sees raw keys.

### Custom Model Configuration

For self-hosted models or internal proxies, configure `~/.pi/agent/models.json`:

```json
{
  "my-vllm": {
    "provider": "openai-compatible",
    "baseUrl": "https://my-vllm.internal/v1",
    "model": "meta-llama/Llama-3-70B"
  }
}
```

### System Prompts

Customize Pi Agent's behavior with markdown files:

- `~/.pi/agent/SYSTEM.md` — replaces the default system prompt entirely
- `work/.pi/APPEND_SYSTEM.md` — appends to the system prompt (project-scoped)

## Default Model

The default model is `openai / gpt-5.4-mini`. Change it via the `/model` command or by editing `~/.pi/agent/settings.json`.
