# 🛡️ OpenClaw Guardian

> A standalone watchdog that keeps your [OpenClaw](https://openclaw.ai) Gateway alive 24/7 — with automatic repair, git-based rollback, and optional Discord alerts.

## Features

- **Auto-monitor** — checks Gateway health every 30 seconds
- **Auto-repair** — runs `openclaw doctor --fix` on failure (up to 3 attempts)
- **Auto-rollback** — resets workspace to last stable git commit if repair fails
- **Daily snapshots** — automatic daily `git commit` of your workspace
- **Discord alerts** — optional webhook notifications on failures and recovery

## How It Works

```
Gateway down detected
        │
        ▼
  doctor --fix  ──→ success? ──→ ✅ Done
  (up to 3x)
        │ all failed
        ▼
  git rollback  ──→ success? ──→ ✅ Done
        │ failed
        ▼
  cooldown 300s → resume monitoring
```

## Quick Start

### 1. Initialize git in your workspace (required for rollback)

```bash
cd ~/.openclaw/workspace
git config --global user.email "guardian@example.com"
git config --global user.name "Guardian"
git init && git add -A && git commit -m "initial"
```

### 2. Install guardian.sh

```bash
cp scripts/guardian.sh ~/.openclaw/guardian.sh
chmod +x ~/.openclaw/guardian.sh
```

### 3. Start Guardian

```bash
nohup ~/.openclaw/guardian.sh >> /tmp/openclaw-guardian.log 2>&1 &
```

### 4. Auto-start on container restart

Add to `~/.openclaw/start-gateway.sh` (before the final `exec` line):

```bash
pkill -f "guardian.sh" 2>/dev/null || true
nohup /home/ubuntu/.openclaw/guardian.sh >> /tmp/openclaw-guardian.log 2>&1 &
```

### 5. Optional: Discord alerts

```bash
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
```

## Configuration

All settings via environment variables — no hardcoded values:

| Variable | Default | Description |
|---|---|---|
| `GUARDIAN_WORKSPACE` | `$HOME/.openclaw/workspace` | Path to workspace git repo |
| `GUARDIAN_LOG` | `/tmp/openclaw-guardian.log` | Log file path |
| `GUARDIAN_CHECK_INTERVAL` | `30` | Health check interval (seconds) |
| `GUARDIAN_MAX_REPAIR` | `3` | Max `doctor --fix` attempts before rollback |
| `GUARDIAN_COOLDOWN` | `300` | Cooldown after all repairs fail (seconds) |
| `DISCORD_WEBHOOK_URL` | _(unset)_ | Discord webhook for alerts (optional) |
| `OPENCLAW_CMD` | `openclaw` | OpenClaw CLI command |

## Verify

```bash
pgrep -a -f "guardian.sh"           # confirm process is running
tail -f /tmp/openclaw-guardian.log  # watch live logs
```

## Works Alongside gw-watchdog

Guardian complements (not replaces) the built-in `gw-watchdog.sh`:

| | gw-watchdog | Guardian |
|---|---|---|
| Check interval | 15s | 30s |
| Action | Fast restart | doctor --fix → rollback |
| Git rollback | ❌ | ✅ |
| Discord alerts | ❌ | ✅ |
| Daily backup | ❌ | ✅ |

Run both for layered resilience.

## Install as OpenClaw Skill

This repo is also published as an [OpenClaw AgentSkill](https://clawhub.ai) — your AI agent can deploy and manage Guardian automatically.

```bash
clawhub install openclaw-guardian
```

## License

MIT
