# Minimal Self-Learning System

**Based on ruvnet's claude-flow/agentic-flow analysis**
**Goal**: PM-style development with automatic learning

---

## What This Gives You

1. **Code as PM, not coder**: Review at phase gates, not in the weeds
2. **Learn from mistakes**: Failures stored and surfaced before repeating
3. **Self-auditing**: Each task/session reviewed and scored
4. **Session continuity**: Next session knows what previous did
5. **Safety hooks**: Dangerous commands blocked, tests enforced

---

## File Structure (Minimal Install)

```
your-project/
├── .claude/
│   ├── settings.json         # Enable hooks
│   ├── claude.md             # PM directives
│   └── hooks/
│       ├── pre-task.sh       # Inject patterns
│       ├── post-task.sh      # Store outcomes
│       ├── pre-command.sh    # Safety checks
│       ├── session-start.sh  # Restore context
│       └── session-end.sh    # Generate summary
└── .swarm/
    └── memory.db             # SQLite pattern storage
```

**Total: 6 files, ~300 lines**

---

## Implementation

### 1. settings.json

```json
{
  "hooks": {
    "enabled": true,
    "pre-task": ".claude/hooks/pre-task.sh",
    "post-task": ".claude/hooks/post-task.sh",
    "pre-command": ".claude/hooks/pre-command.sh",
    "session-start": ".claude/hooks/session-start.sh",
    "session-end": ".claude/hooks/session-end.sh"
  },
  "memory": {
    "database": ".swarm/memory.db",
    "confidence_threshold": 0.7,
    "max_patterns": 5
  }
}
```

### 2. Database Setup (run once)

```bash
mkdir -p .swarm
sqlite3 .swarm/memory.db << 'EOF'
CREATE TABLE IF NOT EXISTS patterns (
    pattern TEXT PRIMARY KEY,
    context TEXT,
    confidence REAL DEFAULT 0.5,
    outcome TEXT,
    occurrence_count INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS failures (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task TEXT,
    error_msg TEXT,
    context TEXT,
    occurred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS sessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    summary TEXT,
    successes INTEGER DEFAULT 0,
    failures INTEGER DEFAULT 0,
    patterns_learned INTEGER DEFAULT 0,
    ended_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_patterns_context ON patterns(context);
CREATE INDEX IF NOT EXISTS idx_patterns_confidence ON patterns(confidence DESC);
CREATE INDEX IF NOT EXISTS idx_failures_task ON failures(task);
EOF
```

### 3. pre-task.sh

```bash
#!/bin/bash
# Inject learned patterns before task execution

TASK="$1"
DB=".swarm/memory.db"

if [ ! -f "$DB" ]; then
    exit 0
fi

# Sanitize input
SAFE_TASK=$(echo "$TASK" | sed "s/'/''/g")

# Query similar patterns
PATTERNS=$(sqlite3 "$DB" << EOF
SELECT pattern, ROUND(confidence, 2) as conf
FROM patterns
WHERE (context LIKE '%${SAFE_TASK}%' OR pattern LIKE '%${SAFE_TASK}%')
  AND confidence > 0.6
ORDER BY confidence DESC
LIMIT 5;
EOF
)

if [ -n "$PATTERNS" ]; then
    echo "LEARNED PATTERNS for similar tasks:"
    echo "$PATTERNS" | while IFS='|' read -r pattern conf; do
        echo "  - $pattern (confidence: $conf)"
    done
    echo ""
fi

# Check for past failures on similar tasks
FAILURES=$(sqlite3 "$DB" << EOF
SELECT error_msg
FROM failures
WHERE task LIKE '%${SAFE_TASK}%'
ORDER BY occurred_at DESC
LIMIT 3;
EOF
)

if [ -n "$FAILURES" ]; then
    echo "PAST FAILURES to avoid:"
    echo "$FAILURES" | while read -r err; do
        echo "  - $err"
    done
    echo ""
fi
```

### 4. post-task.sh

```bash
#!/bin/bash
# Store task outcomes and update patterns

TASK="$1"
OUTCOME="$2"  # success or failure
DURATION="$3"  # optional
ERROR_MSG="$4" # optional, for failures

DB=".swarm/memory.db"

if [ ! -f "$DB" ]; then
    exit 0
fi

SAFE_TASK=$(echo "$TASK" | sed "s/'/''/g")
SAFE_ERROR=$(echo "$ERROR_MSG" | sed "s/'/''/g")
CONTEXT=$(pwd)

# Bayesian confidence update
if [ "$OUTCOME" = "success" ]; then
    # Increase confidence: prior + (1 - prior) * 0.1
    sqlite3 "$DB" << EOF
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES ('$SAFE_TASK', '$CONTEXT', 0.6, 'success')
ON CONFLICT(pattern) DO UPDATE SET
    confidence = confidence + (1 - confidence) * 0.1,
    occurrence_count = occurrence_count + 1,
    last_seen = datetime('now'),
    outcome = 'success';
EOF
    echo "Pattern stored: SUCCESS (confidence updated)"
else
    # Decrease confidence: prior - prior * 0.15
    sqlite3 "$DB" << EOF
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES ('$SAFE_TASK', '$CONTEXT', 0.35, 'failure')
ON CONFLICT(pattern) DO UPDATE SET
    confidence = confidence - confidence * 0.15,
    occurrence_count = occurrence_count + 1,
    last_seen = datetime('now'),
    outcome = 'failure';

INSERT INTO failures (task, error_msg, context)
VALUES ('$SAFE_TASK', '$SAFE_ERROR', '$CONTEXT');
EOF
    echo "Pattern stored: FAILURE (confidence decreased)"
fi
```

### 5. pre-command.sh

```bash
#!/bin/bash
# Safety checks before shell command execution

COMMAND="$1"

# Dangerous command patterns
DANGEROUS_PATTERNS=(
    "rm -rf /"
    "rm -rf ~"
    "rm -rf *"
    "> /dev/"
    "dd if="
    "chmod -R 777"
    "chmod 777 /"
    ":(){ :|:& };"
    "mkfs."
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
    if [[ "$COMMAND" == *"$pattern"* ]]; then
        echo "BLOCKED: Dangerous command pattern detected: $pattern"
        echo "This command will not be executed."
        exit 1
    fi
done

# Check for commands that failed repeatedly
DB=".swarm/memory.db"
if [ -f "$DB" ]; then
    SAFE_CMD=$(echo "$COMMAND" | sed "s/'/''/g" | cut -c1-100)
    FAILURE_COUNT=$(sqlite3 "$DB" "SELECT COUNT(*) FROM failures WHERE task LIKE '%$SAFE_CMD%'")

    if [ "$FAILURE_COUNT" -gt 3 ]; then
        echo "WARNING: This command has failed $FAILURE_COUNT times previously."
        echo "Consider an alternative approach."
    fi
fi
```

### 6. session-start.sh

```bash
#!/bin/bash
# Restore context from previous session

DB=".swarm/memory.db"

if [ ! -f "$DB" ]; then
    exit 0
fi

# Get last session summary
LAST_SESSION=$(sqlite3 "$DB" << 'EOF'
SELECT summary FROM sessions ORDER BY ended_at DESC LIMIT 1;
EOF
)

if [ -n "$LAST_SESSION" ]; then
    echo "PREVIOUS SESSION SUMMARY:"
    echo "$LAST_SESSION"
    echo ""
fi

# Get high-priority pending patterns (low confidence, needs work)
PENDING=$(sqlite3 "$DB" << 'EOF'
SELECT pattern, ROUND(confidence, 2)
FROM patterns
WHERE confidence < 0.5 AND occurrence_count > 1
ORDER BY last_seen DESC
LIMIT 3;
EOF
)

if [ -n "$PENDING" ]; then
    echo "AREAS NEEDING IMPROVEMENT:"
    echo "$PENDING" | while IFS='|' read -r pattern conf; do
        echo "  - $pattern (current confidence: $conf)"
    done
    echo ""
fi

# Get top successes for this project context
SUCCESSES=$(sqlite3 "$DB" << EOF
SELECT pattern, ROUND(confidence, 2)
FROM patterns
WHERE confidence > 0.8
ORDER BY confidence DESC
LIMIT 5;
EOF
)

if [ -n "$SUCCESSES" ]; then
    echo "PROVEN APPROACHES:"
    echo "$SUCCESSES" | while IFS='|' read -r pattern conf; do
        echo "  - $pattern (confidence: $conf)"
    done
fi
```

### 7. session-end.sh

```bash
#!/bin/bash
# Generate session summary and consolidate patterns

DB=".swarm/memory.db"

if [ ! -f "$DB" ]; then
    exit 0
fi

# Count session statistics (last hour)
STATS=$(sqlite3 "$DB" << 'EOF'
SELECT
    COUNT(*) FILTER (WHERE outcome='success' AND last_seen > datetime('now', '-1 hour')) as successes,
    COUNT(*) FILTER (WHERE outcome='failure' AND last_seen > datetime('now', '-1 hour')) as failures,
    COUNT(*) FILTER (WHERE created_at > datetime('now', '-1 hour')) as new_patterns
FROM patterns;
EOF
)

SUCCESSES=$(echo "$STATS" | cut -d'|' -f1)
FAILURES=$(echo "$STATS" | cut -d'|' -f2)
NEW_PATTERNS=$(echo "$STATS" | cut -d'|' -f3)

SUMMARY="Completed $SUCCESSES tasks successfully, $FAILURES failed. Learned $NEW_PATTERNS new patterns."

# Store session
sqlite3 "$DB" << EOF
INSERT INTO sessions (summary, successes, failures, patterns_learned)
VALUES ('$SUMMARY', $SUCCESSES, $FAILURES, $NEW_PATTERNS);
EOF

echo "SESSION SUMMARY"
echo "==============="
echo "$SUMMARY"

# Get top learnings this session
echo ""
echo "Key learnings:"
sqlite3 "$DB" << 'EOF'
SELECT '  - ' || pattern || ' (' || outcome || ', conf: ' || ROUND(confidence, 2) || ')'
FROM patterns
WHERE last_seen > datetime('now', '-1 hour')
ORDER BY confidence DESC
LIMIT 5;
EOF

# Consolidate if many new patterns
if [ "$NEW_PATTERNS" -gt 20 ]; then
    echo ""
    echo "Consolidating patterns..."
    # Remove very low confidence patterns with many failures
    sqlite3 "$DB" << 'EOF'
DELETE FROM patterns
WHERE confidence < 0.2
  AND occurrence_count > 5
  AND last_seen < datetime('now', '-7 days');
EOF
fi
```

### 8. claude.md

```markdown
# Development Protocol

## Role
You are an AI development assistant. The user acts as PM (Product Manager), reviewing your work at phase gates rather than line-by-line.

## Workflow Phases (SPARC Lite)

### Phase 1: Specification + Architecture
1. Understand requirements fully before coding
2. Design high-level approach
3. Define success criteria
4. **GATE**: Get user approval before implementation

### Phase 2: Implementation + Test
1. Write tests first (TDD when possible)
2. Implement in small increments
3. Run tests after each change
4. **GATE**: Verify tests pass before marking complete

## Automatic Learning
- Pre-task hooks inject past learnings (you'll see "LEARNED PATTERNS")
- Post-task hooks store outcomes
- Use proven approaches when available
- Report failures clearly for learning

## Safety Protocol
1. Never run destructive commands without explicit approval
2. Always verify test output (don't claim tests passed without running)
3. When stuck 3+ times on same issue, stop and report
4. No commits without passing tests

## Quality Gates
- [ ] Requirements understood?
- [ ] Architecture reviewed?
- [ ] Tests written and passing?
- [ ] No security vulnerabilities introduced?
- [ ] Documentation updated if needed?

## End of Session
At session end, you'll see a summary of:
- Tasks completed/failed
- Patterns learned
- Areas needing improvement

Review this to understand session progress.
```

---

## Usage

### Install
```bash
# Create directories
mkdir -p .claude/hooks .swarm

# Copy hook files (make executable)
chmod +x .claude/hooks/*.sh

# Initialize database
# (run the SQLite commands from section 2)
```

### Verify
```bash
# Test pre-task injection
.claude/hooks/pre-task.sh "database optimization"

# Test session start
.claude/hooks/session-start.sh

# Manually add a test pattern
sqlite3 .swarm/memory.db "INSERT INTO patterns (pattern, confidence, outcome) VALUES ('Test pattern', 0.9, 'success')"

# Verify it shows up
.claude/hooks/pre-task.sh "test"
```

### Monitor
```bash
# See all patterns
sqlite3 .swarm/memory.db "SELECT * FROM patterns ORDER BY confidence DESC"

# See failures
sqlite3 .swarm/memory.db "SELECT * FROM failures ORDER BY occurred_at DESC"

# See session history
sqlite3 .swarm/memory.db "SELECT * FROM sessions ORDER BY ended_at DESC"
```

---

## What You Get

1. **Before each task**: Claude sees what worked before
2. **After each task**: Outcomes stored with Bayesian confidence
3. **Dangerous commands**: Blocked automatically
4. **Session start**: Context from previous session
5. **Session end**: Summary and pattern consolidation
6. **Over time**: Confidence converges to empirical success rate

---

## Extension Points

### Add MCP Memory Server
If you need explicit memory control:
```json
{
  "mcpServers": {
    "memory": {
      "command": "node",
      "args": ["./memory-server.js"]
    }
  }
}
```

### Add Test Enforcement Hook
```bash
# .claude/hooks/pre-commit.sh
#!/bin/bash
npm test || exit 1
```

### Add Security Audit Hook
```bash
# .claude/hooks/post-edit.sh (for sensitive files)
if [[ "$1" == *"auth"* ]] || [[ "$1" == *"security"* ]]; then
    echo "SECURITY REVIEW REQUIRED: $1 was modified"
fi
```

---

## Comparison to Full Systems

| Feature | Minimal System | Full claude-flow |
|---------|---------------|------------------|
| Files | 6 | 7,760+ |
| Setup time | 15 min | 1-2 hours |
| Learning | Basic Bayesian | Multi-algorithm |
| Agents | Use Claude's built-in | 64 specialized |
| Memory | SQLite | SQLite + HNSW |
| Benefit | 80% of value | 100% of value |
| Complexity | Low | Very High |

**Recommendation**: Start with minimal, add complexity only when needed.
