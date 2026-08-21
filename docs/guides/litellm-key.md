# How to Create a LiteLLM Key

DAM can use the **IBM LiteLLM ETE Proxy** as a model provider. To use it, you’ll need an API key from the **external** LiteLLM proxy.

## Before you start

Open the external LiteLLM proxy and sign in with your IBM SSO:

[https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams](https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams)

You should see the **teams** you have access to.

!!! note
    If you are outside an IBM office, this proxy requires [TUNNEL ALL VPN](https://w3.ibm.com/#/support/article/troubleshoot_secure/cert_issues).

### If you already have a team

Great — you can use any team you already have access to. Go directly to Step 1 below.

### If you don't have a team

If you’re an IBM Research user, let us know when we follow up on your DAM access request or ask in [#dam-community](https://ibm.enterprise.slack.com/archives/C0B3F03NB24). We can request you access to the team dedicated to DAM, called "rits-restricted-agentic".

!!! note
    Getting added to the DAM team requires an access request and may take a few days.

If you're outside IBM Research, you'll need to use an existing team on the external proxy.

## Step 1: Open the External LiteLLM Proxy

Open the external LiteLLM proxy and sign in with your IBM SSO credentials:

[https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams](https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams)

!!! tip
    Bookmark this URL — you'll come back here whenever you need to manage your keys.

## Step 2: Create a Key

1. Click **"+ Create New Key"**
2. For **Team**, pick the team you're using with DAM — the one you already had, or "rits-restricted-agentic" if we added you to it
3. **Name your key** (required)
4. Under models, select **All Team Models**
5. Click **Create**

Selecting **All Team Models** grants the key access to every model in the team you picked.

## Step 3: Copy the Key Immediately

The full key is shown **only once**, at creation time.

!!! warning
    Copy the key now and store it somewhere safe. If you lose it you'll have to delete the key and create a new one — the proxy won't show it again.

The key looks like `sk-...`.

## Step 4: Add the Key When You Create a Sandbox

In DAM, the model provider is configured as part of creating a sandbox.

1. Start creating a sandbox to reach the **Setup your sandbox** step
2. In the provider list, select **IBM LiteLLM ETE Proxy**
3. Paste your LiteLLM API key
4. Continue setting up the sandbox

Once connected, the key is saved and reused — the next time you create a sandbox you can pick the same provider without pasting it again. You can manage your provider keys any time in DAM settings.

## Models in the DAM team

These are the models available through the team dedicated to DAM: "rits-restricted-agentic". A key scoped to a different team will offer a different list.

```
rits/deepseek-ai/DeepSeek-V3.2
rits/moonshotai/Kimi-K2.5
aws/claude-opus-4-8
aws/claude-sonnet-4-6
aws/claude-opus-4-5
azure/gpt-5.5
gcp/gemini-3.1-pro-preview
Azure/text-embedding-3-small-1
Azure/text-embedding-3-large-1
azure/gpt-5.3-codex
```

## Video Walkthrough

Prefer to watch? This covers the whole process end to end.

<video controls preload="metadata" style="width: 100%; max-width: 880px; border-radius: 8px; margin: 1rem 0;">
  <source src="../../assets/dam-litellm-key.mp4" type="video/mp4">
  Your browser doesn't support embedded video.
  <a href="../../assets/dam-litellm-key.mp4">Download the walkthrough</a> instead.
</video>
