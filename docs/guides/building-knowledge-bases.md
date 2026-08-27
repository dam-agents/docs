# Building knowledge bases

A [knowledge base](../core-concepts/knowledge-bases.md) is an agent whose job is to hold what you or
your team knows and answer from it. You feed it sources; it organizes what it learns, keeps it, and
answers questions with citations back to where each answer came from.

This guide follows one real knowledge base — a **team memory** — from what it's for down to how a
meeting becomes a durable, queryable page. It's the shape that gets the most out of the platform,
because its value is in the **links**: who said a thing, which account they were from, and which
decision it fed. Two lighter shapes — a personal brain and a project base — come at the end as
variants once you've seen the full one.

## The example: a team-memory base

Picture a base that holds a product team's own thinking: customer meetings, user research, feedback,
personas, decisions, and the principles behind them. One team built exactly this and runs it as a
shared agent. Its job, in its own words, is to **be the team's memory** — and that mission is worth
stealing wholesale, because each line is a design decision you'll make too:

- **Capture** meetings, calls, research, feedback, and decisions as durable pages.
- **Attribute** everything — feedback to the person and account that gave it, a decision to whoever
  made the call. Never let a quote float free of who said it.
- **Find patterns** — the same complaint from three accounts is one roadmap item, not three notes.
- **Close the loop to where work happens** — make sure feedback lands as a tracked issue, and keep the
  link between the account that asked and the issue that tracks it.
- **Answer questions** from the base for anyone who asks.

Everything below is how you get an agent to do that.

!!! tip "Start narrow"
    A base holding one team's decisions *well* beats a base meant to hold everything. Scope is what
    keeps answers relevant — pick a single purpose before you create anything.

## Set it up

1. From **Knowledge bases**, click **Create knowledge base**.
2. Name it something you'll recognize in a Slack picker (`strategy`, not `kb-1`), and choose a
   template (below).
3. Click **Create knowledge base**. It starts on **Trusted defaults** for network access — the right
   setting here, since the template downloads its toolkit at setup and you'll grant specific hosts as
   you add sources.
4. Open it once created. It greets you and explains how to feed it.

### Choosing a template

Both templates are create-time choices you can't change later, so pick with the base's intended
lifetime in mind:

| Template | Pick it when |
|----------|--------------|
| **LLM Wiki** (default) | You want structure — a wiki with a knowledge graph linking what it learns. **This is the right choice for a team memory**: the graph is doing real work once there are people, accounts, and decisions to link. |
| **Plain Wiki** | You want durable markdown you could read straight out of the workspace, with no toolkit. Best for a base you'll skim by hand, or one that must keep working offline. |

## Give it its charter

The platform decides how a knowledge base stores and retrieves pages. It does **not** decide what your
base is *for*, how minimal to keep it, or what counts as an accurate entry — and those are the
decisions that separate a base people open from one they abandon. You set them once, in plain language,
and the agent holds them.

The team-memory base runs on four standing rules worth copying:

- **Keep it minimal.** A base nobody wants to open has failed, however complete it is. Facts, not
  scaffolding: if a heading has nothing real under it, there's no heading. One fact lives in one place,
  and other pages link to it rather than restating it — copied text drifts out of sync. Tell your agent
  that minimal outranks complete, and that it should *prune on write* rather than append forever.
- **Attribute or mark unknown.** A wrong attributed quote is worse than a missing one. Anything the
  agent doesn't actually know is written as `unknown`, never guessed — in who-said-it, in severity, in
  its answers. This one rule is what makes the base trustworthy enough to act on.
- **Don't over-report.** Not every observation is a problem. A question someone asked and got answered
  on a call is them learning the product, not a defect. Tell the agent: before it records a complaint,
  it should be able to imagine the person recognizing it as one.
- **Keep it confidential.** A team memory holds named people and private meeting content. It doesn't
  get published, shared, or quoted outside the team unless someone explicitly asks for that specific
  thing.

!!! info "Who can change it, and who can only ask"
    Decide early who's allowed to *change* the base versus only *query* it. The team-memory base treats
    a named set of people as **contributors** — they can ingest, correct, delete, and reset the rules —
    and treats everyone else as a **user** who gets full answers but can't alter content. A user's
    correction isn't refused; it's logged for a contributor to decide on. Setting this boundary is what
    lets you open the base widely without it drifting.

## Structure it so questions are answerable

The whole point of the LLM Wiki template is that a well-structured base can answer *patterns*, not just
recall single facts. Structure is what makes that possible. The team-memory base uses a small, fixed
set of page kinds — resist inventing more:

| Page kind | Holds |
|-----------|-------|
| **Account** | One customer/account's home page: who they are, and a running log of every touchpoint |
| **Person** | A stakeholder — created once they matter (2+ touchpoints or real influence), not on first mention |
| **Touchpoint** | One meeting or call. Links the account and each attendee. The atomic unit of capture |
| **Theme** | A recurring ask that shows up across accounts — created on its *third* occurrence, not its first |
| **Synthesis** | The output of a pattern question: "who's asking for X, and how badly" |

Three conventions turn that structure into something queryable:

- **One meeting, one page.** A call becomes a touchpoint page; each distinct piece of feedback becomes
  its own linked bullet on it. That granularity is what makes patterns findable later.
- **Every page links its entities.** A touchpoint links its account and each attendee; a theme links
  the feedback that evidences it. A page with no links is a dead end the graph can't reach.
- **Feedback is verbatim, and carries its metadata inline.** Each feedback item names who said it, the
  theme it belongs to, the issue tracking it, and how severe they said it was — on one line, so nothing
  floats free.

!!! tip "Let it grow into structure"
    You don't hand the agent this schema up front and drill it. You tell it the *purpose* and the
    minimal rules, feed it real meetings, and ask it to reorganize when a topic gets big enough to
    deserve its own page. The structure earns its way in — a theme page appears when the third account
    raises the same thing, not before.

## Feed it a meeting

The core habit is small and constant: after a call, hand the agent the notes and let it file them.

Tell it to:

1. **Create the touchpoint page** and link the account and everyone who attended.
2. **Pull out each distinct piece of feedback** as its own bullet — verbatim, attributed, with the
   severity the person themselves implied (or `unknown`).
3. **Match each item to a theme.** If the same complaint has now shown up a third time, that's when the
   theme page gets created and the earlier touchpoints get linked to it.
4. **Append the meeting to the account's log**, so the account page stays a complete history.

Done as a routine, this is what a good ingest reports back — one line, not a wall of text:

> *Logged the Acme call — 3 feedback items, 1 new theme, 2 that need issues filed.*

## Close the loop to where work happens

Capturing feedback isn't the goal; **acting on it** is. The team-memory base treats an unfiled piece of
feedback as unfinished work, and it's built to carry each item out to a tracked issue and keep the link
back.

- **File real asks as issues** in the repo where the team actually works — a usability complaint as a
  bug, a request for new capability as a feature.
- **Dedupe first, and dedupe hard.** Duplicate issues are the main failure mode: the same complaint
  arrives from several accounts, and most duplicates hide in the *closed* set. Tell the agent to search
  all issues — open and closed — and to read a candidate in full before calling anything a duplicate. A
  keyword match is not a duplicate.
- **Attach evidence.** When an issue already exists, the agent adds the new account as a datapoint
  rather than filing again — which accounts asked, and when, is exactly what makes an issue
  prioritizable.
- **Write the issue link back** into the feedback bullet. An unlinked feedback item is the thing this
  whole system exists to prevent.

!!! warning "Filing an issue is an outward-facing act"
    Issues are more public than the base. Have the agent confirm before it files, carry over the
    technical substance rather than internal politics or commercial terms, and ask before naming an
    account in a public repo. Nothing gets filed without an explicit go from someone allowed to give
    one.

## Reach it, and ask it for patterns

A team memory earns its keep when the team can reach it without opening DAM, and when it answers the
questions that are otherwise expensive to ask.

**Reach it from Slack.** A knowledge base is an ordinary agent, so it binds to a Slack conversation
like any other — invite the bot, run **`/dam bind`**, authenticate, and pick the base. Bound to the
team's **channel**, it answers on `@mention`, so a sweep result or an answer is visible to everyone
instead of making one person the relay. (Bound to a **1:1 DM**, every message reaches it — that's the
right surface for the personal variant below.)

**Ask it for patterns, not just recall.** This is the question a structured base exists to answer:

- *What are the three most-asked-for things across all accounts right now?*
- *Which complaint has come up after we thought we'd closed it?*
- *Which accounts have gone quiet, weighted by how much they matter?*

Grounded answers cite the touchpoints — "three top-tier accounts raised exports this month," with the
pages to prove it, not "customers seem frustrated."

**Republish views as artifacts.** Ask the base to publish recomputed views — an open-items list, a
leadership summary — to your [artifact library](../core-concepts/artifacts.md). An artifact keeps a
stable link, so a regenerated view is something the team can bookmark and share with people who have no
DAM account. Two rules make this work: **update the artifact in place** (a new one silently orphans
everyone holding the old link), and **recompute every number from the source pages** each time rather
than carrying the last version's counts forward.

!!! tip "Keep it on schedule"
    Add a [schedule](../core-concepts/schedules.md) so the base does its upkeep without anyone driving
    — a daily sweep for accounts going quiet or feedback still unfiled, posted to the channel. A good
    sweep post names itself, says what it checked in one line, and posts *nothing* on a quiet day.

---

## Lighter variants

The same machinery scales down. Once you've seen the team memory, these are small adjustments to scope,
sources, and reach — not different tools.

### A personal brain you DM in Slack

The most useful base is often the smallest: your own notes, meeting takeaways, and half-remembered
decisions, reachable from your phone.

- **Create it** like above; either template works — pick **Plain Wiki** if you'd rather your notes stay
  plain files you own.
- **Bind it to a 1:1 DM** with the DAM bot (`/dam bind` inside the DM). In a DM every message reaches
  the agent — no `@mention`, no ambient mode — so it reads like texting a colleague, and it stays
  private to you.
- **Feed it as you go:** after a meeting or decision, paste the gist into the DM and say *remember
  this*. Good sources are meeting takeaways, links and documents you'll want to ask about, and the
  reasoning behind decisions — the part you forget first. Upload files with the
  [File Manager](../core-concepts/file-manager.md) or `dam import`.

### A project base for a group

For a team working on one thing — a chip design programme, a research track, a product area — a shared
base becomes where the project's context lives, so nobody rebuilds it from scroll-back.

- **It lives in a channel, not a DM.** Bind it to the team's Slack channel so anyone can ask it; it
  answers on `@mention`.
- **It needs connections.** A project base usually reads from somewhere — a repo of design documents,
  an issue tracker. Grant the [connection](../core-concepts/connections.md) and allow the host under
  [network access](../core-concepts/network-access.md).
- **Its sources change**, so add a [schedule](../core-concepts/schedules.md) to re-read them on a
  cadence and keep it from drifting out of date.
- **Its highest-value content is result history** — what was tried, what came out of it, what was
  concluded. Feed it each result as it lands (the configuration, the numbers, the takeaway) and it can
  answer *what have we already tried, which approach won last time, has anyone hit this failure mode.*

!!! info "The same shape watches the outside world"
    Point that project-base machinery — connections, a re-read schedule, structure — at papers,
    publications, and blogs instead, and you have a **field-awareness base**: one place you and your
    agents can ask to stay on top of a space. Same setup; its sources are external and its job is to
    summarize the field rather than the team's own results.

---

## Best practices across all of them

- **One base, one purpose.** Two clear bases beat one that holds everything.
- **Feed it as you go.** A base built from a steady trickle is better organized than one loaded in a
  single dump.
- **Ask it to reorganize.** It's a conversation, not a schema — when a topic outgrows its page, say so.
- **Check the knowledge yourself.** It's just files in the workspace — browse or download them anytime
  to see what it actually holds.
- **Mind the single owner.** A knowledge base has one owner, and turns in a bound channel run under
  that owner's credentials. For a base the team depends on, choose that owner deliberately.
- **Export before deleting.** Deleting a base destroys its workspace and the knowledge with it.

## What's next

- [Knowledge Bases](../core-concepts/knowledge-bases.md) — templates, creation, and management in full
- [Slack](../integrations/slack.md) — binding, triggering, and what to expect in a conversation
- [Artifacts](../core-concepts/artifacts.md) — publishing summaries at stable links
- [Schedules](../core-concepts/schedules.md) — unattended sweeps and re-reads
