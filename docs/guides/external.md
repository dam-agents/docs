# Calling an External Service

Allowing your agent to call an external service.

The key idea: **you never put the real API key in your image.** You give the platform the key once, and it injects it securely at request time so the key never actually sits inside the running agent.

There are three pieces.

### 1. A placeholder in the Dockerfile

Add an `ENV` line for whatever environment variable your tool reads its key from. The value is a **placeholder**, not the real key:

```dockerfile
ENV MY_TOOL_API_KEY=dummy-placeholder
```

The platform installs its CA into the image's system trust store at startup, so tools that read the system store — Python's `requests`/`httpx`, Go, Node — trust the platform's network automatically. No extra certificate configuration is needed.

### 2. A credential, created in the UI

Under **Connections → Other → Custom header credential**, create the credential:

- **Name:** any label, e.g. `my-tool-key`
- **Host:** the host your tool calls, e.g. `api.example.com`
- **Header name:** the header the API expects the key in (e.g. `Authorization` or `X-API-KEY`)
- **Value format:** `{value}` for the raw key, or `Bearer {value}` if the API wants that prefix
- **Secret value:** the real key
- **Environment variable:** the same variable name from your Dockerfile, e.g. `MY_TOOL_API_KEY`

Then grant it to the agent from the agent's **Connections** panel. (You can add it to an existing agent — no need to recreate it.) Granting the connection **automatically allows its configured host** on the agent's egress, so there's no separate Network access step for that host. Only *additional* hosts your tool calls need a manual rule under **Network access**.

### 3. How the key reaches your tool

**The real key is never inside the running agent:**

1. Inside the agent, the environment variable holds your **placeholder** — not the real key. (`echo $MY_TOOL_API_KEY` prints the placeholder.)
2. Your tool reads that variable and puts it in the request header it sends.
3. As the request leaves for the credential's host, the platform **swaps the placeholder header for the real key, on the wire.** Only for that host.

The one consequence worth remembering:

- **Your tool has to actually send the request for the swap to happen.** If your client checks the key's *format* before sending — e.g. "must be 32 characters" or "must start with `sk-`" — and rejects the placeholder, no request goes out, so there's nothing to swap and the tool fails as if it had no key. **To fix that, make the placeholder the right shape** (right length, right prefix) so it passes that check. The exact characters don't matter — the platform overwrites them regardless. Tools that just send whatever you give them (plain `curl`, `requests`, most SDKs) work fine with `dummy-placeholder`.
