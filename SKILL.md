---
name: memory-sync
description: >
  Sync context and memory across all OpenClaw channels (Discord, Telegram, webchat,
  etc.) so nothing gets lost between sessions. Activates on "sync my memory",
  "what did I say on [channel]", "pull in context from other sessions", or when
  the agent detects a channel switch and needs cross-channel context.
---

# Memory Sync

Bridge the gap between siloed channel sessions in OpenClaw.

## Auto-Sync Workflow (for cron job)

When triggered by the sync cron, follow these steps:

### Step 1: Scan sessions

```
Use sessions_list with messageLimit: 5 to get all active sessions.
Filter to sessions updated in the last 4 hours.
Skip cron/isolated sessions — only scan user-facing sessions.
```

### Step 2: Read recent history

For each active session, use `sessions_history` (limit: 20) to read recent messages.

### Step 3: Extract important context

Pull out:
- **Decisions** — "I decided to go with X" / "Let's use Y"
- **Tasks & action items** — "Remind me to..." / "I need to..."
- **Preferences** — "I prefer X" / "Always do Y"
- **Personal info** — Names, dates, plans, events
- **Project context** — What they're working on, blockers, progress

Skip: greetings, filler, routine tool calls, credentials/secrets.

### Step 4: Deduplicate

Read today's `memory/YYYY-MM-DD.md`. Compare against extracted items. Only write genuinely new information.

### Step 5: Write to memory files

Append to `memory/YYYY-MM-DD.md`:

```markdown
## Session Sync — HH:MM

- [channel] extracted item 1
- [channel] extracted item 2
```

For significant long-term info (preferences, personal details, recurring patterns), also update `MEMORY.md`.

## On-Demand Sync

When user says "sync my memory" or asks about context from another channel:

1. Run the full workflow above
2. Report what was found and synced
3. Mention which channels had new context

## Agent Instructions

Add these to your `AGENTS.md` for real-time capture:

### Immediate Write Rule

When the user tells you anything meaningful — a preference, decision, task, request, personal info, opinion, or new context — **write it to memory RIGHT NOW in the same turn**. Do NOT rely on session context. Sessions are siloed per channel. If you don't write it down, other sessions will never know.

### Channel Handoff Awareness

When you detect the user is coming from a different channel than last time (or mentions something you don't have context for), proactively check:

1. `memory/YYYY-MM-DD.md` (today + yesterday)
2. Recent session histories via `sessions_list` + `sessions_history`

This catches context that may have come from another channel.
