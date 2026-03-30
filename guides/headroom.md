---
title: Headroom Integration
slug: /guides/headroom
description: Reduce token usage by 70-95% with Headroom context compression.
order: 5
---

# Headroom Integration

[Headroom](https://github.com/chopratejas/headroom) is a context compression layer that reduces token usage by 70-95%. It compresses tool outputs, large code blocks, and conversation context before they reach the LLM — same answers, fraction of the tokens.

## How it fits

```
Claude Code → ModelHop (:4080) → Headroom (:8787) → api.anthropic.com
               auth rotation      compression        LLM API
```

ModelHop handles account rotation. Headroom compresses the request context. The LLM sees optimized input and responds normally.

## Install Headroom

```bash
pip install headroom
```

Or with pipx (recommended):

```bash
pipx install headroom
```

## Start Headroom

```bash
headroom proxy --port 8787
```

Headroom listens on port 8787 by default and forwards to `api.anthropic.com`.

### Headroom options

```bash
# With a daily budget cap
headroom proxy --port 8787 --budget 50.0

# Audit mode (observe without compressing)
headroom proxy --port 8787 --no-optimize

# With structured logging
headroom proxy --port 8787 --log-file /tmp/headroom.jsonl
```

## Configure ModelHop

Add Headroom as a layer in `config.json`:

```json
{
  "port": 4080,
  "target": "https://api.anthropic.com",
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": true
    }
  ]
}
```

Reload ModelHop to pick up the change:

```bash
kill -HUP $(lsof -ti:4080)
```

## Verify the chain

```bash
$ claude-proxy-status | jq '.chain'
[
  "headroom (127.0.0.1:8787)",
  "target (api.anthropic.com:443)"
]
```

## Headroom MCP tools (optional)

Headroom also provides MCP tools for on-demand compression within Claude Code. This is separate from the proxy layer and lets Claude request compression explicitly:

```bash
# Register Headroom's MCP tools with Claude Code
headroom mcp install
```

This adds three tools:

| Tool | Purpose |
|------|---------|
| `headroom_compress` | On-demand compression with hash-based retrieval |
| `headroom_retrieve` | Fetch original uncompressed content by hash |
| `headroom_stats` | Session statistics (compressions, savings, costs) |

## Headroom wrapper mode

Headroom can also wrap Claude Code directly (bypassing ModelHop). Use this only if you want Headroom without multi-account rotation:

```bash
headroom wrap claude
```

With ModelHop, you don't need this — the layer integration gives you both compression and account rotation.

## Monitoring

Check Headroom's compression stats:

```bash
# Headroom health
curl http://127.0.0.1:8787/health

# Detailed stats
curl http://127.0.0.1:8787/stats

# Prometheus metrics
curl http://127.0.0.1:8787/metrics
```

## Disabling Headroom

To temporarily disable without stopping Headroom:

Edit `config.json`:

```json
{
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": false
    }
  ]
}
```

Reload:

```bash
kill -HUP $(lsof -ti:4080)
```

ModelHop will skip Headroom and send requests directly to the API.
