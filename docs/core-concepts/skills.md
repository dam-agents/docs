# Skills

Skills are packaged instructions from git repos that teach your agent new capabilities. Examples:

- **review** — structured PR code review workflow
- **adr** — create architecture decision records
- **spec** — design features following project architecture
- **file-issue** — draft and file GitHub issues

### Skill Sources

| Source | Badge | Description |
|--------|-------|-------------|
| Platform | Blue | Pre-installed by the platform team |
| Agent | Purple | Bundled with the coding agent image |
| User | None | Added by you from any git repo |

### Installing a Skill

1. Open your agent's **Skills** panel
2. Browse available skills or add a new source (git repo URL)
3. Click **Install**

Skills are downloaded to your agent's workspace and become available on the next session — the harness picks them up when it next starts, not mid-session.

!!! tip "Your agent can manage skills itself"
    Beyond the panel, the agent has built-in tools to discover and install skills mid-task — `list_skill_sources`, `list_skills_in_source`, `install_skill`, `uninstall_skill`, and `publish_skill` (open a PR adding a skill it authored to a source). These are always available; nothing to configure.

### Authoring Custom Skills

1. Create a directory with a `SKILL.md` file (YAML frontmatter with `name` and `description`)
2. Add supporting files (instructions, templates, examples)
3. Use the **Publish** button to submit as a PR to a connected git repo

### Where skills live in a source repo

When a git repo is scanned as a skill source, each skill is a directory containing a `SKILL.md`. The scanner looks under these source roots, in order:

1. `skills/`
2. `.claude/skills/`
3. `.agents/skills/`

Skills found across these roots are combined. If none of them contain skills, the scanner falls back to the repo's top-level directories. When the same skill `name` appears under more than one root, the first match wins (in the order above) and the later duplicate is dropped.
