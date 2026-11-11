<div class="welcome-hero" markdown>

<p class="welcome-eyebrow">IBM Research</p>

# Welcome to DAM

<p class="welcome-lead">Run long-lived AI agents in isolated pods, with runtime credential injection, scheduled triggers, and Slack-native access.</p>

<div class="welcome-cta" markdown>
[Launch DAM →](https://ibm.biz/dam-agents){:target="_blank" .md-button .md-button--primary}
</div>

</div>

!!! tip "DAM in 3 minutes"
    Follow one agent from start to finish: pick a harness and model, connect your tools, bind it
    to Slack, put it on a schedule, and gate sensitive actions behind human approval.

    [:material-play-circle-outline: Watch the demo](https://ibm.biz/dam-demo){:target="_blank"}

## What is DAM?

**DAM is IBM Research's platform for running long-lived AI agents.** An agent runs in its own isolated pod with a persistent workspace, injected credentials, and its own network boundary — so it can work against real systems, on a schedule, and in Slack, and keep going after you close your laptop. Each agent is built from the same shared building blocks: connections, skills, schedules, channels, artifacts, and knowledge bases. See **[The building blocks](core-concepts/building-blocks.md)** to learn more.

## Two ways to run DAM

<div class="grid cards" markdown>

- :material-domain: **IBMers: use the IBM hosted deployment**

    ---

    Sign in with your IBM login at [ibm.biz/dam-agents](https://ibm.biz/dam-agents){:target="_blank"}.
    There's no cluster to manage, and it includes IBM internal model endpoints and integrations.
    These docs describe this deployment.

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

## What problem does it solve, and for whom?

It's for **IBM Research engineers and researchers** who run agent workloads — and the **teams and leaders** who want the good ones to spread. Across IBM, people have built genuinely useful agents and workflows, but most live on one laptop and each new team rebuilds the same infrastructure — running agents, managing credentials, scheduling, storing results — before it can even start. So the work rarely compounds. DAM turns that infrastructure into shared building blocks no one has to rebuild: a practice proven in one place can be picked up everywhere, and new ones — from any team, or the wider industry — get captured once and made available to all.

## Start building

Pick your entry point.

<div class="grid cards" markdown>

- :material-rocket-launch-outline: **Quickstart**

    ---

    Launch your first Claude Code agent in minutes — in the web UI or the CLI.

    [Run the Quickstart →](getting-started/quickstart.md)

- :material-map-outline: **Take the tour**

    ---

    A guided walkthrough that builds one agent up capability by capability, so you see the whole platform first.

    [Start the tour →](getting-started/tour.md)

</div>
