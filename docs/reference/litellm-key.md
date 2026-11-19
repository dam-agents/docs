# Create your LiteLLM key

!!! note "IBM hosted deployment only"
    The IBM ETE LiteLLM Proxy is available to IBMers on the IBM hosted deployment. On your own
    deployment, connect any provider it supports, such as Anthropic, OpenAI, or AWS Bedrock.

DAM can use the **IBM ETE LiteLLM Proxy** as a model provider. To use it, you'll need an API key from the **external** LiteLLM proxy.

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

## Step 4: Add the Key in DAM

1. In DAM, open **Settings → Providers** and click **IBM ETE LiteLLM Proxy**
2. Paste your LiteLLM API key and click **Save**
3. To start new agents on it, pick it under **Default for new agents** at the top of the same tab. An existing agent switches on its own **Providers** page.

The key is saved once and reused by every agent you give it to. You can update or remove it any time on the same tab.

## Video Walkthrough

Prefer to watch? This covers the whole process end to end.

<video controls preload="metadata" style="width: 100%; max-width: 880px; border-radius: 8px; margin: 1rem 0;">
  <source src="../../assets/dam-litellm-key.mp4" type="video/mp4">
  Your browser doesn't support embedded video.
  <a href="../../assets/dam-litellm-key.mp4">Download the walkthrough</a> instead.
</video>
