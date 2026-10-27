# Roadmap

What the DAM team is working on now, what comes next, and what's further out.

DAM is a research project, and real users drive its direction. Much of what we build is exploratory: we try things, keep pushing on what works and gets traction, and change course or drop what doesn't. So expect this list to evolve as we learn.

## Now

- **Dead-simple first run**: Sign in and start chatting right away. New users get a ready-made agent with a starter Curve Bender budget, so there's no provider key to set up first. The budget shows in the UI, and you can switch to your own provider at any time. Tracked in [#5055](https://github.com/dam-agents/dam/issues/5055){:target="_blank"}.

- **New agent runtime (microVMs)**: Agents move to a VM-based runtime. The aim: an agent that starts in moments instead of making you wait, sizing you no longer have to pick up front, and agents that can run containers, so you can build and test real systems inside one. Tracked in [#4357](https://github.com/dam-agents/dam/issues/4357){:target="_blank"}.

- **Starter kits**: Ready-made setups for a specific job, such as reviewing code, keeping a knowledge base, or acting as a personal assistant. A kit brings the skills, connections, channels and schedules the job needs, so you start from a working agent rather than a blank one. You can also add your own kits, without an admin changing the platform. See [Agents](../core-concepts/agents.md). Tracked in [#5067](https://github.com/dam-agents/dam/issues/5067){:target="_blank"}, [#3576](https://github.com/dam-agents/dam/issues/3576){:target="_blank"}.

- **Use DAM from your own agent**: Drive DAM from the agent you already work in, such as Claude Code or Codex on your laptop. It learns DAM's command line from a skill, so it can create agents, start runs and manage connections for you. Secrets stay out of its hands: you enter them yourself. Tracked in [#5502](https://github.com/dam-agents/dam/issues/5502){:target="_blank"}.

- **Satellite agents**: Give your agents tools that run on a machine outside DAM, such as your laptop or a server. The machine connects out to DAM, so nothing has to reach in. Only the agents you add it to can use it. Satellites cover some cases of reaching systems DAM can't; the internal deployment below covers others. Tracked in [#3575](https://github.com/dam-agents/dam/issues/3575){:target="_blank"}.

- **Internal deployment**: A deployment of DAM that can reach systems inside IBM's network, GitHub Enterprise included. It runs alongside the current deployment, not instead of it. Tracked in [#3755](https://github.com/dam-agents/dam/issues/3755){:target="_blank"}.

- **Security for sensitive work**: Teams with sensitive data should be able to trust an agent because the platform keeps it safe, not because they checked every setting by hand. We're working through the open questions: which rules DAM enforces in code and which it hands to agents as policy; letting you say an agent works with confidential data, so DAM locks down its network and connections to match; agents that act with each person's own access instead of one shared account; and what an agent in a shared Slack channel can share, and with whom. First up are hard rules, such as which models an agent may use once it holds a given connection. Tracked in [#4166](https://github.com/dam-agents/dam/issues/4166){:target="_blank"}, [#3323](https://github.com/dam-agents/dam/issues/3323){:target="_blank"}.

## Next

- **Start from Slack**: Create your account and your first agent without leaving Slack. Tracked in [#5058](https://github.com/dam-agents/dam/issues/5058){:target="_blank"}.

- **Delegating work to sub-agents**: Handing work to other agents becomes a dependable, visible capability. An agent can split a task across sub-agents and pick the harness and model for each one, for example to run one prompt on two harnesses and compare the answers. You watch the whole tree, with each sub-agent's status and cost, and limits apply to the tree as a whole. A sub-agent that needs a person can ask you without holding up the others. Tracked in [#3419](https://github.com/dam-agents/dam/issues/3419){:target="_blank"}.

- **Agents working with your other agents**: An agent can ask one of your existing agents a question, or hand it a task, and get the answer or the result back. The other agent keeps its own context and access, so you no longer copy setup between agents or carry requests from one to another yourself. You decide which agents may call which, and you see every call. Later, your everyday questions go to whichever of your agents owns that area, and you always see which agent answered. Tracked in [#4409](https://github.com/dam-agents/dam/issues/4409){:target="_blank"}.

- **Shared agents and workspaces**: Working on one agent with other people. You'll be able to give someone a role on your agent, invite a co-builder, or hand the agent over to a new owner. Tracked in [#3218](https://github.com/dam-agents/dam/issues/3218){:target="_blank"}.

- **Background work that keeps running after you leave**: Clear, predictable rules for what keeps running when you close the page or disconnect, and a way to see and control it. Tracked in [#4416](https://github.com/dam-agents/dam/issues/4416){:target="_blank"}.

- **Self-serve telemetry data**: Pull everything DAM records about your own agents, meaning traces, logs and costs, and build your own tooling on it outside DAM. You only ever see your own agents' data, and the content of your conversations is never included. Usage questions get answered without waiting on a person: ask a telemetry agent in Slack, or open a dashboard that stays current. Tracked in [#3486](https://github.com/dam-agents/dam/issues/3486){:target="_blank"}.

- **Cost & performance optimization**: Ways to decide what an agent's work costs before it happens: run each task on a model that fits it, skip work that would find nothing, and set spending limits the platform enforces. Plus clearer answers to "why is my agent slow or expensive?" Tracked in [#4032](https://github.com/dam-agents/dam/issues/4032){:target="_blank"}, [#3322](https://github.com/dam-agents/dam/issues/3322){:target="_blank"}.

- **Software factory**: Agents that work on a team's repositories unattended: picking up issues, opening and reviewing pull requests, and asking a person when they need one. The DAM team is proving it on DAM's own repository first. Tracked in [#3679](https://github.com/dam-agents/dam/issues/3679){:target="_blank"}.

## Later

- **Triggers and GitHub as a channel**: Agents that start work when something happens, not only on a schedule: a label on an issue, a review request, a failing check, or a message from GitHub, Slack or a webhook. A cheap check runs first, so a trigger that finds nothing costs no model time. You can also @-mention an agent in a GitHub issue or pull request and it replies there, the way it does in Slack today. Tracked in [#3623](https://github.com/dam-agents/dam/issues/3623){:target="_blank"} and [#3482](https://github.com/dam-agents/dam/issues/3482){:target="_blank"}.

- **Interactive artifacts**: Artifacts that do more than display. A page you publish can call back to the agent that made it, so a dashboard can refresh itself or a form can kick off work. Private artifacts come first. Sharing interactive pages with others comes later, and it will never give viewers your agent's connections or credentials. Tracked in [#2884](https://github.com/dam-agents/dam/issues/2884){:target="_blank"}.

- **Smart platform**: A built-in assistant inside DAM. It answers questions about the platform, knows the state of your agents, and over time explains problems and helps with setup. Tracked in [#3587](https://github.com/dam-agents/dam/issues/3587){:target="_blank"}.

- **AI <> human collaboration in meetings**: Agents that work alongside people in meetings.

## Have a request?

Feature requests and feedback are welcome in [**#dam-community**](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}, or file them on [github.com/dam-agents/dam](https://github.com/dam-agents/dam/issues){:target="_blank"}.
