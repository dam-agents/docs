# Schedules

Run your agent on a recurring basis without manual intervention.

## Overview

Schedules let you define automated tasks that fire on a timer:

- Daily PR reviews at 9am
- Weekly email triage every Monday morning
- Hourly data sync checks

## Creating a Schedule

1. Open your agent's **Schedules** panel
2. Click **"New Schedule"**
3. Configure:
    - **Prompt** — what the agent should do when it wakes
    - **Frequency** — daily, weekdays, weekly, or custom (RRULE)
    - **Time and timezone**
    - **Quiet hours** — time windows when the schedule is suppressed
4. Click **Create**

!!! tip "Your agent can manage schedules itself"
    The agent has built-in tools to manage its own schedules — `list_schedules`, `create_schedule`, `toggle_schedule`, and `delete_schedule`. These create **persistent** schedules that run on the platform, survive restarts, and show up in this panel — distinct from any in-session timers a harness sets for itself, which vanish when the session ends.

## Session Modes

| Mode | Behavior |
|------|----------|
| **Fresh** | Each schedule fire creates a new session (agent starts with no prior context) |
| **Continuous** | All fires share one session (agent remembers prior runs) |

Use **fresh** for independent tasks (each PR review is standalone). Use **continuous** for cumulative work (tracking progress across days).

## Quiet Hours

Suppress schedule fires during specific time windows. Suppressed runs are **dropped, not deferred** — if a fire lands in quiet hours, it's skipped entirely (not queued for later).

**Example:** You have an hourly schedule that checks for new Slack messages. You set quiet hours from 10pm–7am. The 11pm, 12am, ... 6am runs simply don't happen. The next run fires at 7am as usual.

!!! warning "Dropped, not deferred"
    If you need every run to eventually execute, quiet hours is not the right tool. Use it for polling tasks where skipping off-hours runs is acceptable.

## Viewing History

Each schedule shows a log of past fires with links to the session that was created. You can review what the agent did on each run.
