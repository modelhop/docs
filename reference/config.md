---
title: Configuration
slug: /reference/config
description: Complete reference for config.json and environment variables.
order: 6
---

# Configuration

ModelHop is configured via `config.json` in the project root and environment variables.

## config.json

```json
{
  "port": 4080,
  "target": "https://api.anthropic.com",
  "layers": []
}
```

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `port` | number | `4080` | Port the proxy listens on |
| `target` | string | `https://api.anthropic.com` | Final upstream API URL |
| `layers` | array | `[]` | Middleware [layers](/guides/layers) in the request chain |

### Layer object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Display name for logs and status |
| `url` | string | yes | Layer's URL (e.g. `http://127.0.0.1:8787`) |
| `enabled` | boolean | yes | Whether to route through this layer |

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `4080` | Overrides `config.port` |

## Claude Code environment

Set this on the Claude Code side to route through ModelHop:

| Variable | Value | Description |
|----------|-------|-------------|
| `ANTHROPIC_BASE_URL` | `http://127.0.0.1:4080` | Tells Claude Code to use ModelHop |

## Account configuration

Accounts are not configured in ModelHop directly. They're managed by `ccs` and stored in:

| File | Purpose |
|------|---------|
| `~/.claude-switch-backup/sequence.json` | Account list and order |
| macOS Keychain (`Claude Code-Account-{n}-{email}`) | OAuth tokens per account |

## Hot reload

ModelHop reloads config, accounts, and layers on `SIGHUP`:

```bash
kill -HUP $(lsof -ti:4080)
```

This reloads without dropping active connections.

## Full example

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
      "name": "request-logger",
      "url": "http://127.0.0.1:9090",
      "enabled": false
    }
  ]
}
```
