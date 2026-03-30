---
title: Account Management
slug: /getting-started/accounts
description: How ModelHop manages multiple accounts and handles rate-limit rotation.
order: 3
---

# Account Management

ModelHop reads your accounts from `~/.claude-switch-backup/sequence.json`, which is managed by the `ccs` (Claude Code Switcher) CLI.

## Account order

Accounts are tried in the order they appear in `ccs`:

```bash
$ ccs ls
  1. mdd@eupry.com (eupry) ← tried first
  2. mark@ulties.com       ← fallback on rate limit
```

ModelHop always prefers the lowest-numbered available account. When account 1's rate limit resets, it switches back to account 1 — even if account 2 is still available.

## Rate limit tracking

When an account gets a 429 response, ModelHop:

1. Parses the `Retry-After` or `x-ratelimit-reset` header
2. Records when that account becomes available again
3. Immediately retries the request with the next account
4. Automatically starts using the account again once the cooldown expires

If all accounts are rate-limited, the 429 is passed through to Claude Code.

## Token refresh

OAuth tokens expire periodically. ModelHop automatically refreshes them:

- Checks token expiry before each request
- Refreshes 5 minutes before expiry to avoid failed requests
- Writes refreshed tokens back to macOS Keychain (so `ccs` stays in sync)

## Adding accounts

```bash
# Switch to a new account in Claude Code
ccs sw

# Or add a specific account
ccs add

# ModelHop picks up changes on SIGHUP
kill -HUP $(lsof -ti:4080)
```

## Checking account status

```bash
$ claude-proxy-status
{
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
