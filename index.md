---
title: ModelHop
slug: /
description: Multi-account LLM proxy with automatic rate-limit rotation and pluggable layers.
order: 0
---

# ModelHop

Never hit a rate limit again. ModelHop is a lightweight local proxy that sits between your AI coding tools and LLM APIs, automatically rotating between accounts when one gets rate-limited.

## How it works

```
Claude Code → ModelHop (:4080) → [layers] → api.anthropic.com
```

1. You run `cc` (or `claude`) — it connects to ModelHop on localhost
2. ModelHop picks the first available account and injects its auth token
3. If the API returns 429, ModelHop instantly retries with the next account
4. Requests flow through any enabled layers (like Headroom for compression)
5. You keep working without interruption

## Features

- **Multi-account rotation** — round-robin through accounts on rate limits
- **Ordered preference** — always tries account 1 first, falls back to 2, 3, etc.
- **Rate limit tracking** — knows when each account resets, prefers available ones
- **Auto token refresh** — refreshes expired OAuth tokens via macOS Keychain
- **Pluggable layers** — chain middleware like Headroom for context compression
- **Zero dependencies** — Node.js stdlib only, single file
- **SSE streaming** — transparent passthrough for streaming responses

## Quick start

```bash
# Start the proxy
node index.js

# In another terminal, use Claude Code through the proxy
ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude
```

Or set up [shell aliases](/getting-started/aliases) for a seamless experience.
