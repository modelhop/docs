---
title: Layers
slug: /guides/layers
description: Add middleware layers to the request chain — compression, caching, logging, and more.
order: 4
---

# Layers

Layers are intermediate proxies that requests pass through between ModelHop and the LLM API. They let you add capabilities like context compression, request logging, or caching without modifying ModelHop itself.

## How layers work

```
Claude Code → ModelHop (:4080) → Layer 1 → Layer 2 → api.anthropic.com
                auth rotation     compression   logging     final target
```

ModelHop handles authentication and account rotation, then forwards through each enabled layer in order. The last hop is always the final target (`api.anthropic.com` by default).

## Configuration

Layers are defined in `config.json`:

```json
{
  "port": 4080,
  "target": "https://api.anthropic.com",
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": true
    },
    {
      "name": "logging-proxy",
      "url": "http://127.0.0.1:9090",
      "enabled": false
    }
  ]
}
```

## Enabling and disabling

Toggle a layer by setting `"enabled": true` or `false`. Reload without restart:

```bash
kill -HUP $(lsof -ti:4080)
```

## Layer requirements

Each layer must:

1. **Accept HTTP requests** on the configured host/port
2. **Forward requests upstream** to the next hop (or the final API)
3. **Pass through response headers and body** (including SSE streaming)

Most HTTP proxies satisfy these requirements out of the box.

## Layer order

Layers execute in array order. Put latency-sensitive layers first and optional/expensive layers last:

```json
{
  "layers": [
    { "name": "cache", "url": "http://127.0.0.1:9000", "enabled": true },
    { "name": "headroom", "url": "http://127.0.0.1:8787", "enabled": true }
  ]
}
```

Chain: `ModelHop → cache → headroom → api.anthropic.com`

## Checking the chain

```bash
$ claude-proxy-status | jq '.chain'
[
  "headroom (127.0.0.1:8787)",
  "target (api.anthropic.com:443)"
]
```

## Available layers

| Layer | Purpose | Docs |
|-------|---------|------|
| [Headroom](/guides/headroom) | Context compression (70-95% token savings) | [Setup guide](/guides/headroom) |

Want to build a custom layer? Any HTTP proxy that forwards requests will work. See the [layer requirements](#layer-requirements) above.
