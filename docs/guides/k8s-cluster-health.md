# Kubernetes Cluster Health Monitor

Keep an eye on your cluster without watching it yourself.

This guide walks through setting up a scheduled agent that checks the health of a Kubernetes or OpenShift cluster on a recurring basis and publishes an HTML health report as an artifact — updated in place every run, so the same link always shows the latest state.

!!! note "Single cluster"
    This guide monitors one cluster.
    The same pattern extends to multiple clusters: add a Kubernetes connection per cluster and either run one agent per cluster or extend the prompt to loop over each connection in turn.
    See [Extending to multiple clusters](#extending-to-multiple-clusters) at the end of this page.

## What It Does

Once configured, the agent runs on a schedule and:

1. **Checks node status** — lists all nodes and flags any that are `NotReady` or under memory, disk, or PID pressure
2. **Checks pod health** — finds pods in `CrashLoopBackOff`, `Error`, or long-running `Pending` state, and any with high restart counts
3. **Checks cluster operators** (OpenShift) — lists `ClusterOperator` resources and flags any that are degraded or unavailable
4. **Summarises warning events** — surfaces the most frequent warning reasons from recent events
5. **Publishes an HTML artifact** — a colour-coded health report (🟢 / 🟡 / 🔴) updated in place each run

```
Scheduled Run
     │
     ├── kubectl cluster-info
     ├── kubectl get nodes -o json
     ├── kubectl get pods --all-namespaces
     ├── kubectl get clusteroperators             (OpenShift only)
     ├── kubectl get events --all-namespaces --field-selector type=Warning
     │
     └── Publish HTML artifact "Cluster Health Report"
              └── update in place if it already exists
```

## Requirements

### Provider

Any provider already configured on your account works.
Add one under **Settings → Providers** if you have not done so yet.

### Connection

- **Kubernetes / OpenShift** — the agent uses `kubectl` with credentials injected at the network layer; it never holds your token. See the [RBAC setup](#rbac-setup) section below to create the service account and the token.

### Network access

**Trusted defaults** is sufficient.
When you grant the Kubernetes connection to the agent, the cluster's API server URL is automatically added to the agent's network access allowlist — no manual rule needed.

## RBAC Setup

Create a dedicated service account in your cluster with read-only access. This will be used by the agent to gather data about the connected **Kubernetes / OpenShift** cluster. Save the manifest below as `dam-agent-rbac.yaml` and apply it:

```yaml
# Namespace for the service account
apiVersion: v1
kind: Namespace
metadata:
  name: health-agent
---
# Service account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dam-agent
  namespace: health-agent
---
# Long-lived token (Kubernetes >= 1.24)
apiVersion: v1
kind: Secret
metadata:
  name: dam-agent-token
  namespace: health-agent
  annotations:
    kubernetes.io/service-account.name: dam-agent
type: kubernetes.io/service-account-token
---
# Cluster-wide read-only role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dam-agent-reader
rules:
  - apiGroups: [""]
    resources: ["nodes", "pods", "namespaces", "events", "services"]
    verbs: ["get", "list"]
  - apiGroups: ["events.k8s.io"]
    resources: ["events"]
    verbs: ["get", "list"]
  - apiGroups: ["config.openshift.io"]
    resources: ["clusteroperators"]
    verbs: ["get", "list"]
---
# Bind the role to the service account
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dam-agent-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dam-agent-reader
subjects:
  - kind: ServiceAccount
    name: dam-agent
    namespace: health-agent
```

```bash
kubectl apply -f dam-agent-rbac.yaml
```

!!! note "Verify the secret was created"
    Run `kubectl get secret dam-agent-token -n health-agent` and confirm it shows a non-empty `token` data key before continuing.

!!! note "OpenShift only"
    The `config.openshift.io/clusteroperators` rule is harmless on plain Kubernetes — no such API group exists there, so it grants nothing. Remove it if you want a strictly minimal role on a non-OpenShift cluster.

Extract the values you need for the connection form:

```bash
# API server URL — paste into "API server URL" (must be a DNS hostname, not an IP address)
kubectl cluster-info | grep 'control plane'

# Service account token — paste into "Secret value"
kubectl get secret dam-agent-token -n health-agent \
  -o jsonpath='{.data.token}' | base64 -d
```

!!! note "API server URL format and private CA"
    The **API server URL** must use a DNS hostname rather than an IP address because the gateway routes by TLS SNI. If your cluster uses a private CA, it is also caught automatically at save time with a prompt to paste the CA certificate into **Cluster CA certificate**.

## Setup

1. **Create an agent from scratch** — on **Home**, click **Create agent** (no need to pick any of the starter kits), and name it (e.g. `Cluster Health Monitor`).

2. **Configure image and provider** — select the image that matches your configured provider.

3. **Add and grant the Kubernetes connection** — in the **Connections** section on the create page, click **Add connection** (or select **New connection** from the catalogue), open the **Apps** tab, and select **Kubernetes / OpenShift**. Fill in:
   - **Name** — a name to identify the connection
   - **API server URL** — the output from `kubectl cluster-info` above (must be a DNS hostname)
   - **Secret value** — the base64-decoded token from above
   - **Cluster CA certificate** — optional; only needed if your API server uses a private CA (prompted automatically at save time if required)

   Click **Save** and select the connection so it is granted at agent creation.

4. **Configure lifecycle** — select **Hibernate when idle** and set a short idle timeout (e.g. 5 minutes) so the agent frees compute between runs.

5. Click **Create agent** and wait for the creation to complete.

6. **Create the schedule** — click **⋮ → Configure agent**, open the **Schedules** panel, and click **Create Schedule**.
   Set:
   - **Name**: `cluster-health`
   - **Repeat**: **Every N minutes**, with **Every** = `15` (or `5` for a faster demo cadence)
   - **Prompt**: paste the task prompt from [The Artifact](#the-artifact) section below

7. **Fire the first run immediately** — click **⋮ → Run now** on the schedule card. You do not need to wait for the next scheduled occurrence.

## The Artifact

Paste this prompt into the schedule's **Prompt** field.

!!! tip "Reuse this prompt across agents with a Skill"
    If you want to run the same health check on multiple clusters, turn the prompt into an agent [Skill](../core-concepts/skills.md).
    Skills are per agent, so editing a skill in one agent does not change any other agent. To share one prompt across multiple agents, publish it to a skill source repository, install it from that source in each agent, and replace the schedule prompt with a single line: `Run the Cluster Health Monitor skill.`
    When you update the skill in the source repository, click **Re-scan** or update in each agent to pick up the change.

```
Check the health of the Kubernetes cluster using the connected Kubernetes
credential. Run exactly the commands below — do not substitute alternatives.

1. Cluster info:
   kubectl cluster-info

2. Nodes:
   kubectl get nodes -o json

3. Pods (all namespaces):
   kubectl get pods --all-namespaces

4. Cluster operators (OpenShift only — skip if the command fails):
   kubectl get clusteroperators

5. Warning events:
   kubectl get events --all-namespaces --field-selector type=Warning

From the output, determine:
- NODES_FLAGGED: nodes where Ready is not True, or MemoryPressure /
  DiskPressure / PIDPressure is True, or lastTransitionTime on the Ready
  condition is within the last 30 minutes.
- PODS_FLAGGED: pods in CrashLoopBackOff, Error, or Pending state, or with
  restart count above 5.
- OPERATORS_FLAGGED: cluster operators where DEGRADED=True or AVAILABLE=False
  (skip if step 4 failed).
- TOP_WARNINGS: top 3 warning reasons by frequency (empty list if none).

Overall status:
- RED   if any node is NotReady
- AMBER if any other flag is non-empty
- GREEN otherwise

Then publish an HTML artifact using EXACTLY this template — replace only
the placeholders in ALLCAPS, do not change any HTML, CSS, or structure:

<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8"><title>Cluster Health Report</title>
<style>
body{font-family:Arial,Helvetica,sans-serif;margin:0;background:#f4f4f4;color:#161616}
header{background:#262626;color:#fff;padding:20px 32px}
header h1{margin:0 0 6px;font-size:26px}
header p{margin:2px 0;color:#c6c6c6;font-size:14px}
main{padding:24px 32px}
.badge{display:inline-block;padding:6px 18px;border-radius:4px;font-weight:bold;font-size:18px;color:#161616;margin-top:10px}
.green{background:#24a148;color:#fff}
.amber{background:#f1c21b;color:#161616}
.red{background:#da1e28;color:#fff}
section{background:#fff;border:1px solid #e0e0e0;border-radius:4px;padding:16px 20px;margin-bottom:20px}
h2{margin:0 0 12px;font-size:18px}
table{border-collapse:collapse;width:100%;font-size:13px}
th,td{text-align:left;padding:6px 10px;border-bottom:1px solid #e0e0e0;vertical-align:top}
th{background:#f4f4f4}
.ok{color:#198038;font-weight:bold}
.warn{color:#b28600;font-weight:bold}
.err{color:#da1e28;font-weight:bold}
.note{color:#525252;font-size:13px}
</style></head><body>
<header>
<h1>Cluster Health Report</h1>
<p>Generated: DATE_UTC</p>
<p>API server: CLUSTER_URL</p>
<span class="badge STATUS_CLASS">STATUS_LABEL</span>
<p>STATUS_SUMMARY</p>
</header>
<main>
<section><h2>Nodes</h2>
<table><tr><th>Name</th><th>Status</th><th>Conditions</th></tr>
NODES_ROWS
</table>
<p class="note">NODES_NOTE</p></section>
<section><h2>Problem Pods</h2>
PODS_CONTENT
<p class="note">PODS_NOTE</p></section>
<section><h2>Cluster Operators</h2>
<table><tr><th>Name</th><th>Available</th><th>Progressing</th><th>Degraded</th><th>Message</th></tr>
OPERATORS_ROWS
</table>
<p class="note">OPERATORS_NOTE</p></section>
<section><h2>Recent Warnings</h2>
WARNINGS_CONTENT
<p class="note">WARNINGS_NOTE</p></section>
</main></body></html>

Placeholder rules:
- DATE_UTC: current date and time in UTC, e.g. "2026-10-28 19:15 UTC"
- CLUSTER_URL: API server URL from step 1
- STATUS_CLASS / STATUS_LABEL: use "green"/"GREEN", "amber"/"AMBER", or
  "red"/"RED" based on the overall status above
- STATUS_SUMMARY: one-sentence plain-English summary, e.g.
  "No nodes NotReady; 5 warning events present."
- NODES_ROWS: one <tr> per node; set class="ok", "warn", or "err" on the
  status <td> as appropriate
- NODES_NOTE: summary sentence, e.g.
  "No nodes NotReady, under pressure, or with a recent Ready transition (last 30 min)."
- PODS_CONTENT: if PODS_FLAGGED is empty use <p>None</p>
  otherwise a table with columns Name, Namespace, State, Restarts
- PODS_NOTE: summary sentence, e.g.
  "92 pods checked: 89 Running, 3 Completed; max restart count 3."
- OPERATORS_ROWS / OPERATORS_NOTE: if step 4 failed, replace the entire
  Cluster Operators section (from <section> to </section>) with nothing.
  Otherwise, emit one <tr> per operator — all of them, not just flagged ones,
  matching the output of `kubectl get co`.
  Set class="err" on the Available <td> if Available=False, class="err" on
  the Degraded <td> if Degraded=True, class="ok" otherwise.
  OPERATORS_NOTE: summary sentence, e.g.
  "All 21 operators Available=True, Degraded=False."
  or "2 of 21 operators degraded." if any are flagged.
- WARNINGS_CONTENT: if TOP_WARNINGS is empty use <p>No warning events.</p>
  otherwise a table with columns #, Reason, Count, Example
- WARNINGS_NOTE: summary sentence, e.g.
  "Only 2 distinct warning reasons were found."
  or "No warnings in the last hour." if TOP_WARNINGS is empty

If an artifact named "Cluster Health Report" already exists, update it in
place (same artifact, new version). Leave it private.
```

After the first run, open the **Artifacts** tab in the agent's side panel to find the report.
Click it to preview the HTML page inline.

!!! tip "Making the report shareable"
    The artifact is private by default — only you can see it in the app.
    To share it, click the **Private** badge on the artifact row and choose **Public**.
    A link appears that anyone can open in a browser, no sign-in required.
    Treat a public link as public: anyone with it can view the report.

Because the agent updates the same artifact on every run, the share link stays stable — you can bookmark it or paste it into a dashboard once and it will always show the latest check.

## Schedule Cadence

| Scenario | Repeat setting | Details |
|----------|----------------|---------|
| First demo / quick test | Every N minutes | Every = 5 |
| Active cluster monitoring | Every N minutes | Every = 15 |
| Low-traffic or dev cluster | Daily | Chosen Time + Timezone (e.g. 08:00 UTC) |

Set the cadence in the schedule's **Edit schedule** form (options include **Daily**, **Hourly**, **Every N minutes**, and **Custom (RRULE)**).

!!! tip "Set a short hibernate timeout to save cost"
    The default **Hibernate when idle** timeout is 60 minutes, so on a 15-minute cadence the agent never stays idle long enough to hibernate and holds compute continuously.
    Under **Configure agent → Lifecycle**, set a short **Hibernate when idle** timeout (e.g. 5 minutes) so the agent hibernates shortly after each run finishes.
    You can also stop the agent manually; a scheduled run will still start the agent, and it will hibernate after the idle timeout once the run completes.

## Viewing Past Reports

Each schedule run produces a new **version** of the artifact, not a new artifact.
To see the history, open the artifact and click the version list in the top bar — each entry shows when it was published and how large it was.

The sessions that produced each version can be accessed via the **View results** button on the schedule card (the **⋮** menu holds **Run now**, **Edit schedule**, **Reset session**, and **Delete schedule**).
Open any session to see the raw output from that run.

## Security

- **Read-only** — the `ClusterRole` grants only `get` and `list`. It cannot create, modify, or delete any cluster resource.
- **Token never reaches the agent** — credentials are injected at the network layer by the platform gateway. Even if the agent were compromised, it could not extract the token.
- **Scoped service account** — the `dam-agent` service account lives in its own `health-agent` namespace and holds no other permissions. Deleting the namespace removes all access at once.
- **No CA certificate needed** — this guide assumes your API server's certificate is publicly trusted. If your cluster uses a private CA, add the CA certificate (PEM) to the connection form's **Cluster CA certificate** field.

## Extending to Multiple Clusters

This guide is deliberately scoped to one cluster to keep the setup straightforward.
Once you have it working, monitoring more clusters is a small step:

**Option A — one agent per cluster.**
Create a second agent the same way, grant it the second cluster's connection, and add the same schedule and prompt.
Each agent publishes its own artifact.
This is the simplest approach and keeps failures isolated.

**Option B — one agent, multiple connections.**
Grant all cluster connections to a single agent and update the prompt to iterate: *"For each connected Kubernetes cluster, run the health check and publish (or update) an artifact named 'Cluster Health Report — \<cluster name\>'"*.
The agent runs the same commands once per connection and produces one artifact per cluster. Note that the prompt and HTML template will need further changes to support this — either by parameterizing the cluster name in the artifact name, title, and header, or by redesigning the HTML template into a unified multi-cluster dashboard.

With **Option A**, maintain a single HTML template across your agents by storing it as a [Skill](../core-concepts/skills.md) in a skill source repository. With **Option B**, the prompt and template are already centralized within the single agent.
