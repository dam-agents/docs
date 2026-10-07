# Run Nous campaigns

[Nous](https://github.com/AI-native-Systems-Research/agentic-strategy-evolution){:target="_blank"} runs
hypothesis-driven experiment **campaigns** against a software system. Its own documentation covers what
a campaign is and how Nous works. This guide covers running it on DAM.

Nous is offered as a starter kit: open **Starter Kits** in the sidebar, pick the **Research** shelf, and
open **Nous**. The kit brings Nous pre-installed on its own image, so you create an agent from the kit
and give it your brief in a message. That agent is built for campaigns and nothing else — for ordinary
coding, use a different agent.

This 90-second demo walks through a whole campaign, from picking the kit to the finished report:

<video controls preload="metadata" poster="../../assets/nous-demo-poster.jpg" style="width: 100%; max-width: 880px; border-radius: 8px; margin: 1rem 0;">
  <source src="../../assets/nous-demo.mp4" type="video/mp4">
  Your browser doesn't support embedded video.
  <a href="../../assets/nous-demo.mp4">Download the demo</a> instead.
</video>

## What you need

- **A provider.** The kit runs on **IBM LiteLLM** or **Anthropic** — the provider picker offers only
  those two. See [Create your LiteLLM key](../reference/litellm-key.md).
- **Compute.** A campaign clones, builds and repeatedly runs the target, so the kit asks for more CPU,
  memory and disk than a chat agent. The kit's card on the create page shows what it reserves, and says
  how many of your [compute slots](../core-concepts/agents.md#compute-budget-and-resources) that is when
  it's more than one. CPU and memory stay editable afterwards; **disk is fixed at create**.
- **A GitHub connection** — optional. You need one to clone a **private** target, or to push results
  back. Any GitHub or GitHub Enterprise connection satisfies it: the OAuth sign-in, a personal access
  token or a GitHub App. See [Connections](../core-concepts/connections.md).

The agent holds no model credentials of its own. Its model calls, `git` and `gh` all authenticate
through the platform, so there is never an API key in the pod and nothing should ever ask you for one.

## Create the agent

Open the kit and click **Use this Starter Kit**. Its page lists what it brings under **Included** and
what it needs from you under **You'll set up**. On the **Create an agent** page you're asked for very
little:

1. **Name** — something you'll recognize in the agents list.
2. **Harness** — fixed, because the kit brings its own image.
3. **Provider** — your IBM LiteLLM or Anthropic connection.
4. **Connections** — a GitHub connection if you want one (see above).

Then **Create agent from this kit**. You don't choose network access while creating: the agent starts
on **Trusted defaults**, which already covers GitHub, PyPI and npm, and you can change it afterwards.

!!! tip "Allow the target's hosts before a long run"
    If building or exercising the target reaches a host outside the defaults, that request is held for
    your approval in **Activity** and denied if nobody answers in time — which an overnight run will hit.
    Add the rules up front on the agent's **Network access** section instead. See
    [Network & approvals](../core-concepts/network-access.md).

There's no **Lifecycle** choice to make here, and that's deliberate: the kit creates the agent
**Always on**, so it never hibernates while idle. A campaign launched in the background therefore runs
to completion on its own — you don't have to keep a session, terminal or SSH connection open. The
trade-off is that it holds its compute the whole time, so stop the agent when you're done with it.

## Your first campaign

Say what you want investigated, in your own words, naming the target and the question. A good first
target is the one Nous's own hello-world example uses, the BLIS
[inference simulator](https://github.com/inference-sim/inference-sim){:target="_blank"}:

```text
Run a campaign on github.com/inference-sim/inference-sim: with total input
length held fixed, does increasing the cached prefix reduce time-to-first-token
under moderate load? Keep it to two iterations.
```

You don't need to write a campaign file. Before launching, the agent gives the campaign its own
directory under `~/nous-campaigns`, clones the target into it, pins the models your provider actually
serves (Nous's own defaults usually aren't in your catalogue, so you don't need to name one), and shows
you the campaign with a rough duration. It waits for your go-ahead, so read the proposal rather than
waving it through: it's the cheapest moment to change the question or what's held fixed.

## While it runs

The campaign runs as a background process, so the agent stays conversational — ask it for status any
time, and ask it to stop and it halts cleanly at the next phase boundary. You can close the tab; the run
carries on. In a new conversation, the first thing it does is list the campaigns it has, which are
running, and which it can resume.

Phases are long, and the run log only writes between them, so ask the agent for status rather than
reading the log.

If the agent is connected to a [Slack channel](../integrations/slack.md) or
[Telegram chat](../integrations/telegram.md) when it launches a campaign, it posts each design and
findings summary to that channel or chat as a new message, as the campaign reaches it. The summaries
don't land in the thread where you started the campaign. Delivery is best-effort: a hiccup never
blocks the run.

!!! warning "Those summaries aren't asking you anything"
    Nous has two human gates — one on the design, one on the findings. **The agent always launches
    campaigns auto-approved**, so both pass on their own and a campaign runs unattended end to end. A
    summary that mentions waiting for approval is progress, not a question. If you reply "approve"
    anyway, the agent tells you the campaign is already proceeding — or, if it stopped after a restart,
    resumes it.

If the pod restarts for another reason — an image update, an eviction, a crash — campaign state
lives on the agent's persistent workspace. The next time you message the agent, it resumes the campaign
from its last checkpoint.

## Results

A campaign's output stays in its own directory under `~/nous-campaigns`, which you can browse in the
[File manager](../core-concepts/file-manager.md). Nous's
[data model](https://github.com/AI-native-Systems-Research/agentic-strategy-evolution/blob/main/docs/data-model.md){:target="_blank"}
explains what each file holds.

The Nous wiki commands ship with the agent, so you can ask it to index a finished campaign, render it as
an interactive page, or suggest the next experiment. The wiki and its rendered pages live under
`~/.nous/wiki`, outside the campaign directory.

A rendered page is an ordinary HTML file, so you can ask the agent to publish it to the
[artifact library](../core-concepts/artifacts.md). It starts private. Once you make it **Public**, you
get a link a colleague can open with no DAM account. Treat a public link as public: the URL is the only
thing protecting it.

## Going further

- **Your own repository.** Same prompt, your target. Private repositories need the GitHub connection
  above.
- **A running system instead of a repository**, such as a deployed service or a cluster. Say so in your
  brief, and make sure the agent's network rules let it reach that target.

For a hands-on shell alongside the agent — inspecting a clone, running the target by hand — connect over
[SSH or VS Code](../integrations/vs-code-zed-ssh.md); `nous` is on the path there.
