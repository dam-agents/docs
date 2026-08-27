# VS Code/Zed (SSH)

You can open a running agent's workspace directly in **VS Code** (or **Zed**) and work in it like any other remote folder — editing files, running the terminal, debugging — with the agent's full environment behind it.

This works because `dam ssh` gives you a **real SSH session** into the agent: not the harness TUI you get from `dam chat`, but a plain `/bin/bash` login shell. Because it's standard SSH, everything that rides on SSH works too — `scp`, `sftp`, and port-forwarding — but opening the workspace in your editor is the part most people want.

The session inherits the agent's full environment: the egress proxy, the credential placeholders your tools read, and `PATH`. So `git`, `gh`, and the AI CLIs behave exactly as they do for the harness itself.

!!! info "How it works (you don't need to set any of this up)"
    There's no SSH port exposed on the agent. The connection is tunneled over its own authenticated WebSocket — the same login and ownership check `dam chat` goes through — and SSH terminates at a small `sshd` running *inside* the agent. Your platform login is the real trust boundary; `dam` manages a dedicated SSH keypair for the transport so it never touches your personal `~/.ssh` keys.

## Prerequisites

- The `dam` CLI installed and logged in — see the [CLI reference](../reference/cli.md) (`dam auth login`).
- The OpenSSH client (`ssh`, `ssh-keygen`) on your machine. Present by default on macOS and Linux.
- An agent that isn't stopped. Connecting wakes a hibernated agent automatically; a **stopped** one has to be started first. (An open SSH connection then keeps it awake — see [Keeping agents awake](#keeping-agents-awake) below.)
- For the editor modes: `code` (or `code-insiders`) / `zed` on your `PATH`.

!!! note "Custom images need `sshd`"
    SSH only works on images that ship an SSH server. Every stock image does, because the
    base image includes one — but if you [build a custom image](../guides/custom-image.md) from
    something else, or strip `openssh-server` out, `dam ssh` has nothing to connect to.

## Open a shell

```bash
dam ssh connect my-agent
```

This drops you into a login shell on the agent, starting in the home directory `/home/agent`. The
workspace the agent works in is `~/work` — `cd ~/work` to get there; the editor modes below open it
directly. Exit with `exit` or `Ctrl-D`.

The agent header's **Open in → VS Code / Zed (local)** gives you the same commands, pre-filled with
the agent's ID.

Unlike the web terminal, SSH has no one-viewer limit: a shell and an editor window can be connected
to the same agent at once, each with its own independent connection. SSH connections aren't
sessions, so they don't appear in the agent's session list.

You can reference the agent by **name** or by **ID** (`agent-…`), the same as every other `dam` command.

## Open in VS Code

```bash
dam ssh connect my-agent -x code
```

This launches VS Code with a Remote-SSH window pointed at the agent's workspace. Use `-x code-insiders` for the Insiders build.

!!! tip "First connection downloads the VS Code server"
    On the first connect, VS Code installs its remote server inside the agent. `dam` pre-approves the two hosts that download needs (`update.code.visualstudio.com`, `vscode.download.prss.microsoft.com`) so you don't get a network-approval prompt mid-connect. This is best-effort and idempotent — if it can't seed the rule it warns and continues.

## Open in Zed

```bash
dam ssh connect my-agent -x zed
```

## Use scp / sftp / port-forwarding

Once a host is configured (see [Wiring up other tools](#wiring-up-other-tools-scp-sftp-tunnels) below), the alias works with any SSH-based tool:

```bash
dam ssh configure my-agent          # writes the host alias, prints it

scp ./local.txt dam-my-agent:work/        # copy a file in
scp dam-my-agent:work/out.log .           # copy a file out
sftp dam-my-agent                          # interactive sftp
ssh -L 8080:localhost:3000 dam-my-agent    # forward a port
```

!!! note
    `scp` and `sftp` need an `sftp-server` binary in the image. Every stock image has one; a
    [custom image](../guides/custom-image.md) built from a slimmer base may not, in which case `ssh` and
    port-forwarding still work but file transfer doesn't.

## Choosing the client (`-x` / `--exec`)

`-x, --exec` picks which client `dam ssh connect` launches:

| Command | Launches |
|---------|----------|
| `dam ssh connect <agent>` | the system `ssh` (default) |
| `dam ssh connect <agent> -x code` | VS Code Remote-SSH |
| `dam ssh connect <agent> -x zed` | Zed |
| `dam ssh connect <agent> -x code-insiders` | VS Code Insiders (name recognized) |

The mode is normally inferred from the binary's name (`code`/`code-insiders` → VS Code, `zed` → Zed). If you point `-x` at a binary whose name doesn't give it away, append a `:mode` suffix to force it:

```bash
dam ssh connect my-agent -x /opt/builds/my-code:code
```

The mode suffix is one of `ssh`, `code`, or `zed`.

## Wiring up other tools (scp, sftp, tunnels)

`dam ssh configure` writes a managed SSH host entry **without launching anything**, so any SSH-based tool can resolve the agent by an alias:

```bash
dam ssh configure my-agent
```

It prints the alias — `dam-` plus the name or ID you passed, e.g. `dam-my-agent` — and example `ssh` / `code` / `zed` invocations. Under the hood `dam` keeps its host entries in its own config file and adds a single `Include` line to your `~/.ssh/config` — it never rewrites your existing SSH config.

Configure or clean up in bulk:

```bash
dam ssh configure --all       # one host entry per agent on the active host
dam ssh configure --clear     # remove every dam-managed host entry
```

`--all` is a reconcile against the **active host only**: it adds an entry for each of that host's
agents, then removes *every* other dam-managed entry. `--clear` removes all of `dam`'s managed
entries (and its `Include` line once empty).

!!! warning "`--all` prunes more than deleted agents"
    There is one managed config for all your hosts, so an `--all` run drops entries for agents on
    every *other* server you use. It also writes aliases from agent **names**, so an entry you
    created by ID (`dam ssh configure agent-…`) is pruned and re-added under the name. Re-run
    `--all` per host, or configure those agents individually.

## Keeping agents awake

An open SSH connection counts as activity — like `dam chat` or the web terminal, it pins the agent so it **won't hibernate while you're connected**.

!!! warning "A forgotten editor window keeps the agent running"
    The SSH stream is opaque to DAM, so it can't tell a real keystroke from a VS Code Remote-SSH keepalive. A connected-but-idle editor window therefore keeps the agent awake, holding its reserved compute. **Close the window to let the agent hibernate.** If your connection half-dies, DAM detects it and releases the pin automatically.

## Troubleshooting

??? question "`ssh` / `code` / `zed` not found on PATH"
    `dam ssh connect` hands off to the client you selected. Install the client (OpenSSH for `ssh`, VS Code for `-x code`, Zed for `-x zed`) and make sure it's on your `PATH`.

??? question "VS Code pops a network-approval prompt on first connect"
    That's the VS Code server download. `dam` tries to pre-approve those hosts, but if it couldn't (e.g. your account can't edit network rules), approve the request in the web UI's inbox, or add `update.code.visualstudio.com` and `vscode.download.prss.microsoft.com` under the agent's **Network access**.

??? question "Host key warnings"
    `dam ssh` intentionally disables host-key checking. The connection is already authenticated and encrypted by the WebSocket tunnel, so the in-pod host key authenticates nothing extra — and because it lives on the agent's own volume, replacing that volume would otherwise trigger spurious "host identification changed" failures. You won't see `known_hosts` prompts.

??? question "The shell starts but `git` / network calls fail"
    The agent's environment is replayed into each SSH session on connect. If you added a connection or credential *after* opening the session, reconnect to pick it up — the environment is refreshed per connection, not live within one.

## See also

- [CLI reference → SSH access](../reference/cli.md#ssh-access) — the full command surface.
- [Network & Approvals](../core-concepts/network-access.md) — how egress rules and the approval inbox work.
- [Sessions](../core-concepts/sessions.md) — `dam chat` and the harness TUI, the other way to attach.
