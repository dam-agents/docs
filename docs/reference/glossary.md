# Glossary

Key terms used throughout DAM.

## Agents & Instances

| Term | Definition |
|------|-----------|
| **Template** | A read-only blueprint that defines which AI harness powers an agent |
| **Agent** | A user-owned definition created from a template |
| **Instance** | The running deployment of an agent — its pod, workspace, and configuration |
| **Workspace** | The persistent filesystem (`/home/agent/work`) attached to an agent |

## Sessions

| Term | Definition |
|------|-----------|
| **Session** | A single conversation thread with an agent. Persists across restarts. |
| **Chat session** | Interactive conversation from the UI or Slack |
| **Terminal session** | PTY-based shell access to the agent pod |
| **Schedule session** | A session created by a scheduled trigger firing |

## Lifecycle

| Term | Definition |
|------|-----------|
| **Running** | Agent pod is active and accepting messages |
| **Hibernated** | Agent pod scaled to zero. Workspace preserved, no compute used. |
| **Wake** | Transition from hibernated to running (typically under a minute) |
| **Hibernate** | Transition from running to hibernated after idle timeout |

## Credentials & Security

| Term | Definition |
|------|-----------|
| **Connection** | A linked credential (OAuth token or API key) stored securely by the platform |
| **Gateway** | The paired Envoy proxy pod that injects credentials on the network wire |
| **Egress rule** | A network allowlist entry defining which external hosts an agent can reach |
| **Approval** | A pending human decision about whether to allow an agent's network request |
| **Verdict** | The user's decision on an approval: allow once, allow always, or deny |

## Collaboration

| Term | Definition |
|------|-----------|
| **Channel** | An external communication pathway (Slack channel) connected to an agent |
| **Foreign replier** | A team member (not the agent owner) who interacts via Slack |
| **Fork** | A short-lived pod spawned for a foreign replier's turn, using their credentials |
| **Allowed users** | The list of team members permitted to interact with an agent via Slack |

## Skills

| Term | Definition |
|------|-----------|
| **Skill** | A packaged capability installable from a git repo |
| **Skill source** | A connected git repo from which skills can be installed |

## Scheduling

| Term | Definition |
|------|-----------|
| **Schedule** | A recurring trigger that wakes an agent and runs a task on a timer |
| **Quiet hours** | Time windows during which scheduled fires are suppressed |
| **Fresh mode** | Each schedule fire creates a new independent session |
| **Continuous mode** | All fires share one session (agent remembers prior runs) |
