# Building a Custom Image

A custom image is a layer you build on top of DAM's base platform image, giving your agents access to your own tools, scripts, and environment. You build it, push it to a container registry, and point an agent at it by pasting the image address into the **Custom** card when you create a coding agent.

This guide is **agent-neutral**: the platform supports any agent — Claude Code, Codex, PI Agent, IBM Bob, or your own command-line program — and relies only on a small contract (covered in Step 3). The examples below use **Claude Code**.

The base image handles the platform plumbing:

- It runs the **agent-runtime** process — the ACP server for chat, the terminal and SSH relays, the file service, and the runtime channel — that connects your agent to the platform. You never modify it.
- It controls *when* your agent is launched (a chat session vs. a terminal session). You define *what* gets launched.
- It includes common tools: Node, Python, `git`, and `curl` ready to use, with `gh`, `jq`, and `uv` a call away.

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

- **Node 24** and `npm`, **Python 3.12**, plus `git`, `curl`, `tar`, `gzip`, and `mise`
- `gh`, `jq`, `rg`, `fd`, `uv`/`uvx`, `kubectl`, `oc`, and `gws` as lazy `mise` shims — on `PATH`, but
  the first call installs them through the egress gateway
- The **agent-runtime** process (the ACP server for chat, the terminal and SSH relays, the file
  service, and the runtime channel that delivers env, files, MCP entries, skills, and triggers)
- Default harness scripts you'll override in Step 3
- An SSH server, so [`dam ssh` and the editor modes](../integrations/vs-code-zed-ssh.md) work — keep it if you want
  them; removing `openssh-server` disables SSH for the image
- A non-root user (UID 65532)

The base is a RHEL image, but the final layer is built from a sysroot, so **it has no package
manager** — neither `dnf` nor `apt` is available at build time. Install with `npm`, `uv`, or by
downloading a binary; for an OS package, use the two-stage pattern shown below.

`PYTHONPATH` is preset to the platform's Python library directory. Append to it rather than
replacing it, or you'll break the experiment SDK.

The annotated skeleton follows. Most of it is fixed boilerplate; the parts you change are called out.

```dockerfile
FROM quay.io/dam-agents/platform-base:<version>  # (1) build on the base image; pin a version, not `latest`

USER root                                     # (2) switch to root to make changes

# (3) CUSTOMIZE: install any tools or libraries your agent should have.
#     The base ships node/npm, Python 3.12 and uv — but no package manager,
#     so dnf/apt are unavailable here. Install one of these ways:
#       - an OS package:    use the two-stage pattern below
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

# (7) CUSTOMIZE: seed files for the agent. Stage them at /app/working-dir/ --
#     NOT /home/agent. On first boot the platform copies this directory into the
#     agent's persistent home, which is where it starts and what survives
#     hibernation. This is how you give the agent instructions and files to
#     work from, e.g.:
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
- **Never `COPY` into `/home/agent`.** That path is a persistent volume at runtime, so anything baked there is hidden by the volume. Stage seed files at **`/app/working-dir/`** instead.
- **`workspace/` ships files to the agent.** The contents of `/app/working-dir/` are copied into the agent's home when it first starts. The copy happens *once*, without overwriting — it seeds initial files and does not push updates to an agent that has already run.
- **There is no package manager in the final image.** The base is built from a sysroot, so neither `dnf` nor `apt` exists at runtime. `npm`, `uv`, and downloading a binary all work directly. To add an OS package, install it in a separate builder stage and copy it in:

    ```dockerfile
    FROM registry.access.redhat.com/hi/core-runtime:latest-builder AS extra
    USER root
    RUN dnf install -y --installroot=/extra --use-host-config \
        --releasever="$(. /etc/os-release; echo "$VERSION_ID")" \
        --setopt=install_weak_deps=0 sqlite && dnf clean all
    # ...then, in your main stage:
    # COPY --from=extra /extra/usr/bin/sqlite3 /usr/bin/sqlite3
    # COPY --from=extra /extra/usr/lib64/libsqlite3.so.* /usr/lib64/
    # RUN sqlite3 -version   # fail the build if a shared library is missing
    ```

    A copied binary usually needs its shared libraries copied too, so run it once during the
    build — that turns a silent runtime failure into a build failure.

## Step 3 — Provide the session launchers

This is the entire contract between your image and the platform. When you open a session, the agent-runtime spawns one of two start scripts, and **you decide what each one launches**:

| Path | Mode | What it launches |
|------|------|------------------|
| `/usr/local/bin/harness-chat` | Chat | A program that speaks the [**Agent Client Protocol (ACP)**](https://agentclientprotocol.com/get-started/introduction) over stdio — the standard the chat window uses to exchange messages with an agent. By convention these programs are named `<agent>-acp`. |
| `/usr/local/bin/harness-terminal` | Terminal | Anything interactive attached to a PTY: your agent's command-line tool, or just a plain shell. |

Ship **both**, and override both. The base's `harness-chat` is a stub that exits with *"this agent
image does not provide a chat harness"*, and its `harness-terminal` execs `claude`, which the base
doesn't install — so neither works unmodified. (The built-in Claude Code image's scripts also source a
model-gateway shim before the exec. It only matters if your agent talks the Anthropic API and you
attach a proxy provider such as IBM LiteLLM — the shim is what points the agent at the in-pod
gateway. An agent that reads its provider settings straight from the environment needs nothing
extra, which is why the two lines below suffice.)

Both scripts are launched with their working directory set to **`/home/agent/work`**, so relative
paths resolve there.

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

**You can keep it private** — see [private registries](#private-registries) below. Making it public
also works, and is simpler if the image holds nothing sensitive.

Notes:

- **Match the node architecture.** Most clusters run `linux/amd64`. Building on an Apple-silicon Mac without `--platform` produces an `arm64` image the cluster cannot run. To support both, use `docker buildx build --platform linux/amd64,linux/arm64 ... --push`.
- **Use immutable tags** (`:0.1.0`, a git SHA) rather than `:latest`, so a re-push does not change what running agents pull.

## Step 5 — Create an agent from your image

From the sidebar, open **Coding agents** and click **Create coding agent**. Everything is on one page,
**Setup your coding agent**, filled top to bottom:

1. **Name** — what you'll call it.
2. **Image** — instead of picking a listed card, scroll past them to *or use a custom image* and paste
   your full reference into the **Custom** card (tagged *Advanced*):

    ```
    ghcr.io/<your-org>/my-agent:0.1.0
    ```

3. **Provider** — every agent needs one to power its model. The options are **IBM LiteLLM ETE Proxy**,
   **Bob Shell**, **Anthropic**, and **OpenAI**. Match it to your agent: an OpenAI-compatible harness
   needs an OpenAI-compatible provider.
4. **Connections** — attach any credentials your agent needs. If it calls an external service, that's
   where you attach the credential and allow its host — see
   [how to call an external service](external.md).

Then click **Create coding agent**.

Network access and **Size** aren't part of creation: a new agent starts on **Trusted defaults** at its
template's size, and both are editable straight afterward from the agent's **Agent Setup** page.

No template registration, Helm edit, or platform redeploy is needed; the agent is created directly
from the image address.

### Private registries

You don't have to publish the image publicly. Once the **Custom** card's address field is non-empty, a
**Private registry** section appears inside the card — fill in **Server** (e.g. `ghcr.io`),
**Username**, and **Password** — a PAT, robot account, or access token. All three or none; a partly
filled set blocks the create button.

DAM stores these as a pull secret scoped to that one agent, used only by the cluster to fetch the
image. The agent never sees it, and it's deleted with the agent.

!!! note "Two things to know"
    Credentials are only checked when the image is pulled, so a wrong one shows up as an
    image-pull failure on an agent that won't start — not as an error when you create it. And only
    long-lived credentials work; dynamically minted registry tokens (such as AWS ECR's) aren't
    supported.

## Calling an external service

If your agent calls an outside API, DAM never exposes the real API key inside the agent pod: you declare a placeholder env var in your Dockerfile (e.g. `ENV MY_API_KEY=dummy-placeholder`), and the platform injects the real credential on the wire. Your harness only needs the env var to *exist* — it should not validate the value.

For the full walkthrough — declaring the placeholder, creating the credential, allowing the destination host, and how the key reaches your tool — see [Calling an External Service](external.md).

## Constraints

- **ACP is required** — chat-mode harnesses must speak ACP over stdio.
- **Two scripts only** — the platform spawns exactly `harness-chat` or `harness-terminal`; wrap any multi-process setup inside one of these.
- **Non-root** — the container runs as UID 65532; always end your Dockerfile with `USER 65532`.
- **PVC survives hibernation** — `/home/agent/` (the agent's home directory) is persistent; the ephemeral filesystem is lost when the pod scales down. One exception: `~/.cache` is redirected to pod-local disk and starts empty on every wake, so don't stage anything there you need to keep.
- **No direct network access** — egress is controlled by NetworkPolicy; outbound traffic routes through the credential gateway.

## Troubleshooting

- **Stuck on "Starting"** — the pod can't pull the image. Usually the registry package is **private** and no [private-registry credentials](#private-registries) were entered (or they're wrong — they're only validated at pull time), or the image was built for the wrong **architecture**.
- **"Send failed" in chat** — the chat (ACP) server didn't start or couldn't reach the model. Confirm your `harness-chat` execs your agent's ACP server (the base's placeholder won't), and that an **LLM provider** is attached.
