# Skills

Skills are installable capabilities from git repos that extend what your agent can do.

## What Are Skills?

A skill is a packaged set of instructions and tools that teaches your agent a new capability. Examples:

- **review** — structured PR code review workflow
- **adr** — create architecture decision records
- **spec** — design features following project architecture
- **file-issue** — draft and file GitHub issues

## Skill Sources

| Source | Badge | Description |
|--------|-------|-------------|
| Platform | Blue | Pre-installed by the platform team |
| Agent | Purple | Bundled with the agent template |
| User | None | Added by you from any git repo |

## Installing a Skill

1. Open your agent's **Skills** panel
2. Browse available skills or add a new source (git repo URL)
3. Click **Install** on the skill you want

The skill files are downloaded to your agent's workspace and available immediately.

## Authoring Skills

You can create custom skills:

1. Create a directory with a `SKILL.md` file (YAML frontmatter with `name` and `description`)
2. Add supporting files (instructions, templates, examples)
3. Use the **Publish** button to submit as a PR to a connected git repo
