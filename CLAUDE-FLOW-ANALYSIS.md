# Claude-Flow Analysis: Signal vs Noise

## Executive Summary

After deep analysis of claude-flow (7,850 files) and agentic-flow (3,073 files), I've identified that **~95% is bloat**. The core value can be implemented in **~200 lines of shell/JSON**.

---

## What rUv's System ACTUALLY Does

### The Core Mechanism (What Works)

```
Claude Code Hooks → CLI Commands → Simple Storage → Context Loading
```

1. **Claude Code hooks** in `.claude/settings.json` trigger shell commands
2. Those commands write/read from simple file storage
3. On new sessions, past context is loaded into the prompt

### The Real Architecture

```json
// .claude/settings.json - THE ENTIRE ENGINE
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "npx claude-flow hooks post-edit --file '{}' --update-memory true"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "npx claude-flow hooks session-end --generate-summary true"
      }]
    }]
  }
}
```

That's it. Everything else is complexity for complexity's sake.

---

## What's BLOAT (Skip These)

### 1. The 54+ "Agent" Markdown Files
**Location**: `.claude/agents/`
**Reality**: Just prompt templates that never get used. Claude Code's Task tool already does this.

### 2. The 1,270-line Verification System
**Location**: `src/verification/hooks.ts`
**Reality**: 90% is type definitions and placeholder functions. The actual checks are trivial.

### 3. The 758-line Neural Hooks
**Location**: `src/services/agentic-flow-hooks/neural-hooks.ts`
**Reality**: ML-style "pattern detection" and "training" that does nothing real. Most helper functions are placeholders like:
```typescript
async function loadHistoricalPatterns(modelId, context) {
  // Load historical patterns
  // Placeholder implementation
  return [];
}
```

### 4. The 750-line Hook Manager
**Location**: `src/services/agentic-flow-hooks/hook-manager.ts`
**Reality**: Event emitter + priority queue. Could be 50 lines.

### 5. The 3,245-line SwarmCoordinator
**Location**: `src/swarm/coordinator.ts`
**Reality**: Multi-agent coordination that Claude Code's Task tool already does natively.

### 6. SPARC Methodology
**Reality**: Claude Code has built-in planning. This is redundant.

### 7. Hive-Mind Consensus
**Reality**: Over-engineered for any real use case.

### 8. Performance Claims
**Claims**: "84.8% SWE-Bench solve rate", "96x-164x faster"
**Reality**: No benchmarks found. Production readiness docs show "TBD".

---

## What's ACTUALLY VALUABLE (Core 5%)

### 1. Session Lifecycle Hooks
The idea of calling external scripts at key points:
- **Session start**: Load context from previous sessions
- **After each edit/task**: Store what was done, decisions made
- **Session end**: Generate summary, persist learnings

### 2. Simple Persistent Memory
A JSON/SQLite file storing:
- What tasks were completed
- Errors encountered and how they were fixed
- Key decisions and their rationale

### 3. Session Restore
Before starting work, Claude reads recent session summaries to maintain context across sessions.

### 4. Post-Task Audit Pattern
After completing a task, review:
- Did it succeed?
- Were there errors?
- What should be remembered?

---

## Minimal Implementation (What You Actually Need)

### Directory Structure
```
.claude/
├── settings.json           # Claude Code hooks config
├── hooks/
│   ├── session-start.sh    # ~30 lines
│   ├── session-end.sh      # ~40 lines
│   └── post-task.sh        # ~20 lines
└── memory/
    ├── sessions/           # Session summaries
    ├── learnings.json      # Accumulated knowledge
    └── decisions.json      # Key decisions log
```

### 1. settings.json (Minimal Hooks)
```json
{
  "hooks": {
    "PrePrompt": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/session-start.sh"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/session-end.sh"
      }]
    }]
  }
}
```

### 2. session-start.sh (~30 lines)
```bash
#!/bin/bash
# Load recent session context

MEMORY_DIR=".claude/memory"
SESSIONS_DIR="$MEMORY_DIR/sessions"

echo "📚 Loading context from recent sessions..."

# Get last 3 session summaries
for f in $(ls -t "$SESSIONS_DIR"/*.json 2>/dev/null | head -3); do
  echo "---"
  jq -r '.summary // "No summary"' "$f"
done

# Load learnings
if [ -f "$MEMORY_DIR/learnings.json" ]; then
  echo ""
  echo "📖 Past Learnings:"
  jq -r '.[-5:] | .[] | "- \(.lesson)"' "$MEMORY_DIR/learnings.json"
fi
```

### 3. session-end.sh (~50 lines)
```bash
#!/bin/bash
# Persist session state and generate summary

MEMORY_DIR=".claude/memory"
SESSIONS_DIR="$MEMORY_DIR/sessions"
mkdir -p "$SESSIONS_DIR"

TIMESTAMP=$(date +%Y-%m-%d-%H%M%S)
SESSION_FILE="$SESSIONS_DIR/$TIMESTAMP.json"

# Prompt Claude to generate summary (via stdin from Claude Code)
# This receives the conversation context
read -r CONTEXT

# Create session record
cat > "$SESSION_FILE" << EOF
{
  "timestamp": "$TIMESTAMP",
  "summary": "Session completed. Review git log for changes.",
  "files_modified": $(git diff --name-only HEAD~5 2>/dev/null | jq -R -s 'split("\n") | map(select(length > 0))'),
  "duration_minutes": 30
}
EOF

echo "✅ Session saved to $SESSION_FILE"
```

### 4. post-task.sh (~25 lines)
```bash
#!/bin/bash
# Called after completing a task - log what was done

MEMORY_DIR=".claude/memory"
DECISIONS_FILE="$MEMORY_DIR/decisions.json"

TASK_DESC="$1"
OUTCOME="$2"  # success/failure
NOTES="$3"

# Append to decisions log
if [ ! -f "$DECISIONS_FILE" ]; then
  echo "[]" > "$DECISIONS_FILE"
fi

jq --arg task "$TASK_DESC" \
   --arg outcome "$OUTCOME" \
   --arg notes "$NOTES" \
   --arg ts "$(date -Iseconds)" \
   '. += [{"task": $task, "outcome": $outcome, "notes": $notes, "timestamp": $ts}]' \
   "$DECISIONS_FILE" > tmp.$$ && mv tmp.$$ "$DECISIONS_FILE"
```

### 5. Learning from Mistakes (Simple Pattern)

In your CLAUDE.md or session-start hook, include:
```markdown
## When Things Go Wrong

After ANY error:
1. Document what went wrong in .claude/memory/learnings.json
2. Document how it was fixed
3. Check if similar issues exist in past learnings before trying solutions

## Learning Log Format
Add to .claude/memory/learnings.json:
{
  "date": "2024-...",
  "error": "What happened",
  "cause": "Why it happened",
  "fix": "How it was fixed",
  "lesson": "One-line takeaway"
}
```

---

## The Four Core Patterns to Implement

### 1. PM Mode (Review, Don't Code)
Add to CLAUDE.md:
```markdown
## Work Style: PM Mode

For each task:
1. **Specify** - Write a detailed spec before coding
2. **Plan** - Break into subtasks with clear acceptance criteria
3. **Review** - After each subtask, review the changes
4. **Approve/Reject** - Explicitly approve before moving on
5. **Retrospect** - After completion, log what was learned
```

### 2. Learning from Mistakes
```markdown
## Error Handling Protocol

When an error occurs:
1. STOP - Don't immediately retry
2. ANALYZE - What exactly went wrong?
3. CHECK HISTORY - Look at .claude/memory/learnings.json for similar issues
4. FIX - Apply the fix
5. LOG - Add to learnings.json with cause and solution
```

### 3. Self-Audit After Tasks
```markdown
## Post-Task Audit

After completing any task:
1. Run relevant tests: `npm test` or equivalent
2. Check for linting issues: `npm run lint`
3. Review the diff: `git diff`
4. Ask: "Does this actually solve the problem?"
5. Log outcome to .claude/memory/decisions.json
```

### 4. Session Review Before Starting
```markdown
## Session Start Protocol

Before starting any work:
1. Read .claude/memory/sessions/ - last 3 sessions
2. Read .claude/memory/learnings.json - recent lessons
3. Check if current task relates to past work
4. Apply relevant learnings proactively
```

---

## Implementation Recommendation

### Phase 1: Minimal (Day 1)
1. Create `.claude/memory/` directory structure
2. Add simple `session-end.sh` that logs session info
3. Add instruction in CLAUDE.md to check learnings before work

### Phase 2: Hooks (Day 2)
1. Add PrePrompt hook for session-start
2. Add Stop hook for session-end
3. Test the flow

### Phase 3: Refinement (Week 1)
1. Add learnings.json and decisions.json
2. Refine the prompts based on actual usage
3. Add post-task audit prompts

### What NOT to Do
- Don't install claude-flow as a dependency
- Don't use npx for hooks (adds 5-10s latency per call)
- Don't create dozens of agent files
- Don't implement neural training or pattern detection
- Don't implement swarm coordination (use Task tool)

---

## Bottom Line

**Claude-flow's value is the PATTERN, not the implementation.**

The pattern:
1. Hook into Claude Code lifecycle events
2. Persist context to simple files
3. Load past context at session start
4. Log decisions and learnings

The implementation should be:
- ~100 lines of shell scripts
- ~50 lines of JSON config
- ~20 lines of CLAUDE.md instructions

Not:
- 7,850 files
- 50+ TypeScript modules
- Complex dependency chains
- Unverified performance claims

---

## Next Steps

1. I can create the minimal implementation in this repo
2. Or I can extract specific patterns you want to try first
3. Or we can discuss which of the four patterns is most important to you

Which would you prefer?
