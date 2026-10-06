# Roadmap

What the DAM team is working on now, what comes next, and what's further out.

DAM is a research project, and real users drive its direction. Much of what we build is exploratory: we try things, keep pushing on what works and gets traction, and change course or drop what doesn't. So expect this list to evolve as we learn.

## Now

- **New agent runtime (microVMs)**: Agents move to a VM-based runtime. The aim: an agent that starts in moments instead of making you wait, sizing you no longer have to pick up front, and agents that can run containers, so you can build and test real systems inside one.

- **Starter kits**: Ready-made setups for a specific job, such as reviewing code, keeping a knowledge base, or acting as a personal assistant. A kit brings the skills, connections, channels and schedules the job needs, so you start from a working agent rather than a blank one. See [Agents](../core-concepts/agents.md).

- **Interactive artifacts**: Artifacts that do more than display. A page you publish can call back to the agent that made it, so a dashboard can refresh itself or a form can kick off work. Private artifacts come first. Sharing interactive pages with others comes later, and it will never give viewers your agent's connections or credentials.

- **Satellite agents**: Give your agents tools that run on a machine outside DAM, such as your laptop or a server. The machine connects out to DAM, so nothing has to reach in. Only the agents you add it to can use it.

## Next

- **Onboarding & first-run experience**: A shorter, simpler path to your first working agent, so you get value sooner. Plus an optional guided experience for getting started with DAM.

- **Delegating work to sub-agents**: Handing work to other agents becomes a dependable, visible capability. An agent can split a task across sub-agents and pick the harness and model for each one, for example to run one prompt on two harnesses and compare the answers. You watch the whole tree, with each sub-agent's status and cost, and limits apply to the tree as a whole. A sub-agent that needs a person can ask you without holding up the others. An agent can also call one of your existing agents that already has the context. Later, your everyday questions go to whichever of your agents owns that area, and you always see which agent answered.

- **Self-serve telemetry data**: Pull everything DAM records about your own agents, meaning traces, logs and costs, and build your own tooling on it outside DAM. You only ever see your own agents' data, and the content of your conversations is never included. Usage questions get answered without waiting on a person: ask a telemetry agent in Slack, or open a dashboard that stays current.

- **Cost & performance optimization**: Ways to decide what an agent's work costs before it happens: run each task on a model that fits it, skip work that would find nothing, and set spending limits the platform enforces. Plus clearer answers to "why is my agent slow or expensive?"

- **Software factory**: Agents that work on a team's repositories unattended: picking up issues, opening and reviewing pull requests, and asking a person when they need one. The DAM team is proving it on DAM's own repository first.

- **Internal deployment**: A deployment of DAM that can reach systems inside IBM's network, GitHub Enterprise included. It runs alongside the current deployment, not instead of it.

- **GitHub as a channel**: Talk to an agent from GitHub: @-mention it in an issue or pull request and it replies there, the way it does in Slack today.

## Later

- **Shared agents and workspaces**: Working on one agent with other people. You'll be able to give someone a role on your agent, invite a co-builder, or hand the agent over to a new owner.

- **Smart platform**: A built-in assistant inside DAM. It answers questions about the platform, knows the state of your agents, and over time explains problems and helps with setup.

- **AI <> human collaboration in meetings**: Agents that work alongside people in meetings.

## Have a request?

Feature requests and feedback are welcome in [**#dam-community**](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}, or file them on [github.com/dam-agents/dam](https://github.com/dam-agents/dam/issues){:target="_blank"}.
