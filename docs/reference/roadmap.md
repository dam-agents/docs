# Roadmap

What the DAM team is working on now, what comes next, and what's further out.

DAM is a research project, and real users drive its direction. Much of what we build is exploratory: we try things, keep pushing on what works and gets traction, and change course or drop what doesn't. So expect this list to evolve as we learn.

## Now

- **Slack as a first-class place to work**: Get started with DAM, talk to your agents and get work done entirely in Slack, without hidden commands or a trip to the web app. That means finding what DAM can do without remembering commands, knowing which agent you're talking to, messages that never go missing, an install that works for anyone in the workspace, and agent posts that look right and can ask you things the way the web chat does. Tracked in [#6573](https://github.com/dam-agents/dam/issues/6573){:target="_blank"}.

- **Starter kits that are easy to choose, set up and grow with**: Pick the right kit quickly, get it set up in a few short turns, and keep shaping the agent afterwards: add or remove a kit, and ask it for more than its main job. Tracked in [#6576](https://github.com/dam-agents/dam/issues/6576){:target="_blank"}.

- **Internal deployment (IBM hosted)**: A second IBM hosted deployment of DAM that can reach systems inside IBM's network, GitHub Enterprise included. It runs alongside the current hosted deployment, not instead of it. This is about IBM's own deployment; the open source platform can already be deployed wherever you need it.

- **Security for sensitive work**: Teams with sensitive data should be able to trust an agent because the platform keeps it safe, not because they checked every setting by hand. We're working through the open questions: which rules DAM enforces in code and which it hands to agents as policy; letting you say an agent works with confidential data, so DAM locks down its network and connections to match; agents that act with each person's own access instead of one shared account; and what an agent in a shared Slack channel can share, and with whom. First up are hard rules, such as which models an agent may use once it holds a given connection. Web searches come under the same control: an owner can already hold each web search a Claude Code agent makes for approval; next, every search goes through one DAM search tool that follows the agent's rules on any harness. Tracked in [#6016](https://github.com/dam-agents/dam/issues/6016){:target="_blank"}, [#3323](https://github.com/dam-agents/dam/issues/3323){:target="_blank"}, [#4311](https://github.com/dam-agents/dam/issues/4311){:target="_blank"}, [#6552](https://github.com/dam-agents/dam/issues/6552){:target="_blank"}.

- **Shared agents and workspaces**: Working on one agent with other people. You can already add editors to your agent in its **People** section. Next, you'll be able to hand the agent over to a new owner. Tracked in [#3218](https://github.com/dam-agents/dam/issues/3218){:target="_blank"}. — *Just shipped*

- **Start from Slack**: Create your account and reach your personal assistant without leaving Slack. Tracked in [#5058](https://github.com/dam-agents/dam/issues/5058){:target="_blank"}.

- **Delegating work to sub-agents**: Handing work to other agents becomes a dependable, visible capability. An agent can split a task across sub-agents and pick the harness and model for each one, for example to run one prompt on two harnesses and compare the answers. You watch the whole tree, with each sub-agent's status and cost, and limits apply to the tree as a whole. A sub-agent that needs a person can ask you without holding up the others. Tracked in [#3419](https://github.com/dam-agents/dam/issues/3419){:target="_blank"}. — *Just shipped*

## Next

- **Software factory**: Agents that work on a team's repositories unattended: picking up issues, opening and reviewing pull requests, and asking a person when they need one. The DAM team is proving it on DAM's own repository first. Tracked in [#6749](https://github.com/dam-agents/dam/issues/6749){:target="_blank"}.

- **Agents working with your other agents**: An agent can ask one of your existing agents a question, or hand it a task, and get the answer or the result back. The other agent keeps its own context and access, so you no longer copy setup between agents or carry requests from one to another yourself. You decide which agents may call which, and you see every call. Later, your everyday questions go to whichever of your agents owns that area, and you always see which agent answered. Tracked in [#4409](https://github.com/dam-agents/dam/issues/4409){:target="_blank"}.

- **Agents that ask you, and tell you what they did**: An agent that needs your approval or a decision asks you in the DAM inbox or in Slack, then keeps going with the rest of the task while it waits. The approval is still there when you come back, hours later if need be, and one click lets it finish. When you return, you can see what your agents did while you were away, what finished, and what needs you, without searching through every run. Tracked in [#3627](https://github.com/dam-agents/dam/issues/3627){:target="_blank"}, [#3100](https://github.com/dam-agents/dam/issues/3100){:target="_blank"}.

- **Background work that keeps running after you leave**: Clear, predictable rules for what keeps running when you close the page or disconnect, and a way to see and control it. Tracked in [#4416](https://github.com/dam-agents/dam/issues/4416){:target="_blank"}.

- **Microsoft 365**: Agents that work with your Microsoft 365 account. Meeting transcripts come first, so an agent can pull action items and decisions out of your meetings. Email and calendar follow, so a morning brief knows your day and meeting prep finds the meeting and who is in it. On the IBM hosted deployment, this waits on an IBM security review. The open source platform is not affected. Tracked in [#4405](https://github.com/dam-agents/dam/issues/4405){:target="_blank"}.

- **Spend you can trust**: Spending limits the platform enforces, a model per schedule, skipping scheduled runs that would find nothing, and a cost review an agent runs on itself are already in. After that, the spend you see has to match what your provider bills, and the spend screens have to show at a glance what you spent, which limits apply and how close you are to them. Tracked in [#6629](https://github.com/dam-agents/dam/issues/6629){:target="_blank"}, [#3322](https://github.com/dam-agents/dam/issues/3322){:target="_blank"}. — *Just shipped*

- **Artifacts, interactive artifacts, Environments and sub-agents that fit together**: An agent can publish a file, publish a page that talks back to it, run a sandbox you open by link, or start a sub-agent. Each was built separately, with its own name and its own sharing rules. Next, we're making them one clear set: you know which one to reach for, anyone who opens a shared link sees who made it and whose access it runs under, and all of them follow one set of sharing and security rules. Tracked in [#6628](https://github.com/dam-agents/dam/issues/6628){:target="_blank"}.

- **Use DAM from your own agent**: Drive DAM from the agent you already work in, such as Claude Code or Codex on your laptop. It learns DAM's command line from a skill, so it can create agents, start runs and manage connections for you. Secrets stay out of its hands: you enter them yourself. Tracked in [#5502](https://github.com/dam-agents/dam/issues/5502){:target="_blank"}.

- **Self-serve telemetry data**: Pull everything DAM records about your own agents, meaning traces, logs and costs, and build your own tooling on it outside DAM. You only ever see your own agents' data, and the content of your conversations is never included. Usage questions get answered without waiting on a person: ask a telemetry agent in Slack, or open a dashboard that stays current. Tracked in [#3486](https://github.com/dam-agents/dam/issues/3486){:target="_blank"}. — *Just shipped*

- **Triggers and GitHub as a channel**: Agents that start work when something happens, not only on a schedule: a label on an issue, a review request, a failing check, or a message from GitHub, Slack or a webhook. A cheap check runs first, so a trigger that finds nothing costs no model time. You can also @-mention an agent in a GitHub issue or pull request and it replies there, the way it does in Slack today. Tracked in [#3623](https://github.com/dam-agents/dam/issues/3623){:target="_blank"} and [#3482](https://github.com/dam-agents/dam/issues/3482){:target="_blank"}.

## Later

- **Smart platform**: A built-in assistant inside DAM. It answers questions about the platform, knows the state of your agents, and over time explains problems and helps with setup. Tracked in [#3587](https://github.com/dam-agents/dam/issues/3587){:target="_blank"}.

- **Memory across agents**: What you teach one agent, such as your preferences, rules and context, can carry over to your other agents, and a team can share what it agrees on. You see what is remembered and can trim it. This is still exploratory: we want to learn whether memory solves the problem before we decide what DAM builds and what it leaves to memory services. Tracked in [#6286](https://github.com/dam-agents/dam/issues/6286){:target="_blank"}.

- **Agents in your meetings**: A team's agent takes part in its meetings. It prepares an agenda from what it knows, follows the meeting, and afterwards records decisions and action items and follows up on them. The team decides when it joins and what it keeps. This is still exploratory. Tracked in [#6627](https://github.com/dam-agents/dam/issues/6627){:target="_blank"}.

- **Agent-written changes that do what you approved**: Before an agent implements a change, you approve the behavior it must deliver, as checks that run on their own. The agent can't pass by weakening those checks, and before merge an independent run shows the change passes them unchanged. This builds on the software factory and is still exploratory. Tracked in [#6558](https://github.com/dam-agents/dam/issues/6558){:target="_blank"}.

- **The right model and harness for the work**: Ask whether your agent runs on the right model and harness for what it actually does, and get an answer backed by evidence: a cheaper setup that does the same job, or a stronger one where the work needs it. Nothing changes until you agree, and switching is easy. Still exploratory. Tracked in [#6525](https://github.com/dam-agents/dam/issues/6525){:target="_blank"}.

## Have a request?

Feature requests and feedback are welcome on [github.com/dam-agents/dam](https://github.com/dam-agents/dam/issues){:target="_blank"}. IBMers can also post in [**#dam-community**](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}.
