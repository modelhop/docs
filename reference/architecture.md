---
title: Architecture
slug: /reference/architecture
description: How ModelHop works internally — request flow, token management, and layer chaining.
order: 8
---

# Architecture

## Request flow

```
┌─────────────┐     ┌──────────────────────────────┐     ┌─────────┐     ┌──────────────────┐
│ Claude Code  │────▶│ ModelHop (:4080)              │────▶│ Layers  │────▶│ api.anthropic.com│
│              │◀────│                                │◀────│         │◀────│                  │
└─────────────┘     │ 1. Select account (in order)   │     └─────────┘     └──────────────────┘
                    │ 2. Replace Authorization header │
                    │ 3. Forward through layer chain  │
                    │ 4. On 429: retry next account   │
                    └──────────────────────────────────┘
```

### Step by step

1. **Request arrives** from Claude Code on `127.0.0.1:4080`
2. **Body buffered** — needed in case of 429 retry
3. **Account selected** — first available in sequence order
4. **Token validated** — refreshed if expiring within 5 minutes
5. **Auth header replaced** — original token swapped for account's token
6. **Request forwarded** — through layer chain to final target
7. **Response streamed** — piped back to Claude Code (SSE-compatible)

### On 429

1. Response body consumed (frees the socket)
2. Rate limit reset time parsed from response headers
3. Account marked as rate-limited until reset time
4. Next available account selected
5. Same request body replayed with new auth token
6. If all accounts exhausted, 429 returned to Claude Code

## Account selection

```
for each account in sequence order:
  skip if rate-limited (reset time in future)
  skip if no token loaded
  refresh token if expiring soon
  → return first match
```

Accounts always reset to order preference. If account 1 was rate-limited but has recovered, it's preferred over account 2 even if account 2 is also available.

## Token lifecycle

```
Keychain                     ModelHop memory
┌─────────────────┐          ┌─────────────────┐
│ accessToken      │──load──▶│ accessToken      │
│ refreshToken     │         │ refreshToken     │
│ expiresAt        │         │ expiresAt        │
└─────────────────┘          │ rateLimitedUntil │
        ▲                    └────────┬─────────┘
        │                             │
        └───write back on refresh─────┘
```

- Tokens loaded from macOS Keychain at startup
- Refreshed via OAuth when within 5 minutes of expiry
- Refreshed tokens written back to Keychain (keeps `ccs` in sync)
- Rate limit state is in-memory only (resets on proxy restart)

## Layer chain

Layers are resolved at startup into an ordered chain:

```
config.layers (enabled only) + config.target
         ↓
[layer1, layer2, ..., target]
         ↓
Request sent to layer1, which forwards to layer2, ..., which forwards to target
```

ModelHop sends to the **first hop** in the chain. Each subsequent hop must forward to the next. The final target (`api.anthropic.com`) is always the last entry.

## Signals

| Signal | Action |
|--------|--------|
| `SIGHUP` | Reload config.json, sequence.json, Keychain tokens, and resolve layer chain |
| `SIGTERM` | Graceful shutdown |
| `SIGINT` | Graceful shutdown (Ctrl+C) |

## Zero dependencies

ModelHop uses only Node.js built-in modules:

- `http` — local proxy server
- `https` — upstream API requests
- `fs` — config and sequence file reading
- `child_process` — macOS Keychain access via `security` CLI
- `path`, `os` — file path resolution
