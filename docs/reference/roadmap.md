# Roadmap

What the DAM team is working on now, what comes next, and what's further out.

!!! note "Plans change"
    This is a statement of direction, not a commitment to dates or scope. Items can move between columns, change shape, or drop off. For what you can use today, see the rest of these docs.

## Now

- **New agent runtime (microVMs)**: Agents move to a VM-based runtime. The aim: an agent that starts in moments instead of making you wait, sizing you no longer have to pick up front, and agents that can run containers, so you can build and test real systems inside one.

- **Starter kits**: Ready-made setups for a specific job, such as reviewing code, keeping a knowledge base, or acting as a personal assistant. A kit brings the skills, connections, channels and schedules the job needs, so you start from a working agent rather than a blank one. See [Agents](../core-concepts/agents.md).

- **Interactive artifacts**: Artifacts that do more than display. A page you publish can call back to the agent that made it, so a dashboard can refresh itself or a form can kick off work. Private artifacts come first. Sharing interactive pages with others comes later, and it will never give viewers your agent's connections or credentials.

- **Satellite agents**: Give your agents tools that run on a machine outside DAM, such as your laptop or a server. The machine connects out to DAM, so nothing has to reach in. Only the agents you add it to can use it.

## Next

- **Onboarding & first-run experience**: The product itself teaches a new user what DAM can do. Every empty page explains what its feature is for and gives you one clear next step.

- **Delegating work to sub-agents**: An agent hands parts of a task to other agents and sees each one's status and result in the conversation. Examples: running one task on two different harnesses to compare them, or sending a question to whichever of your agents owns that area.

- **Self-serve telemetry data**: You can pull your own agents' traces, logs and costs yourself, so an agent is no longer a black box. This also gives the team real usage data to base product decisions on.

- **Cost & performance optimization**: Ways to decide what an agent's work costs before it happens: run each task on a model that fits it, skip work that would find nothing, and set spending limits the platform enforces. Plus clearer answers to "why is my agent slow or expensive?"

- **Software factory**: Agents that work on a team's repositories unattended: picking up issues, opening and reviewing pull requests, and asking a person when they need one. The DAM team is proving it on DAM's own repository first.

- **Internal deployment**: A deployment of DAM that can reach systems inside IBM's network, GitHub Enterprise included. It runs alongside the current deployment, not instead of it.

- **GitHub as a channel**: Talk to an agent from GitHub: @-mention it in an issue or pull request and it replies there, the way it does in Slack today.

## Later

- **Shared agents and workspaces**: Working on one agent with other people. You'll be able to give someone a role on your agent, invite a co-builder, or hand the agent over to a new owner.

- **Smart platform**: A built-in assistant inside DAM. It answers questions about the platform, knows the state of your agents, and over time explains problems and helps with setup.

- **AI <> human collaboration in meetings**: Agents that work alongside people in meetings.

## Have a request?

Feature requests and feedback are welcome in [**#dam-community**](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}, or file them on [github.com/dam-agents/dam](https://github.com/dam-agents/dam/issues){:target="_blank"}. See [Contact](contact.md).
