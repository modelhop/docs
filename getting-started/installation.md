---
title: Installation
slug: /getting-started/installation
description: Install ModelHop and configure your first account.
order: 1
---

# Installation

## Prerequisites

- **Node.js** >= 18
- **macOS** (uses Keychain for credential storage)
- **ccs** (Claude Code Switcher) — for managing multiple accounts

## Install ModelHop

Clone the repository:

```bash
git clone https://github.com/modelhop/modelhop.git
cd modelhop
```

No `npm install` needed — ModelHop has zero dependencies.

## Set up accounts with ccs

ModelHop reads account credentials from the `ccs` CLI. If you don't have accounts set up yet:

```bash
# Install ccs
brew install ccs

# Add your current Claude Code account
ccs add

# Log in to another account in Claude Code, then add it too
ccs add

# List your accounts
ccs ls
```

This creates `~/.claude-switch-backup/sequence.json` with your account order.

## Verify setup

```bash
# Start the proxy
node index.js
```

You should see:

```
[...] Loading accounts from sequence.json...
[...] Loaded account 1: you@example.com (expires ...)
[...] Loaded account 2: other@example.com (expires ...)
[...] Chain: ModelHop → target
[...] Proxy listening on http://127.0.0.1:4080
```

## Test it

```bash
ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude -p 'say hello'
```

If you get a response, you're all set. Next: [set up shell aliases](/getting-started/aliases) for a seamless workflow.
