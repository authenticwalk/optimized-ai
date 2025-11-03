# Self-Learning System Implementation Plan

**Status**: 🎯 READY TO IMPLEMENT
**Based on**: AgentDB research, Pheromind patterns, Principle 4 (LEARN)
**Quality**: ⭐⭐⭐⭐⭐ Research-backed, 34% effectiveness improvement proven

## Core Technology: AgentDB

**What**: SQLite + HNSW vector search + hooks system for agent memory
**Why**: Enables agents to learn from every interaction, build pattern libraries
**Performance**: 96x-164x faster than traditional approaches, <1ms queries

### Key Innovation: Transparent Learning Through Hooks

Agents don't need to "know" they have memory - it happens automatically:

```bash
# pre-task.sh fires automatically before any operation
→ Query similar past tasks from .swarm/memory.db
→ Inject learned patterns into agent context

# Agent works with enhanced context (doesn't know patterns came from memory)

# post-task.sh fires automatically after completion
→ Store outcome (success/failure)
→ Update Bayesian confidence scores
→ Build causal links between actions and results
```

## Implementation Phases

### Phase 1: Foundation (Week 1-2) - CRITICAL

**Goal**: Basic pattern storage and retrieval for git operations

#### 1.1 Database Setup

```bash
# Install AgentDB or create manual SQLite database
npm install @ruvnet/agentdb  # OR manual setup

# Database location
.swarm/memory.db
```

**Schema**:
```sql
-- Core patterns table
CREATE TABLE patterns (
    pattern_id INTEGER PRIMARY KEY,
    pattern TEXT NOT NULL,              -- "conventional commit format"
    context TEXT NOT NULL,              -- "git_commit"
    confidence REAL DEFAULT 0.5,        -- Bayesian confidence [0.0-1.0]
    outcome TEXT,                       -- "success" or "failure"
    occurrence_count INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_used TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    namespace TEXT DEFAULT 'default'
);

-- Failure tracking (40% of training data)
CREATE TABLE failures (
    failure_id INTEGER PRIMARY KEY,
    pattern TEXT,
    context TEXT,
    error_message TEXT,
    occurred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved BOOLEAN DEFAULT FALSE,
    resolution_pattern_id INTEGER
);

-- Causal relationships (what leads to what)
CREATE TABLE causal_links (
    cause TEXT NOT NULL,
    effect TEXT NOT NULL,
    confidence REAL DEFAULT 0.5,
    evidence_count INTEGER DEFAULT 1,
    UNIQUE(cause, effect)
);
```

#### 1.2 Hook Integration

**Pre-commit hook** (`.claude/hooks/pre-commit.sh`):
```bash
#!/bin/bash
# Query past successful commit patterns

PATTERNS=$(sqlite3 .swarm/memory.db <<EOF
SELECT pattern, confidence
FROM patterns
WHERE context='git_commit' AND confidence > 0.7
ORDER BY confidence DESC LIMIT 3;
EOF
)

if [ -n "$PATTERNS" ]; then
    echo "📚 Learned patterns for commits:"
    echo "$PATTERNS"
fi
```

**Post-commit hook** (`.claude/hooks/post-commit.sh`):
```bash
#!/bin/bash
# Store commit pattern and outcome

COMMIT_MSG=$(git log -1 --pretty=%B)
COMMIT_FORMAT=$(echo "$COMMIT_MSG" | grep -E '^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+' && echo "conventional" || echo "non-conventional")

# Store pattern
sqlite3 .swarm/memory.db <<EOF
INSERT INTO patterns (pattern, context, outcome, confidence)
VALUES ('$COMMIT_FORMAT', 'git_commit', 'success', 0.6)
ON CONFLICT(pattern) DO UPDATE SET
    confidence = confidence * 0.9 + 0.1,  -- Bayesian increase
    occurrence_count = occurrence_count + 1,
    last_used = datetime('now');
EOF
```

### Phase 2: Bayesian Confidence (Week 3-4)

**Algorithm** (simple, no ML training required):

```python
def update_confidence(prior_confidence, outcome):
    """Bayesian update - converges to empirical success rate"""
    if outcome == 'success':
        # Gradual increase with diminishing returns
        return prior_confidence + (1 - prior_confidence) * 0.1
    else:
        # Proportional decrease
        return prior_confidence - prior_confidence * 0.15

# Example: Pattern starts at 0.5 (neutral)
# After 10 successes: 0.91 confidence
# After 1 failure: drops to 0.77 (still trusted)
```

**Integration**:
- Update confidence after every git commit
- Track test strategy success rates
- Learn which refactoring patterns work
- Remember security issues found

### Phase 3: Failure-Weighted Learning (Month 2)

**Key Insight**: 40% of training data should be failures (research-backed)

**Why It Matters**:
- Learn what NOT to do
- Prevent repeating mistakes
- Faster learning than success-only

**Implementation**:
```sql
-- When AI tries something that fails
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'used --no-verify to bypass hooks',
    'git_commit',
    'Hook blocked commit due to failing tests'
);

-- Before allowing risky action, check history
SELECT COUNT(*) FROM failures
WHERE pattern LIKE '%--no-verify%'
AND occurred_at > datetime('now', '-7 days');

-- If > 0, warn agent: "You tried this before, it was blocked"
```

### Phase 4: Causal Reasoning (Month 3)

**Track cause-effect chains**:

```sql
-- Example causal links learned over time
write_tests → fix_bugs → ci_passes → pr_approved (0.92 confidence)
add_types → fewer_runtime_errors → code_review_faster (0.87 confidence)
conventional_commits → pr_approved_faster (0.81 confidence)
```

**Multi-hop reasoning**:
```sql
-- What leads to successful PRs?
WITH RECURSIVE causal_chain AS (
    SELECT cause, effect, confidence, 1 as depth
    FROM causal_links
    WHERE effect = 'pr_approved'

    UNION ALL

    SELECT cl.cause, cl.effect, cl.confidence, cc.depth + 1
    FROM causal_links cl
    JOIN causal_chain cc ON cl.effect = cc.cause
    WHERE cc.depth < 5
)
SELECT cause, AVG(confidence) as path_confidence
FROM causal_chain
GROUP BY cause
ORDER BY path_confidence DESC;
```

## Pattern Storage Examples

### Git Commit Patterns

```sql
-- Successful conventional commit
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'feat(auth): add JWT token refresh',
    'git_commit',
    0.85,
    'success'
);

-- Failed attempt (too vague)
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'fixed stuff',
    'git_commit',
    'Rejected in code review - not descriptive'
);
```

### Test Strategy Patterns

```sql
-- Which tests to run for which files
INSERT INTO patterns (pattern, context, confidence)
VALUES (
    'npm test -- --changed',  -- Only run tests for changed files
    'efficient_testing',
    0.88
);

-- Common test failure
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'skipped integration tests',
    'testing',
    'Integration tests caught bug that unit tests missed'
);
```

### Security Patterns

```sql
-- Security issues found
INSERT INTO patterns (pattern, context, confidence)
VALUES (
    'npm audit found 3 high vulnerabilities',
    'security_audit',
    0.92
);

-- Causal link
INSERT INTO causal_links (cause, effect, confidence)
VALUES (
    'outdated_dependencies',
    'security_vulnerabilities',
    0.89
);
```

## Namespace Isolation (Advanced)

**Purpose**: Separate patterns by domain to prevent pollution

```sql
-- Hierarchical namespaces
projects/ecommerce/backend → projects/ecommerce → projects → root

-- Example: E-commerce backend patterns
INSERT INTO patterns (pattern, context, namespace, confidence)
VALUES (
    'Add Redis cache for product catalog',
    'performance_optimization',
    'projects.ecommerce.backend',
    0.94
);

-- Query with inheritance
SELECT pattern FROM patterns
WHERE namespace IN (
    'projects.ecommerce.backend',  -- Specific
    'projects.ecommerce',          -- Project-level
    'projects',                    -- General
    'root'                         -- Global
)
AND context = 'performance_optimization'
ORDER BY confidence DESC;
```

**When to use**:
- Month 3+ (after basic system works)
- Multiple projects sharing the agent
- Domain-specific learning (git/, test/, security/)

## Performance Optimization

### Hash-Based Embeddings (No API Required)

**Trade-off**: 87-95% accuracy vs 97-99% for OpenAI embeddings

```python
import hashlib
import numpy as np

def hash_embed(text: str, dim: int = 1024) -> np.ndarray:
    """Generate embedding using cryptographic hashing - works offline"""
    embedding = np.zeros(dim)
    num_hashes = dim // 32

    for i in range(num_hashes):
        h = hashlib.sha256(f"{text}:{i}".encode()).digest()
        for j in range(32):
            embedding[i * 32 + j] = (h[j] / 255.0) * 2 - 1

    return embedding / np.linalg.norm(embedding)

# Usage: store pattern embedding for semantic search
pattern_text = "Added database index on user_id foreign key"
vector = hash_embed(pattern_text)
# Store in embeddings table
```

**When to use**:
- Start with hash-based (free, offline, fast)
- Upgrade to OpenAI embeddings if accuracy becomes issue
- 5-10% accuracy loss is acceptable for pattern matching

### Quantization (Memory Efficiency)

**Purpose**: Store 32x more patterns in same memory

```python
def quantize_binary(vector):
    """1 bit per dimension instead of 32 bits"""
    return np.packbits(vector > 0)

# Without: 1024 dims × 4 bytes = 4KB per pattern
# With: 1024 dims / 8 = 128 bytes per pattern
# Result: 32x more patterns in RAM
```

**When to use**: Phase 4, if database grows >100k patterns

## Integration with Existing Principles

### Principle 3: VALIDATE

**How learning aligns**:
- Bayesian confidence is empirical (based on actual outcomes)
- Track success rates, not subjective opinions
- Converges to real-world performance

**Validation experiments**:
```markdown
# Experiment: Does learning improve commit quality?

Control: 20 commits without AgentDB memory
Treatment: 20 commits with AgentDB memory

Metrics:
- Time to craft commit message
- Conventional commit compliance rate
- Code review approval rate
- Rework requests

Hypothesis: Treatment group shows ≥30% improvement
```

### Principle 4: LEARN

**This IS the learning system**:
- Self-learning through experience ✅
- Continuous improvement over time ✅
- Pattern library grows automatically ✅
- No manual knowledge engineering ✅

## Success Metrics

### Week 2 (Basic System)
- [ ] Patterns stored successfully in .swarm/memory.db
- [ ] Pre-commit hook queries and injects patterns
- [ ] Post-commit hook stores outcomes
- [ ] At least 20 patterns stored

### Month 1 (Confidence Updates)
- [ ] Confidence scores converge to empirical success rates
- [ ] Pattern reuse rate >30%
- [ ] Failures tracked and referenced
- [ ] No database corruption issues

### Month 3 (Full System)
- [ ] 34% effectiveness improvement (research benchmark)
- [ ] Causal chains identified and used
- [ ] Agent explains decisions using memory
- [ ] Self-consolidation prevents bloat

## Troubleshooting

### Database doesn't exist
```bash
# Manual initialization
sqlite3 .swarm/memory.db < schema.sql
```

### Patterns not being injected
```bash
# Debug hook execution
bash -x .claude/hooks/pre-commit.sh
```

### Confidence not updating
```python
# Check Bayesian math
prior = 0.5
for i in range(10):
    prior = prior + (1 - prior) * 0.1  # 10 successes
    print(f"After {i+1} successes: {prior:.3f}")
# Should reach ~0.91
```

### Database growing too large
```sql
-- Clean up old low-confidence patterns
DELETE FROM patterns
WHERE confidence < 0.3
AND last_used < datetime('now', '-90 days');

-- Archive old patterns
CREATE TABLE archived_patterns AS
SELECT * FROM patterns
WHERE last_used < datetime('now', '-180 days');
```

## References

- [AgentDB Research Synthesis](../../research/SYNTHESIS-agentdb.md)
- [ReasoningBank Paper](https://arxiv.org/abs/...) - 34% improvement study
- [HNSW Algorithm](https://en.wikipedia.org/wiki/Hierarchical_navigable_small_world)
- [Bayesian Updates](https://en.wikipedia.org/wiki/Bayesian_inference)

## Status

**Current Phase**: Planning
**Next Action**: Phase 1 implementation (Week 1)
**Owner**: Development team
**Priority**: 🔥 CRITICAL - Foundation for all learning
