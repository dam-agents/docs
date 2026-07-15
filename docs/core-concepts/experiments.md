# Experiments

An **experiment** races several agents — its **arms** — toward one shared **goal**. Each arm is an existing agent you own; the platform records each arm's results as numbered **runs** so you can compare how the arms did against the same objective.

Experiments are owner-scoped: you only ever see and act on your own experiments.

## Model

| Concept | What it is |
|---------|-----------|
| **Experiment** | A named contest with a prose `goal` and an opaque `spec` (shared task/budget config). Has a `status` (see [Lifecycle](#lifecycle)). |
| **Arm** | One competitor: an agent you own, plus an opaque `armSpec`. An agent can be an arm of an experiment only once; the same experiment cannot list the same agent twice. |
| **Run** | One ledger entry produced by an arm. Carries a per-arm `runNumber`, a `score`, a `candidateRef` pointing at a stored artifact, and the session it came from. |

`spec` and `armSpec` are free-form JSON the platform stores but does not interpret — they are handed to the agent when the experiment starts. The platform does **not** rank, normalize, or interpret a run's `score`; scoring semantics are yours to define.

## Lifecycle

An experiment's `status` is one of:

| Status | Meaning |
|--------|---------|
| `draft` | Default at creation. Arms can be added; nothing is running. |
| `running` | The contest is active. |
| `completed` | Finished. |
| `stopped` | Halted. |

A newly created experiment starts in `draft`. (Starting, stopping, and the run-recording path arrive in later releases.)

## Managing experiments

Experiments are managed through the API. Listing and reading an experiment require read access to agents; creating one, adding arms, and deleting require manage-agents permission.

| Operation | Effect |
|-----------|--------|
| `experiments.list` | All your experiments, newest first. |
| `experiments.get` | One experiment with its arms, and each arm's runs nested under it. |
| `experiments.create` | Create a `draft` experiment from a `name`, `goal`, and optional `spec`. Names must be unique per user. |
| `experiments.addArm` | Add an agent you own as an arm, with an optional `armSpec`. Adding an agent that isn't yours, or one already on the experiment, is rejected. |
| `experiments.delete` | Delete the experiment and all its arms and runs. |

### Constraints

- **Names are unique per user.** Creating a second experiment with a name you already used is rejected.
- **Arms must reference an agent you own.** Adding an arm for an unknown or unowned agent is rejected.
- **An agent is an arm at most once per experiment.** Re-adding the same agent is rejected.
- **Delete is a full cascade.** Deleting an experiment removes its arms and runs too.
