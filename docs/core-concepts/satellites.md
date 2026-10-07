# Satellites

!!! warning "Experimental"
    Satellites are experimental. What a satellite offers, and how agents see it, may change between
    releases.

A satellite is a machine outside DAM, such as your laptop, a build box, or a server in a locked-down
network, that offers tools to your agents. It connects out to DAM over HTTPS, so nothing has to reach in:
no open port and no inbound firewall rule. A satellite is yours alone, and only the agents you add it to
can call its tools.

## Two kinds of satellite

You start a satellite on the machine itself, with the `dam` CLI:

- **An MCP server.** `dam satellite mcp` either starts the server you name after `--`, or forwards to one
  already running there with `--url`. Its tools are offered as they are, and the server decides what each
  call may do.
- **A fixed set of commands.** `dam satellite shell` takes one usage line per command you permit, and
  offers them to agents as a single tool. The machine checks every call against those lines and refuses
  anything that matches none of them. The first word of each line has to be a fixed program, so an agent
  can never choose what runs, and the command is run directly rather than through a shell.

```bash
dam satellite shell --name gpu-box --cwd /srv --timeout 6h <<'EOF'
./process.sh (sales.db|events.db)  # Process a database
./train.sh ./data/**/*.db          # Train  [max=1 timeout=2h]
EOF
```

Either way the command stays in the foreground and serves until you stop it. Whatever it runs inherits
the environment of the shell you started it from, so start it from one that holds only what you're
happy for the agent to use. The usage-line syntax and every flag are in the
[CLI reference](../reference/cli.md#satellites).

The machine has to be signed in to DAM. `dam auth login` works. For a machine that stays up, an API key
with only the `satellites:serve` scope is the narrower choice: it can serve a satellite but can't
operate your agents. See [the CLI reference](../reference/cli.md#authentication) for using a key.

## Adding it to an agent

Once a satellite has connected, and not before, it shows up in a **Satellites** card under
**Settings → Connections** and on the **MCP servers** tab of the connection catalogue.

To give an agent its tools, open the agent's **Connections** panel, click **New**, go to the
**MCP servers** tab, and click **Add to agent** on the satellite. The same catalogue is offered while
you create an agent. From the command line, `dam satellite grant <satellite> <agent>` does the same. The
agent picks up the tools the next time its harness starts.

## What the agent can do

The agent sees each of the satellite's tools. A call that finishes quickly returns its result straight
away. A longer one keeps running on the machine, and its outcome comes back to the agent when it's done.
A call to a satellite that's offline or shutting down is refused rather than queued, and so is one that
would take it over its limit of jobs at once.

## States

Each satellite's row shows how many tools it offers and the machine it runs on. A satellite that's
online and serving shows nothing more. Otherwise it carries one of these badges:

| Badge | Meaning |
|-------|---------|
| **Shutting down** | It's finishing its running jobs and taking no new ones |
| **Offline · seen …** | It isn't connected; the badge says how long ago it was last seen |
| **Never connected** | DAM has no record of it being online |

The row's **⋯** menu offers **Show tools**, plus **Remove from this agent** in an agent's panel and
**Remove this satellite** under Settings and in the catalogue. Removing a satellite takes its tools
away from every agent and deletes its job history, but it doesn't stop commands already running on the
machine.
