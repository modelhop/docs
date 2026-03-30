---
title: Status API
slug: /reference/api
description: ModelHop's built-in status and health endpoint.
order: 7
---

# Status API

ModelHop exposes a single status endpoint for monitoring.

## GET /

Returns the current state of accounts, layers, and request stats.

```bash
curl http://127.0.0.1:4080/
```

### Response

```json
{
  "status": "ok",
  "uptime": 3600.5,
  "started": "2026-03-31T08:00:00.000Z",
  "totalProxied": 142,
  "totalRetries": 3,
  "chain": [
    "headroom (127.0.0.1:8787)",
    "target (api.anthropic.com:443)"
  ],
  "layers": [
    {
      "name": "headroom",
      "url": "http://127.0.0.1:8787",
      "enabled": true
    }
  ],
  "accounts": [
    {
      "num": 1,
      "email": "mdd@eupry.com",
      "available": true,
      "rateLimitedUntil": null,
      "tokenExpiresAt": "2026-03-31T16:20:01.483Z",
      "hasToken": true
    },
    {
      "num": 2,
      "email": "mark@ulties.com",
      "available": false,
      "rateLimitedUntil": "2026-03-31T12:05:00.000Z",
      "tokenExpiresAt": "2026-03-31T15:31:39.717Z",
      "hasToken": true
    }
  ]
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Always `"ok"` |
| `uptime` | number | Seconds since proxy started |
| `started` | string | ISO timestamp of proxy start |
| `totalProxied` | number | Total requests forwarded |
| `totalRetries` | number | Total 429 retries across accounts |
| `chain` | string[] | Resolved request chain (layers + target) |
| `layers` | object[] | Configured layers with enabled state |
| `accounts` | object[] | Per-account status |

### Account fields

| Field | Type | Description |
|-------|------|-------------|
| `num` | number | Account number (from ccs) |
| `email` | string | Account email |
| `available` | boolean | Can this account serve requests right now |
| `rateLimitedUntil` | string\|null | ISO timestamp when rate limit expires |
| `tokenExpiresAt` | string\|null | ISO timestamp when OAuth token expires |
| `hasToken` | boolean | Whether a valid token is loaded |

## All other requests

Any request other than `GET /` is proxied upstream through the account rotation and layer chain.
