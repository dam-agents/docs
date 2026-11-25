# Roadmap

What the DAM team is working on now, what comes next, and what's further out.

DAM is a research project, and real users drive its direction. Much of what we build is exploratory: we try things, keep pushing on what works and gets traction, and change course or drop what doesn't. So expect this list to evolve as we learn.

## Now

- **Slack is a first-class place to work with DAM**: Get started with DAM, talk to your agents and get work done entirely in Slack, without hidden commands or a trip to the web app. That means creating your account without leaving Slack, finding what DAM can do without remembering commands, knowing which agent you're talking to, messages that never go missing, an install that works for anyone in the workspace, and agent posts that look right and can ask you things the way the web chat does. Tracked in [#6573](https://github.com/dam-agents/dam/issues/6573){:target="_blank"}, [#5058](https://github.com/dam-agents/dam/issues/5058){:target="_blank"}.

- **Starter kits are easy to choose, set up and grow with**: Pick the right kit quickly, get it set up in a few short turns, and keep shaping the agent afterwards: add or remove a kit, and ask it for more than its main job. Tracked in [#6576](https://github.com/dam-agents/dam/issues/6576){:target="_blank"}.

- **A new user gets a first useful result in their first session**: A new user, especially one who isn't technical, understands what DAM is for them and finishes a first useful task in their first session. The app says what DAM is, what it costs and what data you may use, and the assistant asks about your work, suggests uses that fit it and guides your first task to a result. Tracked in [#6838](https://github.com/dam-agents/dam/issues/6838){:target="_blank"}.

- **Every approval reaches the right person the same way, wherever they work**: When an agent needs a yes, the request reaches the person who must decide, in the web app, a Slack DM or the CLI, and waits until they answer. An answer in one place clears it everywhere, every gate uses the same words, and in a shared channel only the agent's owner and its editors can approve. Tracked in [#6828](https://github.com/dam-agents/dam/issues/6828){:target="_blank"}.

- **Internal deployment (IBM hosted)**: A second IBM hosted deployment of DAM that can reach systems inside IBM's network, GitHub Enterprise included. It runs alongside the current hosted deployment, not instead of it. This is about IBM's own deployment; the open source platform can already be deployed wherever you need it.

## Next

- **Agents that handle confidential data get limits the platform enforces**: Teams with sensitive data should be able to trust an agent because the platform keeps it safe, not because they checked every setting by hand. We're working through the open questions: which rules DAM enforces in code and which it hands to agents as policy; letting you say an agent works with confidential data, so DAM locks down its network and connections to match; agents that act with each person's own access instead of one shared account; and what an agent in a shared Slack channel can share, and with whom. First up are hard rules, such as which models an agent may use once it holds a given connection. Web searches come under the same control: an owner can already hold each web search a Claude Code agent makes for approval; next, every search goes through one DAM search tool that follows the agent's rules on any harness. Tracked in [#6016](https://github.com/dam-agents/dam/issues/6016){:target="_blank"}, [#3323](https://github.com/dam-agents/dam/issues/3323){:target="_blank"}, [#4311](https://github.com/dam-agents/dam/issues/4311){:target="_blank"}, [#6552](https://github.com/dam-agents/dam/issues/6552){:target="_blank"}.

- **An agent can ask the owner's other agents and hand them work**: An agent can ask one of your existing agents a question, or hand it a task, and get the answer or the result back. The other agent keeps its own context and access, so you no longer copy setup between agents or carry requests from one to another yourself. You decide which agents may call which, and you see every call. Later, your everyday questions go to whichever of your agents owns that area, and you always see which agent answered. Tracked in [#4409](https://github.com/dam-agents/dam/issues/4409){:target="_blank"}.

- **Users can't tell or control what keeps running after they leave**: Clear, predictable rules for what keeps running when you close the page or disconnect, and a way to see and control it. Tracked in [#4416](https://github.com/dam-agents/dam/issues/4416){:target="_blank"}.

- **Microsoft 365**: Agents that work with your Microsoft 365 account. The calendar comes first, so a morning brief knows your day and meeting prep finds the meeting and who is in it. Mail follows, and an agent sends a mail only after you approve it. Teams meeting transcripts come after that, so an agent can pull action items and decisions out of your meetings. On the IBM hosted deployment, this waits on an IBM security review. The open source platform is not affected. Tracked in [#6750](https://github.com/dam-agents/dam/issues/6750){:target="_blank"}.

- **Owners see and edit what an agent is about to send before it goes out**: Before an agent sends something for you, such as a GitHub issue, a Slack post or a mail, you see it as it will appear. You can change it or ask the agent to rework it, then send it, right away or later from one place. Tracked in [#6868](https://github.com/dam-agents/dam/issues/6868){:target="_blank"}.

- **Users know when to use artifacts, Environments or sub-agents**: An agent can publish a file, publish a page that talks back to it, run a sandbox you open by link, or start a sub-agent. Each was built separately, with its own name and its own sharing rules. Next, we're making them one clear set: you know which one to reach for, anyone who opens a shared link sees who made it and whose access it runs under, and all of them follow one set of sharing and security rules. Tracked in [#6628](https://github.com/dam-agents/dam/issues/6628){:target="_blank"}.

- **Events from outside DAM start agent runs**: Agents that start work when something happens, not only on a schedule: a label on an issue, a review request, a failing check, or a message from GitHub, Slack or a webhook. A cheap check runs first, so a trigger that finds nothing costs no model time. You can also @-mention an agent in a GitHub issue or pull request and it replies there, the way it does in Slack today. Tracked in [#3623](https://github.com/dam-agents/dam/issues/3623){:target="_blank"} and [#3482](https://github.com/dam-agents/dam/issues/3482){:target="_blank"}.

## Later

- **Every user has a built-in assistant that helps them use DAM**: A built-in assistant inside DAM. It answers questions about the platform, knows the state of your agents, and over time explains problems and helps with setup. Tracked in [#3587](https://github.com/dam-agents/dam/issues/3587){:target="_blank"}.

- **What a user teaches one agent is available to their other agents**: What you teach one agent, such as your preferences, rules and context, can carry over to your other agents, and a team can share what it agrees on. You see what is remembered and can trim it. This is still exploratory: we want to learn whether memory solves the problem before we decide what DAM builds and what it leaves to memory services. Tracked in [#6286](https://github.com/dam-agents/dam/issues/6286){:target="_blank"}.

- **Agents take part in a team's meetings**: A team's agent takes part in its meetings. It prepares an agenda from what it knows, follows the meeting, and afterwards records decisions and action items and follows up on them. The team decides when it joins and what it keeps. This is still exploratory. Tracked in [#6627](https://github.com/dam-agents/dam/issues/6627){:target="_blank"}.

- **Users know whether their agent runs on the right model and harness for its work**: Ask whether your agent runs on the right model and harness for what it actually does, and get an answer backed by evidence: a cheaper setup that does the same job, or a stronger one where the work needs it. Nothing changes until you agree, and switching is easy. Still exploratory. Tracked in [#6525](https://github.com/dam-agents/dam/issues/6525){:target="_blank"}.

## Have a request?

Feature requests and feedback are welcome on [github.com/dam-agents/dam](https://github.com/dam-agents/dam/issues){:target="_blank"}. IBMers can also post in [**#dam-community**](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}.
