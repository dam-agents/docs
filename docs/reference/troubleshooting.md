# Troubleshooting

## Agent Issues

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Agent stuck in "Starting" | Pod provisioning delay | Wait 30-60 seconds; if it persists, delete and recreate |
| Agent not responding | Pending approval blocking it | Check your Inbox for pending approvals |
| Agent waking slowly | Normal hibernation wake time | Allow ~20 seconds |

## Credential Issues

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| GitHub clone fails with 403 | Token lacks org access | Reconnect GitHub with broader org permissions |
| "Egress rule required" popup | First time reaching this host | Approve the egress request |
| "No API key configured" | Anthropic key not connected | Add your key in Connections → Secrets |

## Slack Issues

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Bot not responding | Identity not linked | Run `/dam login` in Slack |
| "Not authorized" | Not on allowed users list | Ask the agent owner to add you |
| Messages delayed | Agent waking from hibernation | Wait ~20 seconds |

## Session Issues

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Session not resuming | Agent was deleted and recreated | Start a fresh session |
| Lost conversation history | Agent was deleted | Deletion is permanent; cannot be recovered |
