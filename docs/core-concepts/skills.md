# Skills

Skills are packaged instructions from git repos that teach your agent new capabilities. Examples:

- **plan-feature** — plan a feature before building it
- **adr** — create architecture decision records
- **write-a-prd** — draft a product requirements document
- **file-issue** — draft and file GitHub issues

Which skills you see depends on the sources your platform team set up and the ones you add yourself.

### Skill Sources

| Source | Can you remove it? |
|--------|--------------------|
| Set up by the platform team for everyone | No |
| Declared by your sandbox's template | No |
| A git repo you added yourself | Yes |

If you add your own source pointing at a repo the platform already provides, yours takes over the
listing; remove yours and the platform entry reappears.

### Installing a Skill

1. Open your sandbox's **Skills** panel
2. Browse available skills or add a new source (git repo URL)
3. Flip a skill's toggle on — it installs right away; flip it off to uninstall

Skills are downloaded to your sandbox's workspace and become available on the next session — the harness picks them up when it next starts, not mid-session. Installing always takes the source's **current HEAD**; re-running install is how you update. When a source moves ahead of what you installed, the skill shows an **Update** pill — click it to reinstall at the latest version.

!!! tip "Your agent can manage skills itself"
    Beyond the panel, the agent has built-in tools to discover and install skills mid-task — `list_skill_sources`, `list_skills_in_source`, `install_skill`, `uninstall_skill`, and `publish_skill` (open a PR adding a skill it authored to a source). These are always available; nothing to configure.

### Authoring Custom Skills

1. Create a directory with a `SKILL.md` file (YAML frontmatter with `name` and `description`)
2. Add supporting files (instructions, templates, examples)
3. Use the **Publish** button to submit as a PR to a connected git repo

You can also **upload Markdown files** directly — the Add-source dialog has an *Upload .md files* tab
that turns each file into its own skill. Skills you create or upload this way appear under
**Created in this sandbox**, separate from image-included skills and your git sources.

**Publish** opens a pull request against the source repo — it never commits directly. Only
GitHub-hosted sources can be published to, and the skill shows **In review** until the PR merges.

### Downloading or deleting a skill you made

Each skill under **Created in this sandbox** has a **⋯** menu:

- **Download skill** — a single-file skill comes down as `<name>.md`, which you can re-upload as-is;
  anything with extra files comes down as a `.zip` containing the skill's folder.
- **Delete skill** — removes it from the sandbox. If you had published it, the pull request stays
  open upstream; withdraw it yourself if you don't want it.

Skills installed from a git source aren't deleted this way — turn them off in their source card
instead.

### Skills that came with the image

Skills baked into your sandbox's image are listed separately under **Included with sandbox image**.
They're read-only: you can't publish them, and editing one only earns it a **Modified** badge, which
means the files on disk now differ from the copy the image shipped.

### Where skills live in a source repo

When a git repo is scanned as a skill source, each skill is a directory containing a `SKILL.md`. The scanner looks under these source roots, in order:

1. `skills/`
2. `.claude/skills/`
3. `.agents/skills/`

Skills found across these roots are combined. If none of them contain skills, the scanner falls back to the repo's top-level directories. When the same skill `name` appears under more than one root, the first match wins (in the order above) and the later duplicate is dropped.

If you point a source at a specific **subdirectory** when you add it, only that directory is scanned
and the roots above are skipped. Changing the subdirectory means removing the source and adding it
again.
