# Egress Rules

Egress rules define which external services your agent is allowed to reach.

## Overview

Each agent has a set of egress rules that act as a network allowlist. Only services matching a rule can be accessed — everything else is blocked or requires [approval](approvals.md).

## Rule Format

| Field | Description | Example |
|-------|-------------|---------|
| Host | The domain the agent can reach | `api.github.com` |
| Method | HTTP method (optional, defaults to all) | `GET`, `POST` |
| Path | URL path pattern (optional) | `/repos/*` |

## How Rules Are Created

1. **From connections** — connecting GitHub automatically creates rules for `api.github.com` and `github.ibm.com`
2. **From approvals** — clicking "Allow always" on an approval creates a rule
3. **Manually** — add rules directly in the Egress Rules panel

## Managing Rules

- **Add** — click "Add Rule" and specify host/method/path
- **Delete** — remove a rule to revoke access (the agent will need approval next time)
