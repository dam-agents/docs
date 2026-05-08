# Terminal Sessions

Access a live terminal (shell) in your agent's environment for direct command execution.

## Overview

Terminal sessions give you PTY-based shell access to the agent pod. Use them for:

- Running commands directly (build, test, deploy)
- Inspecting files and processes
- Debugging issues interactively
- Operations that are easier in a shell than in chat

## Key Behaviors

| Behavior | Detail |
|----------|--------|
| Single viewer | Terminal sessions support one viewer at a time |
| Persistence | Terminal state is maintained while the session is active |
| Workspace access | You're in the same filesystem as chat sessions (`/workspace`, `/home/agent`) |
| Scrollback | Full xterm scrollback buffer is preserved during the session |

## Reconnection

If you lose connection (close tab, network drop, laptop sleep):

- You have a **~30 second grace window** to reconnect
- If you reconnect within 30s, the session resumes exactly where you left off (output preserved)
- If you don't reconnect within 30s, the PTY process is terminated and the session ends
- Start a new terminal session if this happens — your workspace files are unaffected

## When to Use Terminal vs. Chat

| Use Terminal When | Use Chat When |
|-------------------|---------------|
| Running a specific command | Asking the agent to figure out what to do |
| Inspecting output interactively | Delegating a multi-step task |
| Debugging with breakpoints | Wanting the agent to write and test code |
| Need raw shell access | Wanting explanations alongside actions |
