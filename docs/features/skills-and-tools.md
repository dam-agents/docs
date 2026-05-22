# Skills & MCP Tools

Extend your agent with installable skills and built-in platform tools.

## Skills

Skills are packaged instructions from git repos that teach your agent new capabilities. Examples:

- **review** — structured PR code review workflow
- **adr** — create architecture decision records
- **spec** — design features following project architecture
- **file-issue** — draft and file GitHub issues

### Skill Sources

| Source | Badge | Description |
|--------|-------|-------------|
| Platform | Blue | Pre-installed by the platform team |
| Agent | Purple | Bundled with the agent template |
| User | None | Added by you from any git repo |

### Installing a Skill

1. Open your agent's **Skills** panel
2. Browse available skills or add a new source (git repo URL)
3. Click **Install**

Skills are downloaded to your agent's workspace and available immediately.

### Authoring Custom Skills

1. Create a directory with a `SKILL.md` file (YAML frontmatter with `name` and `description`)
2. Add supporting files (instructions, templates, examples)
3. Use the **Publish** button to submit as a PR to a connected git repo

## MCP Tools

Every agent exposes built-in tools via MCP (Model Context Protocol) for programmatic interaction with the platform:

| Tool | Description |
|------|-------------|
| `list_skill_sources` | List all configured skill sources |
| `list_skills_in_source` | List available skills within a source |
| `install_skill` | Install a skill from a source |
| `uninstall_skill` | Remove an installed skill |
| `publish_skill` | Publish a local skill to a git-based source |
| `send_channel_message` | Send a message to a connected channel (Slack, Telegram) |
| `describe_channel` | Get info about a connected channel |
| `list_schedules` | List all schedules for this agent |
| `create_schedule` | Create a new recurring schedule |

### Use Cases

- **Self-installing skills** — agent discovers and installs skills it needs mid-task
- **Proactive notifications** — agent messages your Slack channel when long-running tasks complete
- **Skill publishing** — agent publishes a skill it developed to a shared source

MCP tools are available automatically on every agent — no configuration required.
