# Building a Custom Image

A custom image is a layer you build on top of DAM's base platform image, giving your agents access to your own tools, scripts, and environment. You build it, push it to a container registry, and point an agent at it by pasting the image URL into the **Custom Image** field of the Add Agent dialog.

This guide is **agent-neutral**: the platform supports any agent — Claude Code, Codex, Gemini, or your own command-line program — and relies only on a small contract (covered in Step 3). The examples below use **Claude Code**.

The base image handles the platform plumbing:

- It runs the **agent-runtime** process — an ACP WebSocket server, file service, and trigger watcher — that connects your agent to the platform. You never modify it.
- It controls *when* your agent is launched (a chat session vs. a terminal session). You define *what* gets launched.
- It includes common tools: `git`, `gh`, `ssh`, and `npm`.

Customizing the image is mostly a few lines in the Dockerfile. Options, used alone or in combination:

- **Add instructions or a personality** — seed an instructions file describing who the agent is, what it's for, and how to behave. Claude Code, for example, reads `CLAUDE.md`.
- **Add files to work from** — include documents, a dataset, reference material, or starter code in the agent's home directory.
- **Add a tool or library** — install an npm package, a Python package, or a command-line program.
- **Reach an external service** — if your agent calls an outside API, the platform supplies the credentials securely. This is optional; see [its own section](#calling-an-external-service) at the end.

Each is shown in the steps below. The simplest useful custom image is the base plus an instructions file — no tools, no external calls.

## Step 1 — Set up a build directory

Create a working directory anywhere with these files:

```
my-agent/
  Dockerfile
  harness-chat.sh       # launches a chat session
  harness-terminal.sh   # launches a terminal session
  workspace/            # files to seed into the agent's home directory
                        #   (e.g. an instructions file, documents, a dataset)
```

Only the `Dockerfile` is required; add the rest as needed. For a complete, working starting point to copy and edit, see this [**example**](https://github.ibm.com/dam-agents/materials-project-agent-image){:target="_blank"} — it builds an agent end to end that calls the Materials Project API.

A bare-bones scaffold with nothing implemented is available [**here**](https://github.ibm.com/dam-agents/dam-image-scaffold){:target="_blank"}.

## Step 2 — Write the Dockerfile

Every agent Dockerfile follows the same shape: start `FROM` the published base image, make your changes as the `root` user, and switch back to the normal agent user at the end. The base is published at `quay.io/dam-agents/platform-base` — pin a specific version tag rather than relying on `latest`.

The base image provides:

- A Node runtime, plus `git`, `gh`, `ssh`, `npm`, `node`, `uv` (a Python installer), `curl`, and `jq`
- The **agent-runtime** process (ACP WebSocket server, file service, trigger watcher)
- Default harness scripts (placeholders you'll override in Step 3)
- A non-root user (UID 65532)

The base is a RHEL image: install system packages with **`dnf`** (there is no `apt`). It has **no preinstalled Python** — use the bundled `uv`.

The annotated skeleton follows. Most of it is fixed boilerplate; the parts you change are called out.

```dockerfile
FROM quay.io/dam-agents/platform-base:latest  # (1) always build on the base image

USER root                                     # (2) switch to root to make changes

# (3) CUSTOMIZE: install any tools or libraries your agent should have.
#     The base is RHEL with node/npm and uv (a Python installer); there is no
#     preinstalled Python. Install one of these ways:
#       - a system package: RUN dnf install -y <package>
#       - an npm package:   RUN npm install -g <package>
#       - a Python package: RUN uv venv --python 3.12 /opt/venv \
#                               && uv pip install --python /opt/venv/bin/python <package>
#       - a downloaded program: curl it down and chmod +x it
#     Leave this out entirely if your agent doesn't need extra tools.
RUN <install something here, or delete this block> \
    && chown -R 65532:0 /app                  # (4) hand ownership back to the agent

# (5) Install your agent itself. This is the one part that depends on WHICH
#     agent you run. The example below installs Claude Code — swap it for your
#     own agent's CLI + its chat (ACP) server. See Step 3 for the contract.
#     EXAMPLE — Claude Code: the `claude` CLI plus claude-agent-acp (the chat
#     server).
RUN --mount=type=cache,target=/root/.npm \
    npm install @agentclientprotocol/claude-agent-acp \
    && npm install -g @anthropic-ai/claude-code \
    && ln -sf "$(npm config get prefix)/bin/claude" /usr/local/bin/claude \
    && chown -R 65532:0 /app

# (6) Boilerplate: copy in the two session start scripts. These are the same
#     for any agent — only their CONTENTS differ (Step 3). Copy verbatim.
COPY harness-chat.sh /usr/local/bin/harness-chat
COPY harness-terminal.sh /usr/local/bin/harness-terminal
RUN chmod +x /usr/local/bin/harness-chat /usr/local/bin/harness-terminal

# (7) CUSTOMIZE: seed files into the agent's home directory. Anything you put
#     in your local workspace/ folder shows up where the agent starts. This is
#     how you give the agent instructions and files to work from, e.g.:
#       workspace/<instructions-file> -> instructions the agent reads on startup
#       workspace/data/prices.csv     -> a dataset it can open
COPY --chown=65532:0 workspace/ /app/working-dir/

# (8) OPTIONAL: declare placeholder env vars for any external credentials.
#     Real values are injected on the wire by the platform — never bake a real
#     secret into the image. See "Calling an external service" below.
ENV MY_API_KEY=dummy-placeholder

USER 65532                            # (9) ALWAYS switch back to the agent user
```

Rules enforced by the skeleton:

- **Always end as the agent user.** Make changes as `root`, but the last line must be `USER 65532`. After writing files as root, run `chown -R 65532:0 /app` so the agent can read them. (`65532` is the ID of the non-root user the agent runs as.)
- **The base is RHEL, with no Python out of the box.** Install system packages with `dnf` (there is no `apt`), npm packages with `npm`, Python packages with the bundled `uv`, or add a downloaded program. Already present: `git`, `gh`, `ssh`, `npm`, `node`, `uv`, `curl`, and `jq`.
- **`workspace/` ships files to the agent.** Its contents are copied in when the agent first starts. The copy happens *once*, without overwriting — it seeds initial files and does not push updates to an agent that has already run.

## Step 3 — Provide the session launchers

This is the entire contract between your image and the platform. When you open a session, the agent-runtime spawns one of two start scripts, and **you decide what each one launches**:

| Path | Mode | What it launches |
|------|------|------------------|
| `/usr/local/bin/harness-chat` | Chat | A program that speaks the [**Agent Client Protocol (ACP)**](https://agentclientprotocol.com/get-started/introduction) over stdio — the standard the chat window uses to exchange messages with an agent. By convention these programs are named `<agent>-acp`. |
| `/usr/local/bin/harness-terminal` | Terminal | Anything interactive attached to a PTY: your agent's command-line tool, or just a plain shell. |

Ship **both**. The base image includes placeholder versions for a different setup; override them with your own. (Every built-in agent — Claude Code, Codex, Gemini — ships these same two scripts; only the program each one execs differs.)

The pattern, for any agent, is:

```sh
# harness-chat.sh   ->  exec <your-agent>-acp "$@"
# harness-terminal.sh ->  exec <your-agent-cli> "$@"   (or: exec /bin/bash -l)
```

**Worked example — Claude Code.** With the Step 2 install in place, `claude` and `claude-agent-acp` are both on `PATH`:

`harness-chat.sh` — launch the chat (ACP) server:

```sh
#!/bin/sh
exec claude-agent-acp "$@"
```

`harness-terminal.sh` — open a plain login shell, so the user lands at a prompt and runs `claude` to start the agent:

```sh
#!/bin/sh
exec /bin/bash -l
```

> To drop the terminal *directly* into the agent instead of a shell, have `harness-terminal` exec the agent's CLI directly (e.g. `exec claude "$@"`).

### Session resumption (terminal mode)

In terminal mode, the platform passes a `HARNESS_SESSION_ID` environment variable. If your agent supports resuming a prior conversation, use it to reattach instead of starting fresh:

```sh
#!/bin/sh
SESSION_MARKER="$HOME/.my-harness/session-started"
mkdir -p "$HOME/.my-harness" 2>/dev/null

if [ -f "$SESSION_MARKER" ]; then
  exec my-harness --resume "$HARNESS_SESSION_ID" "$@"
else
  touch "$SESSION_MARKER"
  exec my-harness "$@"
fi
```

## Step 4 — Build and push to a registry

Build the image, tag it for your registry, and push. Use any registry the cluster can pull from (GHCR, Quay, Docker Hub, ECR, an internal registry, etc.):

```sh
# Build for the cluster's architecture (linux/amd64 unless your nodes are arm64).
docker build --platform linux/amd64 -t ghcr.io/<your-org>/my-agent:0.1.0 my-agent/

# Log in to your registry (one-time per machine).
docker login ghcr.io

# Push it.
docker push ghcr.io/<your-org>/my-agent:0.1.0
```

### Pushing to Quay (quay.io)

Quay is a good default; the platform's own base image lives there. Replace `<user-or-org>` with your Quay username or organization:

```sh
# Build and tag for Quay.
docker build --platform linux/amd64 -t quay.io/<user-or-org>/my-agent:0.1.0 my-agent/

# Log in. Use a robot account or an encrypted CLI password, NOT your web password:
#   Account Settings -> "Generate Encrypted Password" -> "Docker Login".
docker login quay.io

# Push. This also CREATES the repository the first time, as PRIVATE by default.
docker push quay.io/<user-or-org>/my-agent:0.1.0
```

**Make the package public.** Otherwise the agent will be stuck trying to start from an inaccessible image.

Notes:

- **Match the node architecture.** Most clusters run `linux/amd64`. Building on an Apple-silicon Mac without `--platform` produces an `arm64` image the cluster cannot run. To support both, use `docker buildx build --platform linux/amd64,linux/arm64 ... --push`.
- **Use immutable tags** (`:0.1.0`, a git SHA) rather than `:latest`, so a re-push does not change what running agents pull.

## Step 5 — Add the agent via the Custom Image field

In the UI, open **Add Agent**. Under **Custom Image**, paste your full image reference and click **Use**:

```
ghcr.io/<your-org>/my-agent:0.1.0
```

Then finish the configure step:

- Give the agent a **name**.
- **Connections → Provider:** attach an LLM provider to power the agent's model. Pick an existing preset (Anthropic, IBM LiteLLM); it supplies the model endpoint and token. Every agent needs this. Match the preset to your agent — for example, an OpenAI-compatible agent needs an OpenAI-compatible provider.
- If your agent calls an external service, attach a credential and allow its host here — see [how to call an external service](external.md).

No template registration, Helm edit, or platform redeploy is needed; the agent is created directly from the image URL.

## Calling an external service

If your agent calls an outside API, DAM never exposes the real API key inside the agent pod: you declare a placeholder env var in your Dockerfile (e.g. `ENV MY_API_KEY=dummy-placeholder`), and the platform injects the real credential on the wire. Your harness only needs the env var to *exist* — it should not validate the value.

For the full walkthrough — declaring the placeholder, creating the credential, allowing the destination host, and how the key reaches your tool — see [Calling an External Service](external.md).

## Constraints

- **ACP is required** — chat-mode harnesses must speak ACP over stdio.
- **Two scripts only** — the platform spawns exactly `harness-chat` or `harness-terminal`; wrap any multi-process setup inside one of these.
- **Non-root** — the container runs as UID 65532; always end your Dockerfile with `USER 65532`.
- **PVC survives hibernation** — `/home/agent/` (the agent's home directory) is persistent; the ephemeral filesystem is lost when the pod scales down.
- **No direct network access** — egress is controlled by NetworkPolicy; outbound traffic routes through the credential gateway.

## Troubleshooting

- **Stuck on "Starting…"** — the pod can't pull the image. Usually the registry package is **private** (make it public, or have an operator add a pull secret), or it was built for the wrong **architecture**.
- **"Send failed" in chat** — the chat (ACP) server didn't start or couldn't reach the model. Confirm your `harness-chat` execs your agent's ACP server (the base's placeholder won't), and that an **LLM provider** is attached.
