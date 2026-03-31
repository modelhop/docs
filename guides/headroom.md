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
pipx install "headroom-ai[proxy]"
```

This installs the `headroom` CLI with proxy support. Requires Python 3.10+.

> **Note**: The package name on PyPI is `headroom-ai`, not `headroom`. The CLI command is `headroom`.

## Configure ModelHop

Add Headroom as a layer with autostart in `config.json`:

```json
{
  "port": 4080,
  "target": "https://api.anthropic.com",
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": true,
      "autostart": {
        "command": "headroom proxy --port 8787 --no-telemetry",
        "healthcheck": "http://127.0.0.1:8787/health",
        "log": "/tmp/headroom.log",
        "pid": "/tmp/headroom.pid"
      }
    }
  ]
}
```

That's it. The next time you run `cc` (or any ModelHop alias), Headroom starts automatically if it's not already running.

## Autostart behavior

When you run `cc`, the ModelHop launcher:

1. Checks if Headroom is responding on its healthcheck URL
2. If not, starts it using the `autostart.command`
3. Waits up to 10 seconds for the healthcheck to pass
4. Then starts the ModelHop proxy (which routes through Headroom)
5. Finally launches Claude Code

On subsequent runs, both processes are already running — startup is instant.

### Autostart fields

| Field | Required | Description |
|-------|----------|-------------|
| `command` | yes | Shell command to start the layer |
| `healthcheck` | no | URL to poll for readiness (defaults to layer `url`) |
| `log` | no | Log file path (defaults to `/tmp/{name}.log`) |
| `pid` | no | PID file path (defaults to `/tmp/{name}.pid`) |

## Manual start (optional)

If you prefer to manage Headroom yourself instead of using autostart:

```bash
headroom proxy --port 8787
```

Remove the `autostart` block from config.json — ModelHop will still route through Headroom when it's running, and the proxy will return a 502 if it's not.

### Headroom options

```bash
# With a daily budget cap
headroom proxy --port 8787 --budget 50.0

# Audit mode (observe without compressing)
headroom proxy --port 8787 --no-optimize

# With structured logging
headroom proxy --port 8787 --log-file /tmp/headroom.jsonl

# With persistent memory
headroom proxy --port 8787 --memory

# With traffic learning (extracts error patterns)
headroom proxy --port 8787 --learn
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

## Monitoring

```bash
# View Headroom logs
tail -f /tmp/headroom.log

# Headroom health
curl http://127.0.0.1:8787/health

# Detailed stats
curl http://127.0.0.1:8787/stats
```

## Disabling Headroom

Set `"enabled": false` in config.json:

```json
{
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": false,
      "autostart": { "..." : "..." }
    }
  ]
}
```

Reload the proxy:

```bash
kill -HUP $(lsof -ti:4080)
```

ModelHop will skip Headroom and send requests directly to the API. The `autostart` config is ignored when `enabled` is false.
