---
name: memory-sync
description: >
  Sync context and memory across all OpenClaw channels (Discord, Telegram, webchat,
  SMS, etc.) so nothing gets lost between sessions. Use when a user asks to "sync
  memory", "sync context", "what did I say on Discord/Telegram", "remember across
  channels", or when the agent detects a channel switch and needs to pull in context
  from other sessions. Also provides a cron-based auto-sync that runs periodically
  to keep shared memory files updated with important info from all active sessions.
---

# Memory Sync

Bridge the gap between siloed channel sessions. Every channel gets its own session in
OpenClaw, which means context said on Discord doesn't automatically appear in Telegram.
This skill fixes that.

## How It Works

Two complementary mechanisms:

### 1. Auto Sync (Cron Job)

A periodic cron job scans all active sessions, extracts important information, and
writes it to shared memory files that every session can read.

**Setup:** Add this cron entry to your OpenClaw config:

```yaml
cron:
  - label: memory-sync
    schedule: "0 */2 * * *"          # Every 2 hours
    prompt: |
      Read the memory-sync skill. Run the auto-sync workflow:
      1. List all active sessions via sessions_list
      2. For each session with recent activity, read history via sessions_history
      3. Extract: decisions, tasks, preferences, personal info, action items
      4. Write consolidated context to memory/YYYY-MM-DD.md
      5. If significant long-term info found, update MEMORY.md
    sessionTarget: isolated
    agentTurn: true
    model: google/gemini-2.0-flash-001  # Cheap model for routine sync
```

### 2. On-Demand Sync

User says "sync my memory" or similar → run the sync workflow immediately.

### 3. Channel Handoff Detection

When the agent detects context that doesn't match the current channel (user references
something not discussed here), proactively check recent memory files and other session
histories.

## Auto-Sync Workflow

### Step 1: Scan Sessions

```
Use sessions_list to get all active sessions.
Filter to sessions with activity in the last 4 hours.
```

### Step 2: Extract Context

For each active session, use `sessions_history` to read recent messages. Extract:

- **Decisions made** — "I decided to go with React" / "Let's use PostgreSQL"
- **Tasks & action items** — "Remind me to..." / "I need to..."
- **Preferences stated** — "I prefer dark mode" / "Always use TypeScript"
- **Personal info shared** — Names, dates, plans, travel
- **Project context** — What they're working on, blockers, progress
- **Emotional context** — Frustrated with X, excited about Y

### Step 3: Deduplicate

Compare extracted info against existing `memory/YYYY-MM-DD.md`. Skip duplicates.
Only append genuinely new information.

### Step 4: Write to Memory Files

Write new context to `memory/YYYY-MM-DD.md` in this format:

```markdown
## Channel Sync — [HH:MM]

**Source:** [channel name] session
**Key context:**
- [extracted item 1]
- [extracted item 2]

**Active tasks:**
- [ ] [task from this session]
```

For significant long-term info (preferences, personal details, recurring patterns),
also update `MEMORY.md`.

## On-Demand Sync Workflow

When user says "sync my memory" or similar:

1. Run the full auto-sync workflow above immediately
2. Report what was found and synced
3. Mention which channels had new context

## Channel Handoff Detection

Add this behavior to your AGENTS.md:

```markdown
### 🔄 Channel Handoff Awareness
When you detect the user is coming from a different channel than last time
(or mentions something you don't have context for), proactively:
1. Read memory/YYYY-MM-DD.md (today + yesterday)
2. Check recent session histories via sessions_list + sessions_history
This catches context from other channels automatically.
```

## Immediate Write Rule

Add this to your AGENTS.md to complement the sync:

```markdown
### ✍️ Immediate Write Rule (CRITICAL)
When the user tells you ANYTHING meaningful — a preference, decision, task,
request, personal info, opinion, or new context — write it to memory RIGHT NOW
in the same turn. Do NOT rely on session context alone. Sessions are siloed
per channel. If you don't write it down, other sessions will never know.
```

This ensures important info is captured at the source, not just during periodic sync.

## Memory File Structure

```
workspace/
├── MEMORY.md                    # Long-term curated memory
└── memory/
    ├── 2026-02-15.md           # Yesterday's context
    ├── 2026-02-16.md           # Today's context
    └── heartbeat-state.json    # Sync tracking
```

## Best Practices

- **Cheap model for cron:** Use a fast/cheap model (Gemini Flash, GPT-4o-mini) for
  the periodic sync — it's just reading and summarizing, no complex reasoning needed
- **Isolated sessions:** Always use `sessionTarget: isolated` for the cron job so it
  doesn't pollute the main conversation
- **Don't over-sync:** The cron job deduplicates. Running more frequently than every
  2 hours rarely helps and wastes tokens
- **Privacy:** Memory files live in the workspace. They're local to the user's machine.
  No data leaves unless the user explicitly shares
- **Pruning:** Periodically clean old daily memory files (>30 days) to keep the
  workspace tidy
