# How to Create a LiteLLM Key

DAM works with several model providers. One of them is the **IBM LiteLLM ETE Proxy**, which gives you access to a curated set of models picked for DAM. When you're granted DAM access, you also get access to these models through the **`rits-restricted-agentic`** group. To use them, create a LiteLLM API key on the proxy — scoped to that group — and add it to DAM as a provider.

This guide walks through both steps. A key from any other IBM LiteLLM proxy won't work here, so create yours fresh on the proxy linked in Step 1. If you get stuck, ask in [#dam-community](https://ibm.enterprise.slack.com/archives/C0B3F03NB24).

## Models you get access to

These are the models available through the `rits-restricted-agentic` group:

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

!!! warning "`rits/zai-org/glm-5-1-fp8-agentic` unavailable"
    Following a [recent RITS GPU change](https://ibm-research.slack.com/archives/C07PNSBQMQW/p1782849277807129), the **GLM** endpoint `rits/zai-org/glm-5-1-fp8-agentic` may be unavailable. DeepSeek (`rits/deepseek-ai/DeepSeek-V3.2`) and Kimi (`rits/moonshotai/Kimi-K2.5`) are **not** affected. The Anthropic and other frontier models (`aws/`, `azure/`, `gcp/`) are also unaffected.

## Video Walkthrough

Prefer to watch? This covers the whole process end to end.

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
3. **Name your key** (required)
4. Under models, select **All Team Models**
5. Click **Create**

Selecting **All Team Models** grants the key access to every model in the group (listed at the top of this page).

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
