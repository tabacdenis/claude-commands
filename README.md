# Claude Code Commands

Custom slash commands for VPS administration with [Claude Code](https://claude.com/claude-code).

## Available Commands

| Command | Description |
|---------|-------------|
| `/crowdsec-setup` | Install, diagnose and fix CrowdSec security engine. Detects running services, installs matching collections, configures log acquisition. |

## Installation

```bash
# Clone directly into Claude Code commands directory
git clone https://github.com/tabacdenis/claude-commands.git ~/.claude/commands
```

If `~/.claude/commands` already exists:

```bash
# Clone elsewhere and symlink
git clone https://github.com/tabacdenis/claude-commands.git ~/claude-commands
ln -sf ~/claude-commands/*.md ~/.claude/commands/
```

## Usage

In Claude Code, type `/crowdsec-setup` and the command will execute automatically.

## Updating

```bash
cd ~/.claude/commands && git pull
```
