# 🧠 Memory Sync

**An OpenClaw skill that syncs context across channels — Discord, Telegram, webchat, and more.**

## The Problem

Every channel in OpenClaw gets its own session. Say something on Discord, switch to Telegram, and it's gone. You end up repeating yourself.

## How It Works

Memory Sync uses shared memory files as a bridge between isolated sessions:

```
Discord ──┐                  ┌── Discord
Telegram ─┤── writes to ──►  │
Webchat ──┘   memory files   ├── reads from ──► All sessions stay in sync
              (daily + long   │
               term)          └── Telegram, Webchat, etc.
```

Three mechanisms work together:

| Mechanism | How | When |
|-----------|-----|------|
| **Auto Sync** | Cron job scans all sessions, extracts key context, writes to shared memory | Every 2 hours (configurable) |
| **Immediate Write** | Agent writes important info to memory files in the same turn | Real-time, every session |
| **Handoff Detection** | Agent checks other sessions when it detects a channel switch | On demand |

## Installation

Copy into your OpenClaw workspace skills directory:

```bash
git clone https://github.com/rajwaitforit/memory-sync.git ~/.openclaw/workspace/skills/memory-sync
```

Or into managed skills (shared across agents):

```bash
git clone https://github.com/rajwaitforit/memory-sync.git ~/.openclaw/skills/memory-sync
```

## Setup

### 1. Add the sync cron job

Use the OpenClaw CLI or cron tool to add a job:

```bash
openclaw cron add --name memory-sync \
  --schedule "0 */2 * * *" \
  --tz "America/Los_Angeles" \
  --session-target isolated \
  --agent-turn \
  --message "Read /path/to/skills/memory-sync/SKILL.md and run the auto-sync workflow."
```

Or via the gateway cron API — see [OpenClaw cron docs](https://docs.openclaw.ai/automation/cron-jobs) for details.

**Recommended model:** Use a cheap, fast model for the sync job — it's just reading and summarizing. Examples: `google/gemini-2.0-flash`, `openai/gpt-4o-mini`, `anthropic/claude-3-haiku`.

### 2. Update your AGENTS.md

Add these two sections to your workspace `AGENTS.md`:

**Immediate Write Rule:**
```markdown
### ✍️ Immediate Write Rule
When the user tells you anything meaningful — a preference, decision, task, request,
personal info, opinion, or new context — write it to memory RIGHT NOW in the same turn.
Sessions are siloed per channel. If you don't write it down, other sessions won't know.
```

**Channel Handoff Awareness:**
```markdown
### 🔄 Channel Handoff Awareness
When you detect the user is coming from a different channel (or mentions something
you don't have context for), proactively check:
1. memory/YYYY-MM-DD.md (today + yesterday)
2. Recent session histories via sessions_list + sessions_history
```

### 3. Create the memory directory

```bash
mkdir -p ~/.openclaw/workspace/memory
```

## Verify It Works

1. Say something on one channel: *"I'm working on Project Alpha"*
2. Wait for sync (or say *"sync my memory"*)
3. Switch channels and ask: *"What am I working on?"*
4. The agent should know about Project Alpha

## Configuration

### Sync frequency

| Use Case | Schedule | ~Cost/day |
|----------|----------|-----------|
| Heavy multi-channel | Every hour | $0.50 |
| **Default** | **Every 2 hours** | **$0.25** |
| Light use | Every 4 hours | $0.12 |
| Budget | 3x daily | $0.06 |

Costs are estimates based on typical session sizes with a cheap model.

### What gets synced

The sync job extracts:
- Decisions and preferences
- Tasks and action items
- Personal info and context
- Project updates and blockers

It skips: credentials, routine chatter, greetings, duplicate info.

## File Structure

```
memory-sync/
├── SKILL.md           # Skill definition and sync workflows
├── README.md          # This file
├── LICENSE            # MIT
└── references/
    └── setup-guide.md # Troubleshooting and advanced config
```

## Compatibility

- **OpenClaw** 2026.2.x+
- Requires: `sessions_list`, `sessions_history`, `cron` tools
- Works with any LLM provider configured in OpenClaw

## Contributing

PRs welcome. Ideas:
- Smarter deduplication
- Memory file pruning (auto-archive old daily files)
- Conflict resolution when sessions disagree

## License

MIT
