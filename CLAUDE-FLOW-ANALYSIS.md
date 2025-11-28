# Claude-Flow & Agentic-Flow Analysis: Signal vs Noise

## Executive Summary

After deep analysis of claude-flow and agentic-flow (combined ~10,000 files), the picture is more nuanced than initially thought:

**~80% is bloat** (agents, proxies, swarm systems, consensus protocols)
**~20% is real substance** - specifically **AgentDB**, which has genuine learning capabilities

The shell hooks in settings.json are thin wrappers. The **real value is AgentDB**: an MCP server with episodic memory, skill consolidation, and causal discovery based on academic research (Reflexion paper, Voyager paper, doubly robust causal estimation).

---

## CORRECTION: AgentDB is NOT Bloat

My initial analysis dismissed AgentDB - that was wrong. Here's what it actually provides:

### AgentDB Core Components (Real Value)

**Location**: `packages/agentdb/src/controllers/`

#### 1. ReflexionMemory (`ReflexionMemory.ts`)
Based on the [Reflexion paper](https://arxiv.org/abs/2303.11366) - verbal reinforcement learning.
```typescript
// Stores episodes with self-critique
storeEpisode({
  sessionId: string,
  task: string,
  input: string,
  output: string,
  critique: string,    // Self-reflection on what worked/failed
  reward: number,      // 0-1 success signal
  success: boolean,
  latencyMs: number,
  tokensUsed: number
})

// Retrieves similar past episodes using embeddings
retrieveRelevant(query: string, k: number) → Episode[]
```

#### 2. SkillLibrary (`SkillLibrary.ts`)
Based on the [Voyager paper](https://arxiv.org/abs/2305.16291) - lifelong learning agents.
```typescript
// Auto-consolidates successful patterns into reusable skills
consolidateEpisodesIntoSkills({
  minAttempts: 3,      // Minimum observations
  minReward: 0.8,      // Success threshold
  extractPatterns: true
})

// Returns: patterns like "When debugging TypeScript, check tsconfig.json first"
```

#### 3. NightlyLearner (`NightlyLearner.ts`)
Automated causal discovery using **doubly robust estimation**:
```
τ̂(x) = μ1(x) − μ0(x) + [a*(y−μ1(x)) / e(x)] − [(1−a)*(y−μ0(x)) / (1−e(x))]
```

This discovers which actions causally improve outcomes, not just correlate with them.

```typescript
// Runs nightly to:
// 1. Discover causal edges from episode patterns
// 2. Complete A/B experiments
// 3. Prune low-confidence edges
// 4. Create new experiments for promising hypotheses
run(): Promise<LearnerReport>
```

#### 4. EmbeddingService (`EmbeddingService.ts`)
Local embeddings using `Xenova/all-MiniLM-L6-v2` (384 dimensions). No API calls needed.

### AgentDB MCP Server (29 Tools)

**Location**: `packages/agentdb/src/mcp/agentdb-mcp-server.ts`

Key tools for your use cases:

| Tool | Purpose |
|------|---------|
| `reflexion_store` | Store episode with self-critique |
| `reflexion_retrieve` | Find similar past episodes |
| `experience_record` | Log tool execution for RL |
| `learning_feedback` | Submit reward signal |
| `learning_train` | Batch train on experiences |
| `learning_explain` | Get explainable AI recommendations |
| `skill_search` | Find relevant learned skills |
| `learner_discover` | Run causal discovery |

### How It Connects to Claude Code

**settings.json hooks** call `npx claude-flow@alpha hooks <command>`, but the real work happens when Claude calls MCP tools directly:

```
Claude Code Session Start
    ↓
MCP: reflexion_retrieve(query="current task context")
    ↓
Returns: Similar past episodes with lessons learned
    ↓
During Task: MCP: experience_record(action, state, reward)
    ↓
Task Completion
    ↓
MCP: reflexion_store(episode with critique)
MCP: learning_feedback(reward signal)
    ↓
Nightly: NightlyLearner.run() → causal discovery
    ↓
SkillLibrary.consolidateEpisodesIntoSkills()
```

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

## What's BLOAT (Skip These - 80% of the codebase)

### 1. The 66+ "Agent" Markdown Files
**Location**: `.claude/agents/`
**Reality**: Just prompt templates. Claude Code's Task tool already provides subagents.

### 2. The 213+ "MCP Tools" Claim
**Reality**: Most are duplicates or thin wrappers. AgentDB's 29 tools are the only real ones.

### 3. The Proxy Servers
**Location**: `src/proxy/anthropic-to-openrouter.ts`, `anthropic-to-gemini.ts`, `anthropic-to-onnx.ts`
**Reality**: Useful if you need OpenRouter/Gemini, but not related to learning.

### 4. The Swarm/Federation Systems
**Location**: `src/swarm/`, `src/federation/`
**Reality**: Over-engineered. Claude Code's Task tool already spawns subagents.

### 5. QUIC Transport
**Location**: `src/transport/quic.ts`
**Reality**: Low-latency networking for distributed agents. Overkill for most use cases.

### 6. SPARC Methodology
**Reality**: Claude Code has built-in planning. This is redundant documentation.

### 7. Byzantine Consensus / Hive-Mind
**Reality**: Academic exercise, not practical for single-user Claude Code.

### 8. Performance Claims
**Claims**: "84.8% SWE-Bench solve rate", "96x-164x faster"
**Reality**: No benchmarks found. Production readiness docs show "TBD".

---

## What's ACTUALLY VALUABLE (Core 20%)

### 1. AgentDB Package (The Real Substance)
**Install**: `npm install agentdb`
**Why it matters**: Real learning algorithms, not just file storage.

| Component | What It Does | Your Use Case |
|-----------|--------------|---------------|
| ReflexionMemory | Stores episodes with self-critique | Learning from mistakes |
| SkillLibrary | Extracts patterns from successes | Self-auditing |
| NightlyLearner | Discovers causal relationships | Understanding what works |
| EmbeddingService | Semantic similarity search | Session review with context |

### 2. MCP Server Integration
AgentDB runs as an MCP server that Claude Desktop/Code can call:
```json
// claude_desktop_config.json
{
  "mcpServers": {
    "agentdb": {
      "command": "npx",
      "args": ["agentdb", "mcp-server", "--db", "./memory.db"]
    }
  }
}
```

### 3. ReasoningBank Hooks
**Location**: `agentic-flow/src/reasoningbank/hooks/`

**pre-task.ts**: Retrieves and injects relevant memories before task execution
**post-task.ts**: Judges trajectory, distills memories, runs consolidation

### 4. The Learning Loop (End-to-End)
```
Session Start
    ↓
pre-task hook: retrieveMemories(query) → inject into system prompt
    ↓
During Task: experience_record() logs tool calls
    ↓
Task Complete
    ↓
post-task hook:
  1. judgeTrajectory() → verdict (correct/incorrect)
  2. distillMemories() → extract lessons
  3. shouldConsolidate() → prune duplicates/contradictions
    ↓
Nightly: NightlyLearner.run()
  1. discoverCausalEdges() → find what actions help
  2. completeExperiments() → A/B test results
  3. pruneEdges() → remove weak patterns
  4. consolidateEpisodesIntoSkills() → extract reusable knowledge
```

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

## Implementation Recommendation: Two Paths

### Path A: Minimal Shell Scripts (Simple, No Dependencies)

If you want the simplest possible solution with no npm dependencies:

**Phase 1**: Minimal hooks (~100 lines shell)
- Create `.claude/memory/` directory
- Shell scripts for session start/end
- JSON files for learnings/decisions

**Pros**: Zero dependencies, fast hooks, easy to understand
**Cons**: No semantic search, no causal discovery, manual pattern extraction

### Path B: AgentDB Integration (Real Learning)

If you want actual machine learning and pattern discovery:

**Phase 1**: Install AgentDB
```bash
npm install agentdb
```

**Phase 2**: Configure MCP Server
```json
// claude_desktop_config.json
{
  "mcpServers": {
    "agentdb": {
      "command": "npx",
      "args": ["agentdb", "mcp-server"]
    }
  }
}
```

**Phase 3**: Add CLAUDE.md instructions
```markdown
## Memory Protocol

When starting a session:
1. Use reflexion_retrieve to find similar past episodes
2. Review any relevant skills from skill_search

After completing a task:
1. Use reflexion_store to save the episode with critique
2. Use learning_feedback to log success/failure

If you encounter an error you've seen before:
1. Check reflexion_retrieve for past solutions
2. Apply learned patterns
```

**Pros**: Real embeddings, causal discovery, automated pattern extraction
**Cons**: npm dependency, SQLite database, more complex

### What NOT to Do (Either Path)
- Don't install full `agentic-flow` (too much bloat)
- Don't use the 66+ agent markdown files
- Don't implement swarm/federation (use Claude Code's Task tool)
- Don't use the proxy servers unless you need OpenRouter/Gemini

---

## Bottom Line

**The value is in TWO things:**

1. **The Pattern**: Hook into Claude Code lifecycle to persist and retrieve context
2. **AgentDB**: Real learning algorithms (ReflexionMemory, SkillLibrary, NightlyLearner)

**The bloat** (skip these):
- 66+ agent markdown files
- 213+ claimed MCP tools (only ~29 real ones in AgentDB)
- Swarm/federation/consensus systems
- QUIC transport
- Multiple proxy servers

**For your specific needs:**
- Learning from mistakes → `ReflexionMemory.storeEpisode()` with critique
- Self-auditing → `SkillLibrary.consolidateEpisodesIntoSkills()`
- Session review → `reflexion_retrieve()` at session start
- PM not coder → CLAUDE.md instructions (no code needed)

---

## Next Steps

1. **Try AgentDB standalone**: `npm install agentdb && npx agentdb mcp-server`
2. **Or build minimal shell version**: I can create the ~100 line implementation
3. **Or extract specific components**: Just the parts you need from AgentDB

Which approach fits your workflow better?
