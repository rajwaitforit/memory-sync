# 🧠 Memory Sync

**An OpenClaw skill that syncs context across all your channels — so your AI never forgets what you said on Discord when you switch to Telegram.**

## The Problem

Every channel in OpenClaw gets its own session. Tell your bot something on Discord, switch to Telegram, and it's gone. Decisions, preferences, tasks, personal context — all siloed. You end up repeating yourself across channels like it's 2015.

## The Solution

Memory Sync bridges the gap with shared memory files that every session can read. Two mechanisms work together:

1. **🔄 Auto Sync** — A cron job runs every 2 hours on a cheap model, scans all active sessions, extracts important context, and writes it to shared memory files
2. **⚡ On-Demand Sync** — Say "sync my memory" for instant cross-channel context sync
3. **🔍 Channel Handoff Detection** — When the agent detects you've switched channels, it proactively checks for context from your other sessions

## How It Works

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Discord  │    │ Telegram │    │ Webchat  │
│ Session  │    │ Session  │    │ Session  │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     └───────┬───────┴───────┬───────┘
             │               │
             ▼               ▼
    ┌─────────────┐  ┌──────────────┐
    │  MEMORY.md  │  │ memory/      │
    │ (long-term) │  │ YYYY-MM-DD.md│
    │             │  │ (daily logs) │
    └─────────────┘  └──────────────┘
             │               │
     ┌───────┴───────┬───────┴───────┐
     │               │               │
     ▼               ▼               ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Discord  │    │ Telegram │    │ Webchat  │
│ (reads)  │    │ (reads)  │    │ (reads)  │
└──────────┘    └──────────┘    └──────────┘
```

**The cron job** uses `sessions_list` and `sessions_history` to scan all active sessions, extracts decisions, tasks, preferences, and personal info, then writes consolidated context to daily memory files.

**The Immediate Write Rule** ensures important info is captured at the source — the agent writes to memory in the same turn, not just during periodic sync.

**Channel Handoff Detection** means when you switch channels, the agent proactively checks recent memory files to pull in context from wherever you were before.

## Features

| Feature | Description |
|---------|-------------|
| 🔄 Auto sync | Cron job every 2h on a cheap model |
| ⚡ On-demand | "Sync my memory" for instant sync |
| 🔍 Handoff detection | Proactive context loading on channel switch |
| ✍️ Immediate writes | Critical info saved same-turn, not deferred |
| 🧹 Deduplication | Won't repeat context already in memory files |
| 🔒 Local-only | All data stays on your machine |
| 💰 Token-efficient | Uses cheap models for background sync |

## Installation

### Option 1: OpenClaw CLI

```bash
openclaw skill install memory-sync
```

### Option 2: Manual

```bash
# Clone into your skills directory
git clone https://github.com/rajwaitforit/memory-sync.git ~/.openclaw/workspace/skills/memory-sync
```

### Setup

**1. Add the cron job** to your OpenClaw config:

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

**2. Add to your AGENTS.md** — Copy the Immediate Write Rule and Channel Handoff Awareness sections from `SKILL.md` into your workspace's `AGENTS.md`.

**3. Create the memory directory:**

```bash
mkdir -p ~/.openclaw/workspace/memory
```

That's it. The skill activates automatically when loaded.

## Usage

The skill works in two ways:

### Passive (Automatic)

Once configured, the cron job runs every 2 hours. No interaction needed. Context flows between channels automatically.

### Active (On-Demand)

Just say:

- *"Sync my memory"*
- *"What did I say on Discord?"*
- *"Pull in context from my other sessions"*

The agent runs the full sync workflow immediately and reports what it found.

## Configuration

### Sync Frequency

| Use Case | Schedule | Token Cost |
|----------|----------|------------|
| Heavy multi-channel | `0 * * * *` (hourly) | ~$0.02/run |
| Default | `0 */2 * * *` (every 2h) | ~$0.02/run |
| Light use | `0 */4 * * *` (every 4h) | ~$0.02/run |
| Budget | `0 8,14,20 * * *` (3x daily) | ~$0.06/day |

### Recommended Models for Sync

The sync job is read-heavy extraction — no complex reasoning needed:

- `google/gemini-2.0-flash-001` — Fast, cheap, recommended
- `openai/gpt-4o-mini` — Solid alternative
- `anthropic/claude-3-haiku` — Good at summarization

## Skill Structure

```
memory-sync/
├── SKILL.md                     # Core skill logic and workflows
├── LICENSE                      # MIT
└── references/
    └── setup-guide.md           # Detailed setup and troubleshooting
```

## How We Built This

We identified the cross-channel memory problem while using OpenClaw across Discord, Telegram, and webchat simultaneously. The agent kept "forgetting" things said on other channels because each channel has its own isolated session.

The solution was surprisingly simple: **shared files bridge isolated sessions.** Every session already reads `AGENTS.md` and memory files on startup. By writing important context to those files, we create a shared memory layer that transcends channel boundaries.

The cron job adds a safety net — even if the agent forgets to write something immediately, the periodic scan catches it. Together, the immediate write rule and periodic sync ensure nothing falls through the cracks.

## Contributing

Ideas for improvement? PRs welcome:

- Better extraction heuristics
- Smarter deduplication
- Memory file pruning strategies
- New trigger phrases

## License

MIT
