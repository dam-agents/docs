# Run experiments

Some of the most powerful things you can build on DAM are systems that improve themselves — an agent that proposes a change, measures whether it helped, keeps what worked, and repeats. That loop is what an [experiment](../core-concepts/experiments.md) is: a design → build → test → learn cycle your agent writes as ordinary Python, and that the platform watches live. It's how you'd run an optimization framework like **NOUS** against a target of your own, or evolve a solution across hundreds of scored candidates.

The fastest way to understand one is to run a small one end to end, which is what this tutorial does.

This tutorial uses a deliberately tiny target: **[tiny-cache][tiny-cache]**, a dependency-free
in-memory key/value cache for Node.js. It's a good first experiment because it has the two things an
optimization loop needs and most real code lacks — a **test suite** that says whether a candidate is
correct, and a **deterministic benchmark** that says how fast it is.

[tiny-cache]: https://github.com/Tomas2D/tiny-cache

!!! info "tiny-cache is an ordinary public repo"
    It isn't bundled with DAM — your agent clones it from GitHub like any other repository. Nothing
    here is special-cased by the platform.

**What you'll end up with:** a loop that rewrites the cache implementation each round, runs the tests,
measures lookup latency across fixed seeds, and scores each candidate — with the whole thing charted
live in your chat.

## What you need

- An **experiment agent** — from **Experiments**, click **Create experiment** (see
  [creating an agent](../core-concepts/agents.md#create)). It's Claude Code with the experiment authoring
  skill already installed.
- A **provider connection** that reaches Claude — IBM LiteLLM ETE Proxy or Anthropic.
- **Trusted defaults** for network access, so the agent can reach GitHub.

!!! warning "Spawned workers need connections passed explicitly"
    Each round runs in a fresh, throwaway agent, and a spawned worker starts with **no credentials at
    all**. A `claude-code` worker with no provider connection fails its first model call and hangs
    until its liveness deadline. The authoring skill knows to pass them — if you write the script
    yourself, don't skip it.

## Step 1 — Get the code into the workspace

Ask your agent, in chat:

```
Clone https://github.com/Tomas2D/tiny-cache into the workspace,
then run the tests and the benchmark so we know the baseline.
```

It should get a passing suite and one line of JSON per benchmark run:

```json
{"scenario":"read-heavy","n":20000,"ops":5000,"seed":1,
 "p50_us":0.42,"p95_us":0.87,"mean_us":0.51,"ops_per_sec":1960784}
```

Those latency figures are your baseline. Write them down — the loop needs something to beat.

!!! tip "Run the baseline before writing any loop"
    A loop that can't beat the starting point is usually a scoring bug, not a search failure. Knowing
    the baseline first is what lets you tell those apart.

## Step 2 — Decide what "better" means

This is the only genuinely important design decision, and it's yours rather than the platform's. DAM
records and charts whatever number you report, but it never interprets it.

For this experiment:

| Question | Answer for tiny-cache |
|----------|----------------------|
| What varies each round? | The cache implementation (`src/tiny-cache.js`) |
| What makes a candidate valid? | The existing test suite passes, unchanged |
| What's the score? | Lookup latency on the benchmark — **lower is better** |
| How do we stop it gaming the score? | Fixed seeds, and tests must pass to score at all |

Two rules make this honest, and both are worth copying to your own experiments:

- **A broken candidate scores nothing.** If the tests fail, the round gets no score — it doesn't get a
  bad one. A cache that returns wrong answers instantly is not a fast cache.
- **The seeds are fixed.** `bench.mjs` is deterministic per seed, so every candidate is measured on
  the same workload. Averaging a few seeds (say `1`, `2`, `3`) stops a candidate winning by luck on
  one.

!!! note "Scores are higher-is-better, so invert latency"
    The platform charts scores as *higher is better*. Latency is the opposite, so report a score that
    rises as latency falls — negated latency (`-p50_us`) or a throughput figure both work. Pick one and
    stay with it, or your chart will read backwards.

## Step 3 — Have the agent write the loop

Describe the loop rather than dictating code — the skill turns this into a script:

```
Set up an experiment on the cloned tiny-cache repo.

Each round: rewrite src/tiny-cache.js to make lookups faster, run
`npm test` to check correctness, then `node bench/bench.mjs` on seeds
1, 2 and 3 with --scenario read-heavy.

Score = negated mean p50_us across the three seeds, so higher is
better. A candidate whose tests fail gets no score at all.
Keep the public API and the TTL, capacity and sorted-keys behavior
intact. Run 10 rounds.
```

The agent will agree the loop's stages with you before writing anything. Expect roughly:

| Stage | What happens |
|-------|--------------|
| **produce** | A fresh worker rewrites the implementation |
| **test** | The suite runs; a failure ends the round unscored |
| **bench** | Three fixed seeds run; the scores are averaged |
| **select** | The best candidate so far is kept |

Then it registers a **plan** — a reviewable draft plus a **Start a new run** button. Read the script
before you start it. It's ordinary Python, and this is the moment to catch a scoring mistake.

## Step 4 — Run it and watch

Click **Start a new run**. The dashboard docks beside your chat and fills in as the loop reports:

- The **graph** of stages, as each round moves through them
- **Per-stage progress**, including which rounds failed their tests
- The **score chart** — the line you actually care about

A running experiment holds its agent awake, so the idle timeout won't hibernate a loop mid-flight.
Ten rounds on a target this small is minutes, not hours.

!!! tip "Read the failures, not just the winner"
    The rounds that failed their tests are the interesting ones. They show you which optimizations
    break correctness — which is usually more transferable than the winning diff.

## Step 5 — Keep the result

When the run ends, DAM publishes a self-contained **results** artifact with the data baked in. It
renders without a live connection, so it's the thing to
[share by link](../core-concepts/artifacts.md#sharing) with people who have no DAM account.

Compare the winning candidate against the baseline from Step 1. On a target this small the honest
outcome is often *a few percent, or nothing* — and that's a useful result too. A loop that reliably
finds nothing on a well-optimized target is working correctly.

## What to take to a real experiment

The tiny-cache setup is a template for anything you'd actually optimize:

- **Correctness gate first, score second.** Find the check that says *this candidate is valid at all*
  before you pick a metric. Without it, a loop optimizes its way into nonsense.
- **Make measurement deterministic.** Fixed seeds, fixed inputs, several repeats. A noisy metric makes
  the search chase measurement error.
- **Score one number.** If you need two, decide their trade-off in the script rather than reading two
  charts and squinting.
- **Start with fewer rounds than you want.** Ten rounds tells you whether the loop is wired up
  correctly. Scale up once the chart moves for the right reasons.

## What's next

- [Experiments](../core-concepts/experiments.md) — the model, lifecycle, and dashboards in full
- [Optimization framework images](../core-concepts/experiments.md#optimization-framework-images) —
  NOUS, GEPA, OpenEvolve and others, for when you'd rather not write the loop yourself
- [Artifacts](../core-concepts/artifacts.md) — publishing and sharing what a run produced
