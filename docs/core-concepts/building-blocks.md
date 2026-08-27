# The building blocks

Every agent in DAM is assembled from the same reusable pieces — compute, credentials, skills, memory, schedules, and more. These are the plumbing you'd otherwise rebuild for each new project, on one laptop at a time. DAM ships the pieces and you compose whatever your team needs, with no opinion baked in about what you build. Here's what you have to work with.

<div class="rows" markdown>

- **Isolated compute + persistent workspace** <span>A long-running Linux pod with a `~/work` directory that survives restarts and hibernation — the agent keeps working after you close your laptop. See [Agents](../core-concepts/agents.md).</span>
- **Network boundary & credential gateway** <span>The agent reaches only the services you allow, through a hardened gateway that holds your tokens and injects them on the wire — it never sees the raw credential. See [Network & Approvals](../core-concepts/network-access.md).</span>
- **Connections** <span>Wire in the accounts an agent acts through — GitHub, API keys, and more — without it ever seeing your tokens. See [Connections](../core-concepts/connections.md).</span>
- **Skills** <span>Packaged instructions that teach an agent a capability — plan a feature, draft a PRD, file an issue — installed once and reused. See [Skills](../core-concepts/skills.md).</span>
- **Schedules** <span>Run an agent unattended on a recurring timer — daily reviews, nightly audits, continuous monitoring — with no human in the loop. See [Schedules](../core-concepts/schedules.md).</span>
- **Channels** <span>Reach a running agent where the team already works — a Slack channel or DM, or a Telegram chat. See [Slack](../integrations/slack.md) and [Telegram](../integrations/telegram.md).</span>
- **Sessions** <span>Persistent chat and terminal conversations with an agent — the thread survives across restarts and hibernation. See [Sessions](../core-concepts/sessions.md).</span>
- **Files & memory** <span>Browse, edit, and upload files in the persistent workspace that holds the agent's configuration and memory across sessions. See [File Manager](../core-concepts/file-manager.md).</span>
- **Artifacts** <span>Finished outputs an agent publishes to keep — a report, a dashboard, a generated page, a data file — that outlive any single run. See [Artifacts](../core-concepts/artifacts.md).</span>
- **Experiments** <span>A design → build → test → learn loop the agent writes as ordinary Python, and the platform charts live — every stage, score, and candidate. See [Experiments](../core-concepts/experiments.md).</span>
- **Knowledge bases** <span>An agent whose job is to hold a body of knowledge you chat with, answering with citations back to the source. See [Knowledge Bases](../core-concepts/knowledge-bases.md).</span>

</div>

The blocks are the point. A coding assistant that reviews pull requests, a research agent that runs experiments, a knowledge base your whole team can query — these are all the same building blocks combined differently. DAM gives you the pieces; what you build with them is yours.

Put them together and you get the things our team actually runs today — a personal brain you DM, an automated code reviewer, a live optimization loop. The [guides](../guides/building-knowledge-bases.md) walk through building a few end to end.
