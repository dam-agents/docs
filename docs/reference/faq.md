# FAQ

## General

**What happens to my work when the agent hibernates?**  
Your workspace files, session history, installed skills, and configuration all persist. Only running processes and in-memory state are lost.

**Can I run multiple agents simultaneously?**  
Yes. Each agent is independent — its own workspace, credentials, and sessions.

**How do I reset my agent's workspace?**  
Delete the agent and create a new one. Deletion is permanent.

**Can I share an agent with my team?**  
Yes, via Slack. Connect your agent to a channel and add colleagues to the allowed users list.

## Credentials

**Is my API key safe?**  
Yes. Credentials are stored in a secure vault and injected by a separate gateway pod. The agent never has access to raw tokens. See [How Credentials Work](../security/index.md).

**What if my token expires?**  
OAuth tokens are auto-refreshed. API keys don't expire unless you rotate them at the provider.

## Agents

**Why is my agent not responding?**  
It may be waking from hibernation (~20 seconds). If it still doesn't respond, check the Inbox for pending approvals.

**Can I change an agent's template after creation?**  
No. Create a new agent with the desired template instead.
