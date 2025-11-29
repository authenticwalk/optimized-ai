# ruvnet Systems Analysis: claude-flow & agentic-flow

**Date**: 2025-11-29
**Purpose**: Deep analysis of rUv's agent orchestration systems to identify core innovations

---

## Executive Summary

After comprehensive analysis of both repositories, here's the reality:

**Why rUv Gets Good Results**:
1. **Hooks-based automatic learning** - The real magic: transparent memory injection without agent code changes
2. **Bayesian confidence updates** - Simple but effective pattern learning
3. **SPARC methodology** - Structured phases with review gates
4. **Pre/Post operation auditing** - Catches errors before they compound
5. **Failure-weighted learning** - 40% of training data is failures

**What's Core vs Marketing**:
- Core: Hooks + SQLite + Bayesian updates = self-learning loop
- Marketing: "64 agents", "96x faster", "84.8% SWE-Bench" (unverified)

---

## System Comparison

### claude-flow vs agentic-flow

| Aspect | claude-flow | agentic-flow |
|--------|-------------|--------------|
| **Focus** | Agent orchestration | Performance + deployment |
| **Memory** | ReasoningBank (SQLite) | AgentDB (SQLite + HNSW) |
| **Transport** | MCP + stdio | MCP + QUIC |
| **Agents** | 64 specialized agents | Ephemeral federation |
| **Learning** | Post-task hooks | Swarm learning optimizer |
| **Deployment** | npm package | Docker + K8s |

**Key Difference**: claude-flow is the **orchestration layer**, agentic-flow is the **enterprise deployment layer** with performance optimizations.

---

## The Core Innovation Stack (What Actually Works)

### 1. Hook-Based Transparent Learning

**The key insight**: Inject learning BEFORE Claude executes, store outcomes AFTER.

```
┌─────────────────┐
│  Claude Code    │
│  receives task  │
└────────┬────────┘
         │
    ┌────▼─────┐
    │ pre-task │──→ Query patterns from SQLite
    │   hook   │
    └────┬─────┘
         │ Inject: "Past learnings for this type of task..."
    ┌────▼─────┐
    │  Claude  │
    │ executes │
    └────┬─────┘
         │
    ┌────▼──────┐
    │ post-task │──→ Store outcome + update confidence
    │   hook    │
    └───────────┘
```

**Why this works**: Zero code changes to agents. Memory "just happens."

### 2. Bayesian Confidence Updates

```python
def update_confidence(prior, outcome):
    if outcome == 'success':
        return prior + (1 - prior) * 0.1  # Increases toward 1
    else:
        return prior - prior * 0.15       # Decreases proportionally
```

After 10 successes: 0.5 → 0.87 (high confidence)
After 1 failure: 0.87 → 0.74 (drops but doesn't reset)

**Why this works**: Simple, interpretable, converges to empirical success rate.

### 3. SPARC Phased Development

```
Specification → Pseudocode → Architecture → Refinement → Completion
     ↓              ↓            ↓             ↓            ↓
 Requirements   Algorithms   System Design   TDD + Code   Deploy
```

**The PM-style benefit**: You review after each phase, not at the end.

### 4. Failure-Weighted Learning

40% of stored patterns are FAILURES. This prevents repeating mistakes:

```sql
-- Before risky operation
SELECT error_msg, COUNT(*) FROM failures
WHERE context = 'database_migration'
GROUP BY error_msg ORDER BY COUNT(*) DESC;
```

**Why this works**: Learn what NOT to do, not just what to do.

### 5. Session Audit + Self-Review

At session end:
1. Count successes/failures
2. Extract new patterns
3. Consolidate duplicates
4. Generate summary
5. Store for next session

---

## Why He Gets Good Results

### Core Reasons

1. **Structured workflow (SPARC)**: Forces planning before coding
2. **Automatic learning loop**: Every task improves future performance
3. **Pre-execution context**: Claude knows what worked before
4. **Post-execution audit**: Catches and stores patterns
5. **Session continuity**: Next session remembers this one

### Likely Amplifiers

1. **Prompt quality**: His CLAUDE.md files are well-structured
2. **Agent specialization**: Right agent for right task
3. **Iteration velocity**: Multiple attempts with learning between them
4. **Domain expertise**: Demos use his familiar domains

---

## Minimal System Proposal

### What You Actually Need (Minimum Viable)

**Files Required**:
```
.claude/
├── settings.json          # Enable hooks
├── hooks/
│   ├── pre-task.sh       # Inject patterns before task
│   ├── post-task.sh      # Store outcomes after task
│   ├── session-start.sh  # Restore context
│   └── session-end.sh    # Generate summary
└── claude.md             # PM-style directives
.swarm/
└── memory.db             # SQLite for pattern storage
```

### Hook Implementations (Simplified)

**pre-task.sh** (~30 lines):
```bash
#!/bin/bash
TASK="$1"
PATTERNS=$(sqlite3 .swarm/memory.db "SELECT pattern, confidence FROM patterns WHERE context LIKE '%${TASK}%' AND confidence > 0.7 ORDER BY confidence DESC LIMIT 5")
if [ -n "$PATTERNS" ]; then
  echo "LEARNED PATTERNS:"
  echo "$PATTERNS"
fi
```

**post-task.sh** (~40 lines):
```bash
#!/bin/bash
TASK="$1"
OUTCOME="$2"
CONF=$([ "$OUTCOME" = "success" ] && echo "0.6" || echo "0.3")
sqlite3 .swarm/memory.db "INSERT INTO patterns (pattern, context, confidence, outcome) VALUES ('$TASK', '$(pwd)', $CONF, '$OUTCOME') ON CONFLICT(pattern) DO UPDATE SET confidence = confidence * 0.9 + 0.1 * (CASE WHEN '$OUTCOME'='success' THEN 1 ELSE 0 END), occurrence_count = occurrence_count + 1"
```

### SQLite Schema (~20 lines)

```sql
CREATE TABLE patterns (
    pattern TEXT PRIMARY KEY,
    context TEXT,
    confidence REAL DEFAULT 0.5,
    outcome TEXT,
    occurrence_count INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE failures (
    id INTEGER PRIMARY KEY,
    pattern TEXT,
    error_msg TEXT,
    context TEXT,
    occurred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE sessions (
    id INTEGER PRIMARY KEY,
    summary TEXT,
    successes INTEGER,
    failures INTEGER,
    patterns_learned INTEGER,
    ended_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### CLAUDE.md Directives (~50 lines)

```markdown
# Development Protocol

## Before Complex Tasks
1. The system will automatically inject past learnings (pre-task hook)
2. Review injected patterns before proceeding
3. Prefer proven approaches over novel ones

## After Task Completion
1. The system will automatically store outcomes (post-task hook)
2. For failures, capture the specific error and context
3. For successes, note what made it work

## Self-Review Triggers
- Before major refactors: Check past refactor patterns
- Before deployments: Review past deployment issues
- When stuck: Query failure patterns for this context

## Quality Gates
- No commit without tests (use pre-commit hook)
- Tests must actually pass (verify output, don't trust AI claims)
- Security audit on auth/data changes
```

---

## What to Skip

### Over-Engineered Features

1. **64-agent system** - Start with 3-5: planner, coder, reviewer, tester
2. **Hive-mind consensus** - Overkill for single-user
3. **QUIC transport** - HTTP is fine for local
4. **Kubernetes controller** - Docker-compose is enough
5. **SPARC full 5-phase** - SPARC Lite (2-phase) covers 90% of cases

### Unverified Claims

1. "96x-164x faster" - Reasonable for SQLite vs network, but benchmark yourself
2. "84.8% SWE-Bench" - No evidence in codebase
3. "352x faster" - Agent Booster claims need verification

---

## Implementation Priority

### Week 1: Core Learning Loop
1. Create `.swarm/memory.db` with minimal schema
2. Write `pre-task.sh` and `post-task.sh` hooks
3. Enable hooks in `.claude/settings.json`
4. Test: Do patterns get stored and retrieved?

### Week 2: Session Continuity
1. Add `session-start.sh` (restore context)
2. Add `session-end.sh` (generate summary)
3. Add CLAUDE.md with PM-style directives
4. Test: Does next session benefit from previous?

### Week 3: Audit + Safety
1. Add `pre-command.sh` for safety checks
2. Add `post-edit.sh` for test running
3. Implement failure pattern storage
4. Test: Are failures being learned from?

### Week 4: Refinement
1. Add Bayesian confidence updates
2. Implement pattern consolidation
3. Add causal linking (optional)
4. Measure: Is task success rate improving?

---

## Key Takeaways

### The Real Innovation
The power isn't in the complexity - it's in the simplicity of the core loop:
1. **Before task**: Inject what worked before
2. **After task**: Store what happened
3. **Over time**: Confidence converges to empirical success rate

### PM-Style Development
You don't need 64 agents. You need:
1. Clear phases with review gates
2. Automatic learning from outcomes
3. Session summaries for continuity
4. Safety hooks to catch errors early

### Minimum Viable Self-Learning
- SQLite database (single file, zero config)
- 4 bash hooks (~120 lines total)
- CLAUDE.md directives (~50 lines)
- That's it. The rest is optimization.

---

## Sources

- [claude-flow Repository](https://github.com/ruvnet/claude-flow)
- [agentic-flow Repository](https://github.com/ruvnet/agentic-flow)
- [claude-flow Wiki](https://github.com/ruvnet/claude-flow/wiki)
- Existing research in `.ai-knowledge/research/`
