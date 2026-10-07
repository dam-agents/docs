# Sub-agents

An agent can hand a piece of work to a **sub-agent**: a separate agent, in its own sandbox, that runs
one task, reports one result back, and is then removed. You don't create sub-agents yourself. You ask
an agent to delegate, and it starts them. The chat calls each one a *temporary agent*.

## When an agent uses one

Most harnesses have their own built-in helpers, which work inside the agent's own sandbox, start in
seconds and use no extra compute. Agents are told to prefer those. A sub-agent is for work that needs
something only a separate agent gives:

- a different harness
- its own setup: a repository, an install command, environment variables, or skills
- more CPU, memory, or disk than the agent has
- isolation from the agent's own workspace
- heavy parallel work
- a result checked against a fixed shape

The agent decides which to use. If you specifically want separate agents, say so and say why. The
agent can also choose each sub-agent's harness and model, for instance a cheaper model for a mechanical
step, or the same task on two models to compare. It can hand work off directly, or write a script that
fans a lot of work out. Both show up the same way in the chat.

A sub-agent runs its task to the end without asking you anything, so a good request gives it
everything it needs up front.

## What you see

The agent's message in the chat carries a block that says how many temporary agents it delegated to,
and how many are still working or have failed. Each sub-agent gets a row showing the first line of its
task and its status:

| Status | Meaning |
|--------|---------|
| **Working** | It's running the task. |
| **Waiting for room** | It's waiting for compute to free up in your budget, and starts on its own once there's room. |
| **Done** | It reported a result. |
| **Failed** | It ended without a result. The row says why. |

Expand a row to see the **Prompt** it was given, its **Result** once done, and two actions:

- **Open conversation** opens the sub-agent's conversation beside the chat, read-only. It's available
  once the sub-agent has reported. While it's still working, the button is disabled. Occasionally a
  conversation isn't kept, and the button says so.
- **Stop** ends a running sub-agent after you confirm. It's removed, and the agent gets a failed result.

A sub-agent that delegates further shows its own sub-agents nested inside its row. The block stays in
the conversation after the sub-agents are gone, so you can come back to the results later.

## Where they run

Each sub-agent is an agent of its own, but it doesn't get a row on **Home**. While any are running, the
row of the agent that started them says how many temporary agents are running and what they're using.

A sub-agent is removed shortly after it reports. It also fails and is removed if it runs past the
time limit the agent gave it, if a setup step fails, or if it restarts. Deleting the agent that
started it fails its running sub-agents and removes them.

While an agent has sub-agents running, it doesn't hibernate, even if nobody is chatting with it.
Stopping or pausing it still works. If the agent handed work off directly and ended its turn before a
sub-agent finished, the result arrives later as a new turn in the conversation, and wakes the agent if
it needs to. A stopped agent
isn't started for it. The result waits until you start the agent again.

## What a sub-agent can use

- **Model provider**: the same one as the agent that started it. A harness that can't run on that
  provider is refused before anything is created.
- **Connections**: only the ones the agent passes on, and only from connections already granted to
  it. A sub-agent never gets more than its agent has.
- **Network access**: the [network rules](network-access.md) of the agent you asked. A request those
  rules neither allow nor deny raises an approval for that agent, and allowing it permanently adds the
  rule there.
- **Compute**: a sub-agent counts against your budget like any running agent. One too large for your
  whole budget is refused at once.
