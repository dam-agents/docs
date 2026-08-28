# Building knowledge bases

A teammate that remembers everything your team learns — and answers when you ask.

A **knowledge base** is an agent that collects what your team knows, organizes it, and answers questions
from it with citations back to the source.

The most useful knowledge bases have one clear purpose. A project base might remember approaches,
results, and decisions. A team base might track customer conversations and recurring feedback. A
personal base can simply remember your own notes and decisions.

!!! tip "Start narrow"
    Pick one thing you want the base to know well. A focused base tends to give sharper, better-cited
    answers than one that tries to remember everything — and you can always add a second base later.

## A team memory, in practice

Imagine a shared agent sitting in your team's Slack channel.

After a meeting, you give it the notes. It captures the important details, links them to the right
people and accounts, and keeps the history organized. Later, anyone can ask:

- What are customers asking for most often?
- Which accounts have raised the same issue?
- What have we already tried, and which approach worked best?

The base doesn't just remember individual facts. Because related information is linked, it can surface
patterns across everything you've fed it.

That's the example we'll build below. Your job is to point it at the right things and keep feeding it —
you're the source and the editor, it's the librarian. The organizing, linking, and recall are its side
of the work.

## Where the knowledge actually lives

It's **plain files in the agent's workspace**, under `~/work` — not a database you can only reach
through the agent. You can browse and edit them yourself in the
[File Manager](../core-concepts/file-manager.md), and they survive hibernation and restarts.

That has one consequence worth planning for up front: **by default there is exactly one copy**, and
deleting the agent destroys it.

### Sync it to a Git repo

So point the base at a repo of your own. Create an empty one, grant the agent a
[GitHub connection](../core-concepts/connections.md), then tell it what you want:

> The knowledge lives in a git repo. After you file anything, commit it with a short message saying
> what changed, and push.

You get three things at once, which is why this is worth doing on day one rather than later:

- **A backup** that isn't the agent.
- **History** — a diff of how the knowledge changed, and who prompted each change.
- **A way out.** Cloning the repo is how you get everything at once; DAM's own File Manager downloads
  one file at a time.

The credentials never sit in the workspace — the connection injects them into the agent's outbound
requests, so `git push` just works. If you connected **GitHub.com**, DAM also writes your GitHub name
and email into the agent's git config, so commits are attributed to you rather than to an anonymous
pod identity. That write is best-effort and GitHub.com-only: on GitHub Enterprise, or if the lookup
didn't answer, set `user.name` and `user.email` in the agent yourself.

!!! tip "Make the repo private"
    Whatever your team tells the base ends up in it — customer names, unreleased plans, internal
    feedback. A private repo keeps the sync from becoming a disclosure.

## How locked down it is, is your call

Two separate controls decide what a knowledge base can reach, and both are yours from the start:

- **Network access** — the hosts it may reach. A knowledge base is created on **Trusted defaults**
  (GitHub, npm, PyPI, Anthropic and other common hosts). Switch it to **Strict default-deny** and
  every request needs your approval before it goes through, so you review each host as it comes up.
  Or allow just the specific hosts you want and nothing else.
- **Connections** — the credentials it holds. A knowledge base can only reach a system you've granted
  it a [connection](../core-concepts/connections.md) for. Grant none and it has no credentials at all.

So if you want it locked down: keep the connections list short, and either approve hosts one at a time
or name only the hosts it needs. See [network access](../core-concepts/network-access.md) for how rules
and approvals work.

!!! warning "Strict default-deny can block first-time setup"
    A new knowledge base downloads its toolkit from GitHub on creation. Create it on **Trusted
    defaults** and tighten afterwards, or the toolkit may not install.

## 1. Create the base

From **Knowledge bases**, click **Create knowledge base**. Choose a name you'll recognize in Slack,
such as `team-memory`.

Then pick a template. This is the one decision you can't undo later, so it's worth knowing what each
one actually installs.

A template is **not** a different kind of agent or a different engine. Both are the same Claude Code
agent, and both keep their knowledge as markdown files in the workspace. What differs is the *working
method* installed on top: a set of instructions the agent reads every session, and — for one of them —
a toolkit of commands and checks.

| | **LLM Wiki** (default) | **Plain Wiki** |
|---|---|---|
| **What gets installed** | A Claude Code **skill** with seven slash commands, plus session hooks and maintenance scripts. | Two files: an operating manual and a single onboarding command. Nothing else. |
| **How knowledge is organized** | An **interlinked graph** — pages carry structured frontmatter and `[[wikilinks]]`, with a generated index over them. | **Atomic markdown notes**, one topic per file, cross-linked by ordinary relative links, with an index you can read by hand. |
| **What you drive it with** | Commands as well as conversation: ingest a source, query, save an answer as its own page, render the graph, run a health check, work a review queue. | Conversation only. You tell it what to file and what to answer; there are no commands to learn. |
| **Where things land** | Sources you drop in stay immutable; compiled pages live beside them. | Raw material in one folder, durable notes in another — sources are evidence, notes are the memory. |
| **Upkeep** | The toolkit can audit itself: find broken links, orphaned pages, stale entries, malformed frontmatter. | Whatever you and the agent keep up by hand. |

The real trade-off is **structure versus simplicity**. LLM Wiki compiles knowledge once and keeps it
current, so it can answer across many pieces at once — "which accounts asked for this," "what have we
tried" — which is what this guide's example needs. It's also more machinery: more conventions, more
commands, more that can drift. Plain Wiki is a manual and a folder of notes, and it works entirely
offline once installed, with nothing to break; you just don't get the graph or the audits.

Pick **LLM Wiki** when the connections between facts are the point, **Plain Wiki** for a personal base
or a straightforward reference an agent tidies for you.

!!! note "Both are open, and neither is a black box"
    Each template is installed from a public repo by a one-shot setup script when the base is created —
    nothing runs a turn, and nothing is hidden. The knowledge, the instructions, and any commands all
    end up as files in the workspace, so you can read exactly what your base is following, and change
    it. That's also why either one syncs to a repo cleanly: it's all just files.

## 2. Give it a clear purpose

When you first open the base, it greets you and asks what it's for. This is where you set its
purpose — not in a settings field, but by telling it, in that first chat, what to remember and how
to behave.

Say something like:

> You're our team's memory. Our team builds products for AI Incubation. Our goal is to understand and track trends closely so we can make smart product decisions. Capture customer meetings and feedback, and follow these rules:

> - **Keep it minimal.** Store each fact once and link to it — don't copy it around.
> - **Attribute everything.** Record who said it, where it came from, and what it refers to. If something's unknown, write `unknown` rather than guessing.
> - **Keep it confidential.** Don't share anything outside the team unless I ask.

### Ask it to write that down

**Then say so explicitly:**

> Write these instructions into your own instructions file so you follow them in every conversation
> from now on, not just this one.

This matters more than it looks. A chat is one conversation's context — a Slack thread is its own
session, and starting a new chat starts fresh. Rules that live only in *this* conversation's history
won't reliably reach the next one. Asking the agent to record them puts them in a file it reads at the
start of every session, so the purpose sticks.

The same goes for anything you want it to keep behaving by — the team roster below, a naming
convention, a page you always want linked. **If it should outlive the conversation, ask it to write it
down.** It's a habit worth keeping past setup: when you correct the base months from now, add "and
remember that" and the correction holds.

### Tell it who your team is

This is the most valuable thing you can put in that first conversation, and it's the closest thing to
per-person permissions a knowledge base has.

There's no read-only role. Anyone who can reach the agent — everyone in a bound Slack channel, for
one — talks to it on equal footing and can ask it to change what it holds. So introduce your team to
it explicitly:

> Our team is:
>
> - **Priya Raman** (`@priya`, GitHub `praman`) — engineering lead. Can correct or delete anything.
> - **Dan Osei** (`@dan`, GitHub `daosei`) — PM. Can correct or delete anything.
> - **Everyone else in this channel** — ask questions freely. If they ask you to change or remove
>   knowledge, log it as a suggestion and flag it for Priya or Dan instead of applying it.
>
> Attribute what people tell you to them by name, and use these handles when you need to mention
> someone.

Roles, handles, and who may edit versus who may ask: agents follow instructions like these well. But
they *are* prompt engineering, not enforced rules — the real boundary is who's in the channel and who
has the base open in DAM. Set the door deliberately, then use these instructions to keep the inside
tidy.

## 3. Connect it to Slack

Do this early — Slack is where a knowledge base earns its keep, and it's easier to feed from there
than from the UI.

Invite the **DAM** bot to a conversation and send **`/dam bind`**, then pick the base from the picker.
Two shapes work well:

- **Your own 1:1 DM with the bot** — the best place to build a base up privately before anyone else
  sees it, and the right home for a personal base.
- **A team channel** — everyone can feed it and ask it questions where the work already happens.

Once bound, `@mention` the base with a question and the answer appears in the thread, with citations.
`@mention` it with notes and `file this` and it captures them.

You can bind both, and add the channel later — one agent can hold several bindings at once. See
[Slack](../integrations/slack.md) for the full command surface.

!!! tip "Let it read along"
    With **ambient mode** (`/dam ambient on`) the base follows the whole conversation and speaks up
    when it has something to add — useful in a channel where decisions get made in passing. It reads
    *every* message, so tell the channel when you turn it on.

## 4. Let the structure emerge

Don't start by building a giant schema. Start with the things your team actually works with, then let
the base grow as patterns appear.

A customer-focused base might use:

- **Account**: the customer's history
- **Person**: an important stakeholder
- **Touchpoint**: one meeting or call
- **Theme**: a recurring request or complaint
- **Synthesis**: an answer built from patterns across the base

A project memory might instead use **Approach** (something you're trying), **Result** (one run),
**Decision** (a decision and its reasoning), and **Synthesis**.

The names don't matter. The relationships do: **one event becomes one page, and every page links to
the things it relates to.** That's what makes patterns discoverable later.

## 5. Feed it as work happens

The best knowledge bases aren't built in one giant import. Feed them continuously — and with far less
ceremony than you'd expect.

Most of the time, no instructions at all. Drop meeting notes in the channel, `@mention` the base, and
it files them: creating the touchpoint, linking the account and attendees, capturing each piece of
feedback with its source, and attaching it to an existing theme or opening a new one. It'll tell you
what it did:

> Logged the Acme call: 3 feedback items, 1 new theme, 2 that need issues filed.

Be specific only when you want something specific — a particular framing, a page you want linked, a
detail you know matters and it can't infer.

!!! tip "Attach the file, don't paste it"
    Drop a transcript, spreadsheet, or PDF straight into the Slack message. Non-image files are saved
    into the agent's workspace and handed over as a path, so it opens them with ordinary tools —
    up to **20 MB per file** and 20 MB per message. Images are read directly (PNG, JPEG, GIF, WebP,
    up to 30 MB per message). Anything left out is named to you privately, so you're never guessing.

### Have it read Slack itself

The **Slack** card in the connection catalogue's **Apps** tab is a different thing from the binding
above: it connects *your* Slack account, so the base can search, read, post, and react as you. It's
backed by Slack's own MCP server, so it arrives as a set of tools the agent calls.

That means you can point it at channels it isn't bound to, and put it on a cadence:

> Read #customer-feedback and #support-escalations every weekday at 6pm. File anything new as feedback
> against the right account, and skip what you've already captured.

Pair it with a [schedule](../core-concepts/schedules.md) (step 8) and the base keeps itself current
without anyone remembering to paste. Note the Slack connection reads under your account, so it sees
what you can see.

## 6. Ask for patterns

This is where a structured knowledge base pays off:

- What are the three things customers ask for most often?
- What have we already tried on exports, and which worked best?
- Which complaints keep coming back?
- Which accounts have gone quiet?

Because the answers come from linked source pages, the agent can show you the evidence behind each one.

You can also ask it to publish a view — an open-items list, a leadership summary — to the
[artifact library](../core-concepts/artifacts.md), and **share it**: an artifact starts private, and
publishing it mints a link that needs no DAM account, so you can hand it to anyone. The link stays the
same when the base regenerates the contents, so a summary you've circulated keeps working as the
knowledge behind it changes.

!!! warning "A shared link is public"
    The URL's unguessable token is the only thing protecting it — no sign-in, no allowlist. Treat a
    shared artifact as public, and keep internal context out of one you circulate.

## 7. Turn knowledge into action

A knowledge base shouldn't just collect. When feedback represents a real product ask, connect it to
your issue tracker and have it:

- Search existing **open and closed** issues before creating one.
- Add new accounts as evidence when an issue already exists.
- Link the issue back to the original feedback.

!!! warning "Get confirmation before filing"
    Issues are outward facing. Have it confirm with you before filing, and hold back internal context.

## 8. Put it on a schedule

Ask the base for recurring work in plain language and it creates real
[schedules](../core-concepts/schedules.md) — they run on the platform, survive restarts, wake the agent
when they fire, and show up in its **Schedules** panel:

> Every weekday morning, check for unfiled feedback and accounts that have gone quiet. Every night,
> re-read the results repo.

A good scheduled sweep is quiet when there's nothing to report. The goal is signal, not a daily status
post.

!!! note "Say the timezone you mean"
    Ask for a local time and name the zone — "9am Eastern" — and the schedule holds that wall-clock
    time year-round, across daylight saving. Ask for a bare cron expression instead and it runs in
    **UTC** and ignores quiet hours. Schedules the agent creates for itself can't be edited in the
    panel; changes go through chat.

## Keeping it responsive

A knowledge base is an ordinary [agent](../core-concepts/agents.md), so it **hibernates after a period
of inactivity** and wakes on the next thing that needs it — a mention in Slack, opening it in DAM, a
schedule firing. Nothing is lost when it sleeps; the workspace comes back untouched. But a wake takes a
moment, so the first question after a quiet spell can be slow to answer.

For a base a team leans on all day, that wait is worth removing. Under **Agent Setup** → **Lifecycle**,
set the hibernation timeout to **`0`** — it then never hibernates and answers immediately, every time.

!!! warning "An always-on base holds its compute"
    A never-hibernating agent reserves its share of your compute budget even while idle, and DAM won't
    reclaim that room to start something else — it keeps it away from your other agents. Use `0` for a
    base your team genuinely queries all day; leave the default on a personal one, where a few seconds'
    wake is no bother.

## A personal base

Not every knowledge base needs to be shared. A personal base is a place for your own notes,
decisions, links, and meeting takeaways — created the same way, bound to your 1:1 DM with the bot.

From there, treat it like a colleague with a very good memory. *Remember this.* Later: *What did we
decide about the vendor, and why?* It answers from what you've given it, with a citation back to the
original note.

## Keep these principles in mind

- **One base, one purpose.**
- **Feed it continuously**, with as little ceremony as possible.
- **Let the structure evolve.**
- **Check what it actually knows.**
- **Set the door deliberately** — connections, network rules, and channel membership.
- **Sync it to a repo.** It's the backup, the history, and the way out.

Once the base is connected to the places your team already works, the goal is simple: **capture
knowledge once, make it easy to find, and keep it useful over time.**
