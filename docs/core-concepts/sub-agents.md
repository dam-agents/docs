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

A sub-agent can stop on a question for you, when it needs a decision only you can make. Its row then
reads **Needs your answer**, and you answer in the chat's message box, which names the sub-agent the
answer goes to. **Later** puts the question aside, and it stays above the message box. Without an
answer, the sub-agent goes ahead on a default it named. A good request still gives it everything it
needs up front.

## What you see

The agent's message in the chat carries a block that says how many temporary agents it delegated to,
and how many are still working or have failed. Each sub-agent gets a row showing the first line of its
task and its status:

| Status | Meaning |
|--------|---------|
| **Working** | It's running the task. |
| **Waiting for room** | It's waiting for memory to free up, and starts on its own once there's room. |
| **Needs your answer** | It asked you a question and waits for your answer. |
| **Done** | It reported a result. |
| **Failed** | It ended without a result. The row says why. |
| **Stopped** | You, or the agent that started it, stopped it. The row says who. |

Expand a row to see the **Prompt** it was given, its **Result** once done, and two actions:

- **Open conversation** opens the sub-agent's conversation beside the chat, read-only. It's available
  once the sub-agent has reported. While it's still working, the button is disabled. Occasionally a
  conversation isn't kept, and the button says so. From a finished one, **Continue in new chat**
  opens a new chat with the agent that carries the sub-agent's prompt and result into your first
  message.
- **Stop** ends a running sub-agent after you confirm, together with every sub-agent it started. It's
  removed, and the agent gets a failed result. The agent that started a sub-agent can stop it too.

A sub-agent that delegates further shows its own sub-agents nested inside its row. The block stays in
the conversation after the sub-agents are gone, so you can come back to the results later.

## Where they run

Each sub-agent is an agent of its own, but it doesn't get a row on **Home**. While any are running, the
row of the agent that started them says how many temporary agents are running and what they're using.

A sub-agent is removed shortly after it reports. It also fails and is removed if it runs past the
time limit the agent gave it, if a setup step fails, or if it restarts. Deleting the agent that
started it fails its running sub-agents and removes them.

While an agent has sub-agents running, it doesn't hibernate, even if nobody is chatting with it. If the
agent handed work off directly and ended its turn before a sub-agent finished, the result arrives later
as a new turn in the conversation, and wakes the agent if it needs to.

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
