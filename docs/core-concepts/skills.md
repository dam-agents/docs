# Skills

A **skill** is a folder of packaged instructions that teaches your agent a capability — how to plan a
feature, draft an ADR, write a PRD, file a GitHub issue. Your sandbox's **Skills** panel is where you
see and manage them.

## The three kinds of skill

Every skill in the panel arrived one of three ways, and the panel groups them accordingly. Which
group a skill is in decides what you can do with it, so this is the first thing to get straight:

| Group in the panel | Where it came from | Who manages it |
|---|---|---|
| **Created in this sandbox** | You authored it here — wrote the files in the workspace, or uploaded Markdown | Yours: edit, download, delete, publish |
| **Included with sandbox image** | Baked into the image your sandbox boots from | Read-only — comes and goes with the image |
| **Sourced from GitHub** | A git repo connected as a **skill source**; you install individual skills from it | Toggle each skill on or off; the repo is the source of truth |

The rest of this page takes them in that order.

## Created in this sandbox

Two ways to make your own:

- **Write it in the workspace.** A skill is a directory containing a `SKILL.md` — YAML frontmatter
  with `name` and `description`, then the instructions — plus any supporting files (templates,
  examples, scripts). Create it with the [file manager](file-manager.md) or just ask your agent to.
- **Upload Markdown.** The Add-source dialog has an **Upload .md files** tab that turns each file you
  give it into its own skill.

Either way it lands under **Created in this sandbox**. Each one's **⋯** menu offers:

- **Download skill** — a single-file skill comes down as `<name>.md`, ready to re-upload as-is;
  anything with extra files comes down as a `.zip` of the skill's folder.
- **Delete skill** — removes it from this sandbox.

### Publishing one

**Publish** opens a pull request against a connected GitHub source — it never commits directly. The
skill then shows **In review** with a link to the pull request. That pill records that you published
and isn't checked against GitHub afterwards, so it stays put once the PR is merged or closed — follow
the link for the real state. Only GitHub-hosted sources can be published to, so you need at least one
connected before the button does anything.

Publishing shares a skill; it doesn't move it. If you delete a skill you published, the pull request
stays open upstream — withdraw it yourself if you don't want it.

## Included with sandbox image

Some images ship with skills already inside them, so a fresh sandbox is useful immediately. Which
ones you get depends on the image you picked in the create wizard — a specialized image usually
carries the skills its job needs.

These are **read-only**. You can't publish them, and editing one only earns it a **Modified** badge,
meaning the files on disk now differ from the copy the image shipped. They aren't yours to delete
either — they arrive with the image and change when you
[update the image](agents.md#updating-the-image).

## Sourced from GitHub

A **skill source** is a git repo full of skills. Connecting one doesn't install anything; it makes
that repo's skills available to install individually.

1. Open your sandbox's **Skills** panel
2. Browse the skills in an existing source, or **Add source** with a git repo URL
3. Flip a skill's toggle on — it installs right away; flip it off to uninstall

Installed skills are downloaded into your sandbox and become available on the **next session** — the
harness picks them up when it next starts, not mid-session. Installing always takes the source's
**current HEAD**, so re-installing is how you update. When a source moves ahead of what you have, the
skill shows an **Update** pill; click it to reinstall at the latest version.

To remove one of these, turn it off in its source card — the **Delete skill** action is only for
skills you created here.

### Which sources you have

| Source | Can you remove it? |
|--------|--------------------|
| Set up by the platform team for everyone | No |
| Declared by your sandbox's template | No |
| A git repo you added yourself | Yes |

If you add your own source pointing at a repo the platform already provides, yours takes over the
listing; remove yours and the platform entry reappears.

### Where skills live in a source repo

When a repo is scanned as a source, each skill is a directory containing a `SKILL.md`. The scanner
looks under these roots, in order:

1. `skills/`
2. `.claude/skills/`
3. `.agents/skills/`

Skills found across these roots are combined. If none of them contain skills, the scanner falls back
to the repo's top-level directories. When the same skill `name` appears under more than one root, the
first match wins and the later duplicate is dropped.

If you point a source at a specific **subdirectory** when you add it, only that directory is scanned
and the roots above are skipped. Changing the subdirectory means removing the source and adding it
again.

!!! tip "Your agent can manage skills itself"
    Beyond the panel, the agent has built-in tools to discover and install skills mid-task — list
    sources, list a source's skills, install, uninstall, and publish a skill it authored as a pull
    request. These are always available; nothing to configure.
