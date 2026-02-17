# Setup Guide

## Prerequisites

- OpenClaw 2026.2.x or later
- At least two channels configured (Discord, Telegram, webchat, etc.)
- A cron-compatible model configured (any cheap model works)

## Quick Setup

### 1. Install the skill

```bash
# Into workspace skills (single agent)
git clone https://github.com/rajwaitforit/memory-sync.git ~/.openclaw/workspace/skills/memory-sync

# Or into managed skills (shared across all agents)
git clone https://github.com/rajwaitforit/memory-sync.git ~/.openclaw/skills/memory-sync
```

### 2. Add the cron job

Via OpenClaw cron tool or CLI. The job should:
- Run every 2 hours (`0 */2 * * *`)
- Use an isolated session (`sessionTarget: isolated`)
- Use a cheap model for token efficiency
- Read SKILL.md and execute the auto-sync workflow

### 3. Add agent instructions to AGENTS.md

Copy the "Immediate Write Rule" and "Channel Handoff Awareness" sections from SKILL.md into your workspace's AGENTS.md.

### 4. Create memory directory

```bash
mkdir -p ~/.openclaw/workspace/memory
```

## Troubleshooting

### Sync not running

- Verify the cron job exists: check your cron job list
- Check that the model specified is available in your auth profiles
- Review gateway logs for cron execution errors

### Context not crossing channels

- Confirm memory files exist in `~/.openclaw/workspace/memory/`
- Verify your AGENTS.md includes instructions to read daily memory files on session start
- Check that the sync job is writing to the correct path

### High token usage

- Increase the sync interval (every 4h instead of 2h)
- Use a cheaper model (Gemini Flash, GPT-4o-mini, Haiku)
- Reduce the session history scan depth

### Duplicate entries in memory files

- The sync workflow includes deduplication, but edge cases exist
- If duplicates appear, the sync model may need a more explicit dedup prompt
- Consider periodically cleaning memory files during heartbeat cycles
