# Setup Guide

## Quick Setup

### 1. Install the Skill

Copy the `memory-sync/` folder into your OpenClaw skills directory:

```bash
cp -r memory-sync/ ~/.openclaw/workspace/skills/memory-sync/
```

Or use the OpenClaw CLI:

```bash
openclaw skill install memory-sync
```

### 2. Add the Cron Job

Add to your OpenClaw config (`~/.openclaw/config.yaml` or via the dashboard):

```yaml
cron:
  - label: memory-sync
    schedule: "0 */2 * * *"
    prompt: |
      Read the memory-sync skill. Run the auto-sync workflow:
      1. List all active sessions via sessions_list
      2. For each session with recent activity, read history via sessions_history
      3. Extract: decisions, tasks, preferences, personal info, action items
      4. Write consolidated context to memory/YYYY-MM-DD.md
      5. If significant long-term info found, update MEMORY.md
    sessionTarget: isolated
    agentTurn: true
    model: google/gemini-2.0-flash-001
```

#### Customizing the Schedule

| Schedule | Cron Expression | Best For |
|----------|----------------|----------|
| Every 2 hours | `0 */2 * * *` | Default — good balance |
| Every hour | `0 * * * *` | Heavy multi-channel use |
| Every 4 hours | `0 */4 * * *` | Light use, save tokens |
| 3x daily | `0 8,14,20 * * *` | Structured schedule |

#### Choosing a Model

The sync job is read-heavy and doesn't need advanced reasoning. Recommended models:

- `google/gemini-2.0-flash-001` — Fast, cheap, good at extraction
- `openai/gpt-4o-mini` — Reliable alternative
- `anthropic/claude-3-haiku` — Good at summarization

### 3. Update AGENTS.md

Add the Immediate Write Rule and Channel Handoff Awareness sections from the SKILL.md
to your workspace's AGENTS.md. These ensure context is captured at the source and
retrieved on channel switches.

### 4. Create Memory Directory

```bash
mkdir -p ~/.openclaw/workspace/memory
```

The skill will create daily files automatically, but the directory must exist.

## Verification

After setup, test the sync:

1. Say something memorable on one channel (e.g., "I'm working on Project Alpha")
2. Wait for a sync cycle (or say "sync my memory")
3. Switch to another channel and ask "what am I working on?"
4. The agent should know about Project Alpha

## Troubleshooting

**Sync not running?**
- Check cron job is registered: `openclaw cron list`
- Verify the model is available in your config
- Check logs for errors

**Context not crossing channels?**
- Verify memory files exist: `ls ~/.openclaw/workspace/memory/`
- Check AGENTS.md includes the memory-reading instructions
- Ensure the agent reads daily memory files on session start

**Too many tokens used?**
- Increase sync interval (every 4h instead of 2h)
- Use a cheaper model
- Reduce session scan window (last 2h instead of 4h)
