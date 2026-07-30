# Experiments

An **experiment** is a design→build→test→learn loop that your agent writes as an ordinary Python
script — and that the platform watches live while it runs.

The platform deliberately does **not** run the loop or interpret its scores. The loop's shape lives in
your script; the script reports what it's doing, and the platform gives you the live view, the history,
and the dashboard.

## Model

| Concept | What it is |
|---------|-----------|
| **Driver** | The sandbox that authors and runs the loop script. |
| **Skeleton** | The stages and loops your script declares up front — a statement of intent. |
| **Span** | One tagged step the script reports as it runs, carrying a status, an optional numeric score, and any artifacts it produced. |
| **Run** | One execution of the script. Each run captures the script as it was at that moment. |

Scores are opaque: the platform records and plots them, but never normalizes or ranks them. What a
score *means* is yours to define.

!!! note "The skeleton is lenient, not a straitjacket"
    If your script reports a stage it never declared, the graph grows and the step is flagged as
    **drift** — it's never an error. A long run should not fail hours in over a declaration mismatch.

## Getting started

The quickest path is an **experiment sandbox** — pick it as your starting point when you
[create a sandbox](agents.md#create). It's a Claude Code sandbox with the experiment authoring skill
already installed, so you can just describe the loop you want and let the agent write it.

!!! info "Needs an image your operator enables"
    The experiment starting point only appears if your operator has enabled its image. The wizard
    tells you if it's missing.

Any sandbox can drive an experiment, though — the SDK (`experiment_sdk`, Python, standard library
only) ships in the platform base image. Declaring a skeleton takes a handful of lines around code you
would write anyway.

### Framework starting points

If you'd rather begin from an established optimization framework than write the loop yourself, the
wizard also offers preconfigured starting points for **NOUS**, **OpenEvolve**, **ShinkaEvolve**, and
**GEPA**. Each is marked experimental and, like the experiment sandbox, depends on an image your
operator has enabled.

## Lifecycle

1. **Register a plan.** Running the script in plan mode executes its declarations only. That creates a
   **draft** and publishes the script — plus a dashboard — into your [artifact library](artifacts.md).
   Re-registering the same experiment refreshes the draft in place.
2. **Start a run.** The draft is the reusable blueprint; starting a run clones the script and launches
   it in the background on the driver sandbox.
3. **Watch it.** While a run is active you get a live feed of spans as they're reported.
4. **It finishes** as either completed or failed — or you **stop** it.

A running experiment **pins its sandbox awake**, so the idle timeout won't hibernate a loop mid-flight
(a hard stop still wins). The SDK sends a heartbeat while your script lives, so a loop that's quiet —
blocked on a spawn, or deep in a computation — stays healthy. If the heartbeat stops (the script
crashed, the pod was lost), the run is marked failed and the sandbox is released.

A script that's wedged but still alive keeps heartbeating and stays visibly running — the platform
can't tell stuck from slow, which is what **Stop** is for.

## Dashboards

Every experiment gets a dashboard published to your [artifact library](artifacts.md). Your script can
supply its own bespoke HTML instead of the stock one. Dashboards render sealed off from the network
and from credentials — they visualize the data the run reported, nothing more.

Because it's an artifact, a dashboard can be [shared with a public link](artifacts.md#sharing) — which
is what makes an experiment's results shareable outside DAM.

## Managing experiments

Experiments have their own destination in the DAM UI, listing yours newest-first. Open one for its
skeleton, its runs, and the live trace of the current run. Deleting an experiment removes its runs with
it.

Experiments are owner-scoped: you only see and act on your own.
