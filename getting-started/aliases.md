---
title: Shell Aliases
slug: /getting-started/aliases
description: Set up shell aliases so every Claude session goes through ModelHop automatically.
order: 2
---

# Shell Aliases

Instead of typing `ANTHROPIC_BASE_URL=http://127.0.0.1:4080` every time, add these aliases to your shell config (`~/.zshrc`, `~/.bashrc`, or `~/.dotfiles/shell/.aliases`):

```bash
# Claude Code through ModelHop
alias cc='ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude'
alias ccr='ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude --resume'

# With --dangerously-skip-permissions
alias ccd='ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude --dangerously-skip-permissions'
alias ccdr='ANTHROPIC_BASE_URL=http://127.0.0.1:4080 claude --dangerously-skip-permissions --resume'

# Proxy management
alias claude-proxy='node /path/to/modelhop/index.js'
alias claude-proxy-status='curl -s http://127.0.0.1:4080/ | python3 -m json.tool'
```

Reload your shell:

```bash
source ~/.zshrc  # or ~/.bashrc
```

## Usage

```bash
# Start the proxy (keep running in a terminal or background)
claude-proxy

# Use Claude Code — automatically routed through ModelHop
cc
ccr            # resume last session
ccd            # with dangerous permissions
ccdr           # resume with dangerous permissions

# Check proxy status
claude-proxy-status
```

## Running in the background

To keep ModelHop running without a dedicated terminal:

```bash
# Start in background, log to file
claude-proxy > /tmp/modelhop.log 2>&1 &

# Check it's running
claude-proxy-status

# View logs
tail -f /tmp/modelhop.log
```
