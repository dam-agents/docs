# Create your own starter kit

Turn an agent you have built into a starter kit: a repository that sets up a new agent the same way every time.

A starter kit is a repository with a `kit.yaml` at its root. The `kit.yaml` declares what the platform sets up before the agent's first turn: the connections it needs, its schedules, channels, size and network access. The rest of the repository is the agent's definition: its instructions, skills and scripts. See [Starter kits](../core-concepts/agents.md#starter-kits) for what a user sees when they create an agent from one.

You don't have to write any of this by hand. The DAM repository ships a skill for it, `dam-agent-creator`.

## What the skill does

Give it one sentence about what the agent should do. It then:

1. **Interviews you** about the job: the unit of work, what the agent reads and writes, whether it runs on a schedule or answers in a channel, who it talks to, and what it costs to run.
2. **Proposes a design** and waits for your approval. It writes no files before you approve.
3. **Writes the repository**: `kit.yaml`, the agent's `CLAUDE.md`, an onboarding runbook the agent follows on its first session, a README, procedure docs and scripts, a version and changelog, and CI.
4. **Checks it** with a validation script for the mistakes a kit most often has, such as a kit the catalog can't list, and adds the same checks to the repository's CI. The platform's own schema stays the final check.
5. **Hands you a checklist** for the next steps.

The skill is built for shared, unattended agents: a team's triage bot, a scheduled report, a channel helper. It isn't meant for a personal assistant.

## Requirements

- An agent with the `dam-agent-creator` skill.
- A GitHub repository for the kit.

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

## Share it

There are two separate ways to use a kit you built:

- **Import it in the UI, just for you.** Paste the kit's link into **Import from link** on **Starter Kits** and create an agent from it. Only you see it. Nothing is added to Starter Kits for anyone else, and no review or admin is involved.
- **Contribute it for everyone, by pull request.** Add the kit to the [dam-agents/dam-starter-kits](https://github.com/dam-agents/dam-starter-kits) catalog, which our deployment reads. Once your pull request is merged, the kit is on everyone's **Starter Kits** page, marked **By &lt;your name&gt;** so users can tell it from the DAM team's kits, which read **By DAM**.

Both need the kit's repository to be **public on github.com**, with `kit.yaml` at its root. If the kit seeds the agent from another repository, that one must be public on github.com too.

| | Import from link | Contribute by pull request |
|---|---|---|
| Who sees it | Only the person who imports the link | Everyone on the deployment, on the **Starter Kits** page |
| Review | None. The person importing it reviews it | The catalog's maintainers review your pull request |
| Shown as | A kit that's not in a catalog, which nobody on the platform reviewed | A user-contributed kit, **By &lt;your name&gt;** |
| Available | Right away | Once your pull request is merged |

Start by importing the kit yourself to try it. Contribute it when it's ready for everyone.

### Import from link (just for you)

1. Push the kit to a public repository on github.com.
2. Open **Starter Kits**, click **Import from link** beside the search, and paste the repository's link, for example `https://github.com/<you>/<repo>`. Link to the repository itself, not to a branch or a folder in it.

The kit opens with everything it sets up, a notice that nobody on the platform reviewed it, the repository and commit it was read from, and its install command and environment. The agent is created from the exact commit you saw.

An imported kit is not saved, and it never appears in the **Starter Kits** list, for you or anyone else. To use it again, import the link again. A teammate can do the same with your link, or with the address of the kit's page, which can itself be shared.

### Contribute it to Starter Kits (for everyone)

Besides the DAM team's built-in kits, the **Starter Kits** page lists the kits in the [dam-agents/dam-starter-kits](https://github.com/dam-agents/dam-starter-kits) catalog. To add yours, open a pull request there that lists your repository and names you as its publisher.

1. Push the kit to a public repository on github.com. Before you open the pull request, check that it works: create an agent from it with **Import from link**.
2. Fork [dam-agents/dam-starter-kits](https://github.com/dam-agents/dam-starter-kits) and add an entry to the end of the `kits:` list in `catalog.yaml`:

    ```yaml
    kits:
      # ...existing kits...
      - url: https://github.com/<you>/<repo>
        publisher:
          name: Jane Doe
          url: https://github.com/<you>
    ```

    - `url` is your kit's repository. Don't put a branch or a version in it.
    - `publisher.name` is exactly what users see on the kit's card and in its window, as **By Jane Doe**. Use your full name, or your team's name, so people recognize who built the kit. Write it as plain text, not a GitHub handle: your GitHub profile goes in `publisher.url`. Up to 80 characters.
    - `publisher.url` is optional and must start with `https://`. In the kit's window, your name links to it. Your GitHub profile is a good choice.

3. Open a pull request. Say what the kit does and who it's for. CI checks `catalog.yaml` against the catalog's schema.
4. Once a maintainer merges it, your kit appears on **Starter Kits** at the platform's next catalog refresh, within about 10 minutes. No redeploy is needed.

!!! important "Always add `publisher` for your own kit"
    A catalog entry without `publisher` is shown as a kit from the DAM team. Only the catalog entry sets who published a kit. `kit.yaml` has no field for it, so a kit can't claim to be the team's.

#### Keep it up to date

You don't need another pull request for an update. Push to your repository's default branch:

- New agents are created from the new commit after the next catalog refresh.
- If your kit seeds the agent from a repository and doesn't pin the seed to a commit, agents already created from it get a **Kit update** offer when that branch moves. Their owners choose which changes to take. Add a `VERSION` and a `CHANGELOG.md` so they see what changed.

To change your name or link, or to remove your kit from Starter Kits, open a pull request that edits or deletes your entry in `catalog.yaml`.

## Check your kit as you write it

Every `kit.yaml` starts with a schema line:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/dam-agents/dam-starter-kits/main/kit.schema.json
```

With the YAML extension, your editor then validates and autocompletes the file as you type. The schema is generated from the platform's own, so it matches what the platform accepts.

For every field a kit can declare, see [What a kit may declare](https://github.com/dam-agents/dam-starter-kits#what-a-kit-may-declare).
