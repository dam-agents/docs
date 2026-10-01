# Create your LiteLLM key

DAM can use the **IBM LiteLLM ETE Proxy** as a model provider. To use it, you'll need an API key from the **external** LiteLLM proxy.

## Before you start

Open the external LiteLLM proxy and sign in with your IBM SSO:

[https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams](https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams)

You should see the **teams** you have access to. Any team you already have access to
will work — pick whichever one you want to use with DAM.

!!! note
    If you are outside an IBM office, this proxy requires **TUNNEL ALL VPN** — a stronger mode than the standard VPN DAM itself needs. You only need TUNNEL ALL for this key-creation step, not for using DAM day to day.

## Step 1: Open the External LiteLLM Proxy

Open the external LiteLLM proxy and sign in with your IBM SSO credentials:

[https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams](https://ete-litellm.ai-models.vpc.res.ibm.com/ui/?page=teams)

!!! tip
    Bookmark this URL — you'll come back here whenever you need to manage your keys.

## Step 2: Create a Key

1. Click **"+ Create New Key"**
2. For **Team**, pick the team you're using with DAM
3. **Name your key** (required)
4. Under models, select **All Team Models**
5. Click **Create**

Selecting **All Team Models** grants the key access to every model in the team you picked.

## Step 3: Copy the Key Immediately

The full key is shown **only once**, at creation time.

!!! warning
    Copy the key now and store it somewhere safe. If you lose it you'll have to delete the key and create a new one — the proxy won't show it again.

The key looks like `sk-...`.

## Step 4: Add the Key When You Create an Agent

In DAM, the model provider is configured as part of creating an agent.

1. Start creating an agent — **Create agent** on **Home** — and scroll to **Provider**
2. In the provider list, select **IBM LiteLLM ETE Proxy**
3. Paste your LiteLLM API key
4. Continue setting up the agent

Once connected, the key is saved and reused — the next time you create an agent you can pick the same provider without pasting it again. You can manage your provider keys any time in DAM settings.

## Video Walkthrough

Prefer to watch? This covers the whole process end to end.

<video controls preload="metadata" style="width: 100%; max-width: 880px; border-radius: 8px; margin: 1rem 0;">
  <source src="../assets/litellm-key.mp4" type="video/mp4">
  Your browser doesn't support embedded video.
  <a href="../assets/litellm-key.mp4">Download the walkthrough</a> instead.
</video>
