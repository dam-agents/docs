# Calling an External Service

Allowing your sandbox to call an external service.

The key idea: **you never put the real API key in your image.** You give DAM the key once, and it injects it securely at request time so the key never actually sits inside the running sandbox.

There are three pieces.

### 1. A placeholder in the Dockerfile

Add an `ENV` line for whatever environment variable your tool reads its key from. The value is a **placeholder**, not the real key:

```dockerfile
ENV MY_TOOL_API_KEY=dummy-placeholder
```

DAM makes its CA available to your tools at startup, so common HTTP clients — Python's `requests`/`httpx`, Go, Node — trust the network automatically. No extra certificate configuration is needed.

### 2. A credential, created in the UI

On the **Connections** page click **New**, open the **Custom Headers** tab, and click **New** on the card:

- **Name:** any label, e.g. `my-tool-key`
- **Host:** the host your tool calls, e.g. `api.example.com`
- **Header name:** the header the API expects the key in — pre-filled with `Authorization`; change it if the API wants something else (e.g. `X-API-KEY`)
- **Value format:** pre-filled with `Bearer {value}`; use plain `{value}` if the API wants the raw key
- **Secret value:** the real key
- **Environment variable:** the same variable name from your Dockerfile, e.g. `MY_TOOL_API_KEY`. The
  form treats this as optional, and leaving it blank does **not** stop the header swap — the host is
  still allowed and the real key still injected on the wire. What you lose is the variable your tool
  reads to know what to send, so fill it in whenever your tool gets its key from the environment

Then add it to the sandbox from its **My connections** panel. (You can add it to an existing sandbox — no need to recreate it.) Adding the connection **automatically allows its configured host** on the sandbox's egress, so there's no separate Network access step for that host. Only *additional* hosts your tool calls need a manual rule under **Network access**. Hosts match exactly, so a rule for one host never covers its subdomains — `files.example.com` needs its own rule even if `api.example.com` is already allowed.

### 3. How the key reaches your tool

**The real key is never inside the running sandbox:**

1. Inside the sandbox, the environment variable holds your **placeholder** — not the real key. (`echo $MY_TOOL_API_KEY` prints the placeholder.)
2. Your tool reads that variable and puts it in the request header it sends.
3. As the request leaves for the credential's host, DAM **swaps the placeholder header for the real key, on the wire.** Only for that host.

The one consequence worth remembering:

- **Your tool has to actually send the request for the swap to happen.** If your client checks the key's *format* before sending — e.g. "must be 32 characters" or "must start with `sk-`" — and rejects the placeholder, no request goes out, so there's nothing to swap and the tool fails as if it had no key. **To fix that, make the placeholder the right shape** (right length, right prefix) so it passes that check. The exact characters don't matter — DAM overwrites them regardless. Tools that just send whatever you give them (plain `curl`, `requests`, most SDKs) work fine with `dummy-placeholder`.
