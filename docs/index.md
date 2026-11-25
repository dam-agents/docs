<div class="welcome-hero" markdown>

<p class="welcome-eyebrow">Open source, from IBM Research</p>

# Welcome to DAM

<p class="welcome-lead">AI agents that work for you on their own: on a schedule, in Slack, and after you close your laptop.</p>

<div class="welcome-cta" markdown>
[Launch DAM for IBMers →](https://ibm.biz/dam-agents){:target="_blank" .md-button .md-button--primary}
[:material-source-branch: Deploy it yourself](https://github.com/dam-agents/dam#run-your-own){:target="_blank" .md-button}
</div>

<p class="welcome-ibm">IBMers on the IBM hosted deployment: get help in <a href="https://ibm.enterprise.slack.com/archives/C0B3F03NB24" target="_blank">#dam-community</a> on IBM Slack, and check <a href="https://ibm.biz/dam-status" target="_blank">DAM status</a> if something isn't working.</p>

</div>

## What is DAM?

**DAM gives you AI agents that do work for you, not just answer questions.** An agent can send you a
morning brief, keep a knowledge base of your project you can ask questions about, review every pull
request in a repo, or draft your weekly update every Friday. It works on a schedule, in your Slack
channels, and keeps going after you close your laptop.

**It's for anyone at IBM, technical or not.** A few examples:

- **Admins and operations:** answer the same questions from a knowledge base, track requests, send reminders.
- **Analysts and researchers:** watch sources overnight and send a summary in the morning.
- **Managers:** keep track of workstreams and draft status updates.
- **Developers:** review pull requests, fix issues, and run long coding tasks.

**Your first agent is already there.** The first time you sign in, you meet **Dammy**, the personal
agent everyone gets. Tell it what you do, and it helps you find a first task worth handing over. See
the [new user tour](getting-started/tour.md#meet-dammy-your-first-agent).

## Is DAM for me?

- **Do I need to be technical?** No. You work with agents by telling them what you need, in plain words,
  in the chat or in Slack.
- **Does it cost anything?** No. On the IBM hosted deployment, DAM is free to use, and agents run on
  CurveBender, IBM-hosted open-source models, unless you connect a provider of your own.
- **How is it different from ChatGPT or Gemini?** A chat assistant answers when you ask. A DAM agent also
  works when you're not there: it runs on a schedule, works in your Slack channels, keeps its files
  between sessions, and remembers how you like things done.
- **What data can I use?** See the Terms of Use. You accept them when you first sign in, and you can
  read them again any time in **Settings** › **About DAM** › **Terms of Use**.

## Two ways to run DAM

<div class="grid cards" markdown>

- :material-domain: **IBMers: use the IBM hosted deployment**

    ---

    Sign in with your IBM login at [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"}.
    There's no cluster to manage, and it includes IBM internal model endpoints and integrations.
    These docs describe this deployment.
    Get help in [#dam-community](https://ibm.enterprise.slack.com/archives/C0B3F03NB24){:target="_blank"}, and check
    [ibm.biz/dam-status](https://ibm.biz/dam-status){:target="_blank"} if something isn't working.

- :material-source-branch: **Open source: deploy DAM yourself**

    ---

    Run DAM on your own Kubernetes cluster. Start with the
    [deployment guide](https://github.com/dam-agents/dam#run-your-own){:target="_blank"} in the
    [dam-agents/dam](https://github.com/dam-agents/dam){:target="_blank"} repository.

</div>

!!! note "Running your own deployment?"
    The building blocks, features, and CLI work the same way. A few things on these pages are specific
    to the IBM hosted deployment and won't match yours: the sign-in, the VPN note, the IBM ETE LiteLLM
    and CurveBender providers, the server URL, and the IBM Slack workspace. Where a page covers one of
    these, it says so.

## How it works

**DAM is an open platform for running long-lived AI agents, built at IBM Research.** Each agent runs in
its own microVM with a persistent workspace, injected credentials and its own network boundary, so it
can work against real systems safely. Every agent is built from the same shared building blocks:
connections, skills, schedules, channels, artifacts and knowledge bases. A practice proven in one place
can be picked up everywhere, so good work compounds instead of being rebuilt by each team. See
**[The building blocks](core-concepts/building-blocks.md)** to learn more.

## Start building

Pick your entry point.

<div class="grid cards" markdown>

- :material-map-outline: **Take the tour**

    ---

    New to DAM? Meet Dammy, then see what agents can do for you, one piece at a time.

    [Start the tour →](getting-started/tour.md)

- :material-rocket-launch-outline: **Quickstart**

    ---

    Want to jump straight in? Create your own agent in minutes, in the web UI or the CLI.

    [Run the Quickstart →](getting-started/quickstart.md)

</div>
