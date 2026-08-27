# Experiments

An **experiment** is a design→build→test→learn loop that your agent writes as an ordinary Python
script — and that the platform watches live while it runs.

The platform deliberately does **not** run the loop or interpret its scores. The loop's shape lives in
your script; the script reports what it's doing, and the platform gives you the live view, the history,
and the dashboard.

## Model

| Concept | What it is |
|---------|-----------|
| **Driver** | The agent that authors and runs the loop script. |
| **Skeleton** | The stages and loops your script declares up front — a statement of intent. |
| **Span** | One tagged step the script reports as it runs, carrying a status, an optional numeric score, and any artifacts it produced. |
| **Run** | One execution of the script. Each run captures the script as it was at that moment. |

Scores are opaque: the platform records and plots them, but never normalizes or ranks them. What a
score *means* is yours to define.

!!! note "The skeleton is lenient, not a straitjacket"
    If your script reports a stage it never declared, the graph grows and the step is flagged as
    **drift** — it's never an error. A long run should not fail hours in over a declaration mismatch.

## Getting started

The quickest path is an **experiment agent** — its own choice when you
[create an agent](agents.md#create). It's a Claude Code agent with the experiment authoring skill
already installed, so you can just describe the loop you want and let the agent write it.

Any agent can drive an experiment, though — the SDK (`experiment_sdk`, Python, standard library
only) ships in the platform base image. Declaring a skeleton takes a handful of lines around code you
would write anyway.

### Optimization frameworks

If you'd rather work in an established optimization framework than write the loop yourself, ask the
agent to run one — point it at a target and a measurable objective in chat and it drives the framework
for you. Available frameworks (all badged **Alpha**) include **NOUS**, **OpenEvolve**, **ShinkaEvolve**,
**GEPA**, **k-search**, and **SkyDiscover** (which carries the **AdaEvolve** and **EvoX** search strategies).

These run their own campaigns in their own terms — they don't register a plan or appear under
Experiments.

## Lifecycle

1. **Register a plan.** Running the script in plan mode executes its declarations only. That creates a
   **draft** and publishes the script — plus a dashboard — into your [artifact library](artifacts.md).
   Re-registering the same experiment refreshes the draft in place.
2. **Start a run.** The draft is the reusable blueprint; starting a run clones the script and launches
   it in the background on the driver agent.
3. **Watch it.** While a run is active you get a live feed of spans as they're reported.
4. **It finishes** as either completed or failed — or you **stop** it.

A running experiment **pins its agent awake**, so the idle timeout won't hibernate a loop mid-flight
(a hard stop still wins). The SDK sends a heartbeat while your script lives, so a loop that's quiet —
blocked on a spawn, or deep in a computation — stays healthy. If the heartbeat stops (the script
crashed, the pod was lost), the run is marked failed and the agent is released.

A script that's wedged but still alive keeps heartbeating and stays visibly running — the platform
can't tell stuck from slow, which is what **Stop** is for. Stop also ends the run's in-flight
sub-agents, which is what frees a loop parked waiting on one.

## Dashboards

Every experiment gets a dashboard published to your [artifact library](artifacts.md). Your script can
supply its own bespoke HTML instead of the stock one. Dashboards render sealed off from the network
and from credentials — they visualize the data the run reported, nothing more.

Everything DAM publishes for one experiment is filed together, in its own **Experiments** folder named
after the experiment — the draft script, the dashboard, and each run's script copy and results page.
The library lists those apart, in a collapsed **Experiments** section below your own artifacts.

The live dashboard only works inside DAM, where it can receive the run's data. When a run finishes,
the platform publishes a self-contained **results** artifact — the same view with the finished run's
data baked in. That's the one to
[share with a public link](artifacts.md#sharing) outside DAM.

## Managing experiments

Experiments have their own destination in the DAM UI, grouped under the agent running them — one
agent can hold several, so the agent is the container. From there you reach an experiment's runs,
the subagents a run currently has working, and its per-run artifacts, and open the agent's chat
where the dashboard docks.
Deleting an experiment removes its runs with it; the artifacts it published survive. Stop a running
experiment before deleting it — the platform refuses while a run is live. Deleting the *agent*
doesn't delete its experiments: the runs stay listed, and their published results stay in your
library.

Experiments are owner-scoped: you only see and act on your own.
