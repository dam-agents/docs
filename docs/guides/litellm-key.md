# How to Create a LiteLLM Key

DAM agents reach Claude (and other models) through the **IBM LiteLLM ETE Proxy**. To connect an agent to the proxy you need a LiteLLM API key, which you add to DAM as a model provider.

This guide walks through generating that key and wiring it into DAM.

!!! note
    This isn't a DAM-only proxy — it's a separate LiteLLM proxy at `ete-litellm.ai-models.vpc.res.ibm.com` with its own group (`rits-restricted-agentic`) that grants the specific models DAM uses. If you've used a LiteLLM proxy elsewhere at IBM, that key won't work here. You need a key created on this proxy and scoped to this group, as shown below.

## Video Walkthrough

<video controls preload="metadata" style="width: 100%; max-width: 880px; border-radius: 8px; margin: 1rem 0;">
  <source src="../../assets/dam-litellm-key.mp4" type="video/mp4">
  Your browser doesn't support embedded video.
  <a href="../../assets/dam-litellm-key.mp4">Download the walkthrough</a> instead.
</video>

## Step 1: Open the LiteLLM Proxy

Open the IBM LiteLLM ETE Proxy and sign in with your IBM SSO credentials:

[https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=api-keys](https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=api-keys){:target="_blank"}

!!! tip
    Bookmark this URL — you'll come back here whenever you need to manage your keys.

## Step 2: Create a Key

1. Click **"+ Create New Key"**
2. For **Team**, specify **`rits-restricted-agentic`**
3. Under models, select **All Team Models**
4. Click **Create**

The team grants access to these models:

```
rits/zai-org/glm-5-1-fp8-agentic
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

## Step 3: Copy the Key Immediately

The full key is shown **only once**, at creation time.

!!! warning
    Copy the key now and store it somewhere safe. If you lose it you'll have to delete the key and create a new one — the proxy won't show it again.

The key looks like `sk-...`.

## Step 4: Add the Key When You Create a Sandbox

In DAM, the model provider is configured as part of creating a sandbox.

1. Start creating a sandbox to reach the **Configure your sandbox** step
2. In the provider list, select **IBM LiteLLM ETE Proxy**
3. Paste your LiteLLM API key
4. Continue setting up the sandbox

Once connected, the key is saved and reused — the next time you create a sandbox you can pick the same provider without pasting it again. You can manage your provider keys any time in DAM settings.
