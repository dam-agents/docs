# Schedules

Run your sandbox on a recurring basis without manual intervention.

## Overview

Schedules let you define automated tasks that fire on a timer:

- Daily PR reviews at 9am
- Weekly email triage every Monday morning
- Hourly data sync checks

## Creating a Schedule

1. Open your sandbox's **Schedules** panel
2. Click **"Add Schedule"**
3. Configure:
    - **Prompt** — what the agent should do when it wakes
    - **Frequency** — every N minutes, hourly, daily, or custom (RRULE). For all but custom you also pick which days of the week apply (e.g. weekdays only).
    - **Time and timezone** — time is picked in half-hour steps; the timezone defaults to your browser's and can be any IANA zone. Schedules fire at that local wall-clock time year-round, adjusting across daylight saving.
    - **Quiet hours** — time windows when the schedule is suppressed
4. Click **Create**

!!! tip "Your agent can manage schedules itself"
    The agent has built-in tools to manage its own schedules — `list_schedules`, `create_schedule`, `toggle_schedule`, and `delete_schedule`. These create **persistent** schedules that run on the platform, survive restarts, and show up in this panel — distinct from any in-session timers a harness sets for itself, which vanish when the session ends.

!!! note "Older cron schedules run in UTC"
    Schedules you create in the panel always carry a timezone. Schedules defined with a raw cron
    expression (which an agent may do) run in **UTC** and ignore quiet hours — so a 9am-local cron
    drifts by an hour when daylight saving changes. They're also read-only once created: recreate
    them as a regular schedule to edit.

## Session Modes

| Mode | Behavior |
|------|----------|
| **Fresh** | Each schedule fire creates a new session (agent starts with no prior context) |
| **Continuous** | All fires share one session (agent remembers prior runs) |

Use **fresh** for independent tasks (each PR review is standalone). Use **continuous** for cumulative work (tracking progress across days).

**Fresh** is the default. Editing a schedule to clear its session mode reverts it to **fresh** — a previously set **continuous** mode does not persist through the edit. (This mainly affects edits made through the CLI or API; the panel always sends an explicit mode.)

Fresh runs can overlap if one is still going when the next fires; continuous runs queue behind each other.

**Starting over** — you can reset a continuous schedule so its next run begins a fresh session. The reset only lands while the sandbox is running: if it stays asleep, the reset lapses and the next run resumes the old session as before.

## Quiet Hours

Suppress schedule fires during specific time windows. Suppressed runs are **dropped, not deferred** — if a fire lands in quiet hours, it's skipped entirely (not queued for later).

**Example:** You have an hourly schedule that checks for new Slack messages. You set quiet hours from 10pm–7am. The 11pm, 12am, ... 6am runs simply don't happen. The next run fires at 7am as usual.

!!! warning "Dropped, not deferred"
    If you need every run to eventually execute, quiet hours is not the right tool. Use it for polling tasks where skipping off-hours runs is acceptable.

A window may cross midnight (10pm–7am works as written). If your quiet hours would swallow *every* run, the schedule won't save.

## Sleeping and stopped sandboxes

A schedule fire **wakes a hibernated sandbox** automatically — you don't need to keep it running. It
also restarts a sandbox you stopped by hand: stopping a sandbox does not pause its schedules, and the
UI warns you about this at stop time.

If a sandbox stays down past a run's window, that run is **dropped** rather than queued, so you won't
get a burst of catch-up runs when it eventually wakes.

## Viewing History

Each schedule lists the sessions it has produced — open one to see what happened on that run. A
**fresh** schedule accumulates one session per run; a **continuous** schedule has a single session you
can revisit. The panel also shows when the schedule last ran and whether that run succeeded.
