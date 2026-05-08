# GitHub

Connect your GitHub account to give agents access to your repositories.

## Supported Platforms

DAM integrates with both:

| Platform | Domain | Use Case |
|----------|--------|----------|
| **GitHub.com** | `github.com` | Open-source repos, personal projects |
| **GitHub Enterprise** | Your org's domain (e.g., `github.ibm.com`) | Internal/corporate repositories |

Each is configured as a separate connection — you can have both active simultaneously.

## What Agents Can Do

With GitHub connected, your agents can:

- Clone private repositories
- Push branches and create pull requests
- Read and comment on issues
- Access GitHub APIs (commits, reviews, releases)

All without exposing your personal access token to the agent.

## Connect via OAuth

### GitHub.com

1. Navigate to **Connections** in the DAM UI
2. Click **"Connect GitHub"**
3. Complete the OAuth flow on github.com
4. Authorize the requested repository access

### GitHub Enterprise

1. Navigate to **Connections** in the DAM UI
2. Click **"Connect GitHub Enterprise"**
3. Complete the OAuth flow on your enterprise instance
4. Authorize the requested repository/org access

## Verify the Connection

Open a chat with your agent and try:

```
Clone github.ibm.com/my-org/my-repo into /workspace
```

or for github.com:

```
Clone github.com/my-org/my-repo into /workspace
```

If successful, the repo appears in your workspace.

## How It Works

Your GitHub token is stored securely and injected by the credential gateway (see [How Credentials Work](../security/index.md)). The agent never sees the raw token — it just sees successful authenticated requests.

The `GH_TOKEN` environment variable is automatically set so that tools like the `gh` CLI work out of the box.

## Common Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Clone fails with 403 | Token doesn't have access to that org | Re-authorize with broader org access |
| Push rejected | Branch protection rules | Create a PR instead of pushing directly |
| "Egress rule required" | First time reaching the GitHub host | Approve the egress request |
| Wrong account used | Multiple GitHub connections | Check which connection is active in Settings |
