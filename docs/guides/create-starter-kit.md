# Create your own starter kit

Turn an agent you have built into a starter kit, so anyone can create their own copy of it from **Starter Kits**.

A starter kit is a repository with a `kit.yaml` at its root. The `kit.yaml` declares what the platform sets up before the agent's first turn: the connections it needs, its schedules, channels, size and network access. The rest of the repository is the agent's definition: its instructions, skills and scripts. See [Starter kits](../core-concepts/agents.md#starter-kits) for what a user sees when they create an agent from one.

You don't have to write any of this by hand. The DAM repository ships a skill for it, `dam-agent-creator`.

## What the skill does

Give it one sentence about what the agent should do. It then:

1. **Interviews you** about the job: the unit of work, what the agent reads and writes, whether it runs on a schedule or answers in a channel, who it talks to, and what it costs to run.
2. **Proposes a design** and waits for your approval. It writes no files before you approve.
3. **Writes the repository**: `kit.yaml`, the agent's `CLAUDE.md`, an onboarding runbook the agent follows on its first session, a README, procedure docs and scripts, a version and changelog, and CI.
4. **Validates it** against the same rules the platform checks when it reads a kit, so a kit doesn't silently drop out of the catalog.
5. **Hands you a checklist** for publishing it.

The skill is built for shared, unattended agents: a team's triage bot, a scheduled report, a channel helper. It isn't meant for a personal assistant.

## Requirements

- An agent with the `dam-agent-creator` skill.
- A GitHub repository for the kit. It must be **public** if the kit seeds from its own repository, because the catalog is read without credentials. For a private definition, see [Private definitions](#private-definitions).

## Setup

### 1. Add the skill to your agent

1. Open your agent's **Skills** panel.
2. **Add source** with `https://github.com/dam-agents/dam`.
3. Turn on **dam-agent-creator**.

The skill is available from the agent's next session. See [Skills](../core-concepts/skills.md#sourced-from-github).

To use it locally instead, clone [dam-agents/dam](https://github.com/dam-agents/dam). The skill is in [`.agents/skills/dam-agent-creator`](https://github.com/dam-agents/dam/tree/main/.agents/skills/dam-agent-creator).

### 2. Run it

Start a new session and ask for the agent you want, in one sentence:

```
Use the dam-agent-creator skill. I want an agent that turns GitHub issues labelled "factory" into reviewed pull requests.
```

Answer the interview and approve the design. When it finishes, push the repository it wrote to GitHub.

### 3. List it in the catalog

The kits in **Starter Kits** come from the catalog in [dam-agents/dam-starter-kits](https://github.com/dam-agents/dam-starter-kits). Open a pull request that adds your repository to `catalog.yaml`:

```yaml
kits:
  - url: https://github.com/<org>/<repo>
```

If you can't push to that repository, open the pull request from a fork.

After it merges, the kit shows up in **Starter Kits** within minutes. No platform release is needed.

## Private definitions

A kit that seeds from its own repository needs that repository to be public. If the definition must stay private, put the `kit.yaml` in the catalog repository under `kits/<id>/kit.yaml`, list it as `- path: kits/<id>`, and point its `seed.url` at the private repository. The agent clones it through the user's GitHub connection.

## Check your kit as you write it

Every `kit.yaml` starts with a schema line:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/dam-agents/dam-starter-kits/main/kit.schema.json
```

With the YAML extension, your editor then validates and autocompletes the file as you type. The schema is generated from the platform's own, so it matches what the platform accepts. A kit that fails it is left out of the catalog.

For every field a kit can declare, see [What a kit may declare](https://github.com/dam-agents/dam-starter-kits#what-a-kit-may-declare).
