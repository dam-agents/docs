# Claude Code

Claude Code is a general-purpose coding agent powered by Anthropic's Claude. It excels at writing, debugging, refactoring, and explaining code.

## When to Use Claude Code

- Writing new features or modules
- Debugging issues (paste error messages, stack traces, or issue links)
- Refactoring and modernizing existing code
- Writing tests
- Code review and architecture exploration
- Documentation generation
- Interacting with GitHub (creating PRs, reading issues)

## Required Connections

| Connection | Required? | Purpose |
|------------|-----------|---------|
| Anthropic API key | **Yes** | Powers the Claude model behind the agent |
| GitHub | Recommended | Clone repos, push branches, create PRs |

See [Connections](../features/connections.md) for setup instructions.

## Getting Started

### 1. Clone Your Repo

Your first message should typically clone the project you want to work on:

```
Clone github.ibm.com/my-org/my-project and give me a summary
```

The agent will clone the repo into its persistent workspace. This only needs to happen once — the repo persists across sessions and hibernation.

### 2. Provide Context

Claude Code works best when it understands your project. You can:

- **Add a `CLAUDE.md`** file to your repo root with project conventions, architecture notes, and build commands
- **Point to documentation** — "Read the README and docs/ directory to understand this project"
- **Explain your setup** — "This is a TypeScript monorepo using pnpm, with packages in packages/"

### 3. Give It a Task

Be specific about what you want:

```
Fix the bug described in issue #42. The login form isn't validating
email addresses correctly. The relevant code is in src/auth/login.ts.
```

## Example Workflows

### Feature Development

```
I need to add a /health endpoint to the API server. It should:
- Return 200 with { status: "ok", uptime: <seconds> }
- Be unauthenticated (no JWT required)
- Include it in the existing Express router at packages/api/src/routes/

Write the implementation and tests.
```

### Bug Investigation

```
Users are reporting intermittent 500 errors on POST /api/orders.
The error log shows "TypeError: Cannot read property 'id' of undefined"
at orders-service.ts:142. Investigate the root cause and fix it.
```

### Code Review

```
Review the changes on branch feature/new-auth compared to main.
Focus on security concerns, error handling, and adherence to
our existing patterns.
```

### Refactoring

```
Refactor src/utils/date-helpers.ts to use date-fns instead of moment.
Maintain the same public API. Update all call sites and ensure tests pass.
```

## Tips for Best Results

- **Be specific** — include file paths, error messages, branch names
- **Share context** — link to issues, paste relevant logs, mention related files
- **Let it explore** — "Read through src/auth/ and explain how the middleware works" before asking for changes
- **Iterate** — if the first attempt isn't right, give feedback: "Good, but also handle the case where the token is expired"
- **Use CLAUDE.md** — project-level instructions persist across sessions and set expectations for code style, testing patterns, and conventions

## Skills

Claude Code supports installable skills that extend its capabilities. Common skills include:

- **review** — structured PR code review
- **adr** — create architecture decision records
- **spec** — design features following project architecture
- **file-issue** — draft and file GitHub issues

Browse and install skills from the **Skills** panel in the agent settings.

## Persistent Memory

Claude Code maintains a memory file at `/home/agent/.claude/memory.md`. This accumulates context across sessions — project decisions, your preferences, things you've taught it. You can view and edit this file in the File Manager.

## Configuration

| Environment Variable | Purpose | Default |
|---------------------|---------|---------|
| `ANTHROPIC_API_KEY` | API key for Claude (injected via credential gateway) | Required |
| `GITHUB_TOKEN` | GitHub access (injected via credential gateway) | Optional |

These are typically managed via Connections, not set manually.
