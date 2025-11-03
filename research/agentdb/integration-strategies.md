# Integration Strategies: Implementing AgentDB with Claude Code

## Overview

This guide explores different approaches to integrating AgentDB with Claude Code, from hooks-based automation to skills and claude.md directives.

## Strategy Comparison

| Strategy | Automation Level | Setup Complexity | Transparency | Control |
|----------|-----------------|------------------|--------------|---------|
| **Hooks** | Fully automatic | Medium | Transparent | High |
| **MCP Server** | Programmatic | Low | Explicit | High |
| **Skills** | Invocation-based | Low | Explicit | Medium |
| **claude.md** | Directive-based | Very Low | Semi-transparent | Low |
| **Hybrid** | Mixed | High | Variable | Very High |

## Strategy 1: Hooks-Based (Recommended)

### Philosophy
Automatic learning with zero agent code changes. Hooks intercept operations and inject/store learnings transparently.

### Setup

```bash
# 1. Install claude-flow with AgentDB
npm install -g claude-flow

# 2. Initialize project
cd your-project
claude-flow init --memory agentdb

# 3. Enable hooks
claude-flow hooks enable
```

### Hook Implementation

```bash
# .claude/hooks/pre-task.sh
#!/bin/bash
TASK="$1"

# Query ReasoningBank for relevant patterns
PATTERNS=$(sqlite3 .swarm/memory.db <<EOF
SELECT pattern, confidence
FROM patterns
WHERE context LIKE '%${TASK}%'
  AND confidence > 0.7
ORDER BY confidence DESC
LIMIT 5;
EOF
)

if [ -n "$PATTERNS" ]; then
  echo "LEARNED_PATTERNS:"
  echo "$PATTERNS" | while IFS='|' read pattern conf; do
    echo "- $pattern (confidence: $conf)"
  done
fi
```

```bash
# .claude/hooks/post-task.sh
#!/bin/bash
TASK="$1"
OUTCOME="$2"
DURATION="$3"

# Extract pattern and store
sqlite3 .swarm/memory.db <<EOF
INSERT INTO patterns (pattern, context, outcome, duration_ms)
VALUES ('$TASK', '$(context)', '$OUTCOME', $DURATION)
ON CONFLICT(pattern) DO UPDATE SET
  confidence = confidence * 0.9 + 0.1 * (CASE WHEN '$OUTCOME'='success' THEN 1 ELSE 0 END),
  occurrence_count = occurrence_count + 1;
EOF
```

### Pros
- ✓ Zero code changes needed
- ✓ Works with all Claude operations
- ✓ Transparent learning
- ✓ Automatic pattern injection

### Cons
- ✗ Requires hook setup
- ✗ Debugging can be tricky
- ✗ Less explicit control

### Best For
- Long-running autonomous agents
- Production deployments
- Teams wanting "set and forget"

## Strategy 2: MCP Server

### Philosophy
Explicit memory operations via Model Context Protocol. Agent explicitly reads/writes memory.

### Setup

```bash
# 1. Install AgentDB MCP server
npm install -g agentdb-mcp

# 2. Configure in claude_desktop_config.json
{
  "mcpServers": {
    "agentdb": {
      "command": "agentdb-mcp",
      "args": ["--db", ".swarm/memory.db"]
    }
  }
}
```

### Available MCP Tools

```typescript
// 20 MCP tools available

// Memory Operations
agentdb_store_pattern(pattern, context, confidence)
agentdb_query_patterns(query, threshold, limit)
agentdb_update_confidence(pattern_id, new_confidence)

// Vector Search
agentdb_semantic_search(query_text, k)
agentdb_embed_text(text)

// Causal Reasoning
agentdb_add_causal_link(cause, effect, confidence)
agentdb_query_causal_chain(effect, max_depth)

// Session Management
agentdb_start_session(agent_id)
agentdb_end_session(summary)
agentdb_get_session_stats()

// Analysis
agentdb_analyze_failures(context)
agentdb_get_success_patterns(min_confidence)
agentdb_evaluate_pattern(pattern_id, score)

// Maintenance
agentdb_consolidate_patterns()
agentdb_export_memory(format)
agentdb_import_memory(data)
```

### Usage Example

```javascript
// Agent explicitly uses MCP tools

// Before task
const patterns = await mcp.agentdb_query_patterns("database optimization", 0.7, 5);
console.log("Past learnings:", patterns);

// Perform task
const result = optimizeDatabaseQueries();

// After task
await mcp.agentdb_store_pattern(
  "Added compound index on (user_id, created_at)",
  "database optimization",
  result.success ? 0.85 : 0.4
);
```

### Pros
- ✓ Explicit control
- ✓ Standard protocol (MCP)
- ✓ Easy to debug
- ✓ Works across different agents

### Cons
- ✗ Agent must know about memory
- ✗ Requires explicit calls
- ✗ Not automatic

### Best For
- Explicit learning workflows
- Research and development
- Fine-grained memory control

## Strategy 3: Skill-Based

### Philosophy
Create a reusable skill that agents can invoke to interact with memory.

### Implementation

```javascript
// .claude/skills/memory.skill

export const skill = {
  name: "memory",
  description: "Interact with agent memory system",

  commands: {
    learn: async (pattern, context, confidence = 0.7) => {
      // Store a new pattern
      await db.insert({
        table: "patterns",
        values: { pattern, context, confidence, outcome: "success" }
      });
      return `Learned: ${pattern}`;
    },

    recall: async (query, threshold = 0.7) => {
      // Query past patterns
      const patterns = await db.query(`
        SELECT pattern, confidence
        FROM patterns
        WHERE context LIKE '%${query}%'
        AND confidence > ${threshold}
        ORDER BY confidence DESC
        LIMIT 5
      `);
      return patterns;
    },

    assess: async () => {
      // Self-assessment
      const stats = await db.query(`
        SELECT
          COUNT(*) as total_patterns,
          AVG(confidence) as avg_confidence,
          SUM(CASE WHEN outcome='success' THEN 1 ELSE 0 END) as successes,
          SUM(CASE WHEN outcome='failure' THEN 1 ELSE 0 END) as failures
        FROM patterns
      `);
      return stats;
    },

    improve: async () => {
      // Consolidate and improve patterns
      await db.execute("CALL consolidate_patterns()");
      const improvements = await db.query(`
        SELECT * FROM improvements
        WHERE created_at > datetime('now', '-1 hour')
      `);
      return improvements;
    }
  }
};
```

### Usage

```
User: "Build a REST API for user management"

Claude: Let me recall past API patterns...
[Invokes: memory.recall("REST API")]

Returns:
- "Use Express.js with TypeScript" (0.92)
- "Implement JWT authentication" (0.88)
- "Add rate limiting middleware" (0.85)

Claude: Based on past successes, I'll use Express with TypeScript...

[Builds API]

Claude: Recording this success...
[Invokes: memory.learn("Express + TypeScript + JWT for REST API", "API development", 0.9)]
```

### Pros
- ✓ Explicit and visible
- ✓ Reusable across projects
- ✓ Easy to understand
- ✓ Good for teaching agents

### Cons
- ✗ Agent must remember to invoke
- ✗ Not automatic
- ✗ Can be verbose

### Best For
- Deliberate learning workflows
- Teaching/training scenarios
- Transparent learning processes

## Strategy 4: claude.md Directives

### Philosophy
Add memory directives to `.claude/claude.md` to guide agent behavior.

### Implementation

```markdown
# .claude/claude.md

You are an AI agent with persistent memory capabilities.

## Memory Protocol

### Before Complex Tasks
1. Query your memory database for similar past tasks:
   ```sql
   SELECT pattern, confidence FROM patterns
   WHERE context LIKE '%[task_type]%' AND confidence > 0.7
   ORDER BY confidence DESC LIMIT 5;
   ```
2. Review learned patterns and apply proven strategies first
3. Note any past failures to avoid repeating mistakes

### After Task Completion
1. Extract the core pattern from your approach
2. Assess the outcome (success/failure)
3. Store the learning:
   ```sql
   INSERT INTO patterns (pattern, context, confidence, outcome)
   VALUES ('[pattern]', '[context]', [confidence], '[outcome]');
   ```
4. Update causal links if applicable

### Self-Assessment Protocol (End of Session)
1. Count successes and failures
2. Identify highest-confidence patterns
3. Review failure patterns for improvement
4. Consolidate duplicate patterns
5. Generate session summary

## Always Active Rules
- Check memory before starting multi-step tasks
- Record all failures with error details
- Update confidence scores after each task
- Build causal links between actions and outcomes

## Memory Database
Location: `.swarm/memory.db`
Tables: patterns, embeddings, trajectories, causal_links, failures
```

### Pros
- ✓ Very easy to setup
- ✓ Agent-readable instructions
- ✓ No code required
- ✓ Transparent process

### Cons
- ✗ Relies on agent compliance
- ✗ Not enforced (agent might forget)
- ✗ Manual SQL in prompts
- ✗ Verbose

### Best For
- Quick prototyping
- Learning/educational purposes
- Non-critical applications

## Strategy 5: Hybrid Approach (Most Powerful)

### Philosophy
Combine multiple strategies for maximum effectiveness.

### Architecture

```
┌─────────────────────────────────────────┐
│          .claude/claude.md              │
│   (High-level memory directives)        │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│          Hooks System                   │
│   (Automatic pre/post operations)       │
└─────────┬───────────────────────┬───────┘
          │                       │
┌─────────▼──────────┐   ┌───────▼────────┐
│   MCP Server       │   │  Skills        │
│ (Explicit control) │   │ (Invocable)    │
└─────────┬──────────┘   └───────┬────────┘
          │                      │
          └──────────┬───────────┘
                     │
              ┌──────▼──────┐
              │  AgentDB    │
              │   Core      │
              └─────────────┘
```

### Implementation

```yaml
# .claude/agentdb-config.yaml

memory:
  backend: sqlite
  database: .swarm/memory.db

hooks:
  enabled: true
  pre-task: .claude/hooks/pre-task.sh
  post-task: .claude/hooks/post-task.sh
  # Automatic baseline learning

mcp:
  enabled: true
  server: agentdb-mcp
  # For explicit memory operations when needed

skills:
  - name: memory
    path: .claude/skills/memory.skill
    # Invocable for deliberate learning

directives:
  source: .claude/claude.md
  # High-level guidance
```

### Usage Flow

```
1. [claude.md] Agent reads: "Check memory before complex tasks"

2. [Hooks] pre-task automatically fires:
   - Queries patterns
   - Injects context

3. [Agent] Sees injected patterns, decides to use MCP for deeper analysis:
   - Calls: agentdb_semantic_search("microservices architecture")
   - Gets: Top 10 related learnings

4. [Agent] Executes task

5. [Hooks] post-task automatically fires:
   - Stores basic outcome

6. [Agent] Explicitly uses skill for detailed learning:
   - Calls: memory.learn("API Gateway pattern resolved auth complexity", "microservices", 0.9)

7. [End of session - claude.md directive]
   - Agent invokes: memory.assess()
   - Generates self-reflection report
```

### Pros
- ✓ Best of all approaches
- ✓ Automatic + explicit control
- ✓ Graceful degradation
- ✓ Maximum flexibility

### Cons
- ✗ Most complex setup
- ✗ Potential for conflicts
- ✗ Requires understanding all parts

### Best For
- Production autonomous agents
- Research platforms
- Maximum learning effectiveness

## Self-Assessment Integration

### Approach 1: Post-Session Skill

```javascript
// .claude/skills/memory.skill

assess_session: async () => {
  const stats = await db.query(`
    SELECT
      COUNT(*) FILTER (WHERE outcome='success') as successes,
      COUNT(*) FILTER (WHERE outcome='failure') as failures,
      AVG(confidence) FILTER (WHERE created_at > datetime('now', '-1 hour')) as avg_confidence,
      COUNT(*) FILTER (WHERE created_at > datetime('now', '-1 hour')) as new_patterns
    FROM patterns
  `);

  const top_learnings = await db.query(`
    SELECT pattern, confidence
    FROM patterns
    WHERE created_at > datetime('now', '-1 hour')
    ORDER BY confidence DESC
    LIMIT 5
  `);

  const failures = await db.query(`
    SELECT error_type, error_msg
    FROM failures
    WHERE occurred_at > datetime('now', '-1 hour')
  `);

  return {
    summary: `Session: ${stats.successes} successes, ${stats.failures} failures`,
    new_patterns: stats.new_patterns,
    avg_confidence: stats.avg_confidence,
    top_learnings,
    failures_to_review: failures
  };
}
```

**Invocation**: At end of session, agent calls `memory.assess_session()`

### Approach 2: Automatic Session-End Hook

```bash
# .claude/hooks/session-end.sh
#!/bin/bash

# Generate automatic assessment
SUMMARY=$(sqlite3 .swarm/memory.db <<EOF
SELECT
  'Completed ' || COUNT(*) FILTER (WHERE outcome='success') || ' tasks successfully, ' ||
  COUNT(*) FILTER (WHERE outcome='failure') || ' failed. ' ||
  'Learned ' || COUNT(*) FILTER (WHERE created_at > datetime('now', '-1 hour')) || ' new patterns.'
FROM patterns;
EOF
)

echo "SESSION SUMMARY: $SUMMARY"

# Store session record
sqlite3 .swarm/memory.db <<EOF
INSERT INTO sessions (summary, ended_at)
VALUES ('$SUMMARY', datetime('now'));
EOF
```

**Trigger**: Automatically runs when Claude Code session ends

### Approach 3: claude.md Self-Reflection Prompt

```markdown
# .claude/claude.md

## End of Session Protocol

When the user signals session end (or after completing major tasks), perform:

1. **Quantitative Assessment**
   - Count patterns learned this session
   - Calculate success rate
   - Identify highest confidence learnings

2. **Qualitative Reflection**
   - What worked well?
   - What strategies failed?
   - What would you do differently?

3. **Pattern Consolidation**
   - Merge similar patterns
   - Update confidence scores
   - Clean up duplicates

4. **Report Generation**
   ```
   SESSION SUMMARY
   ===============
   Tasks completed: [N]
   Success rate: [X%]
   Top 3 learnings:
   1. [Pattern] (confidence: [C])
   2. [Pattern] (confidence: [C])
   3. [Pattern] (confidence: [C])

   Areas for improvement:
   - [Failure pattern 1]
   - [Failure pattern 2]

   Next session focus: [Recommendation]
   ```
```

## Does AgentDB Modify claude.md Files?

### Current State: No

AgentDB **does not** automatically modify `.claude/claude.md` or skills files.

**Reasoning**:
- Configuration as code should be version controlled
- Automatic modifications could introduce bugs
- Developers should review changes
- Maintains predictability

### What It Could Do (Future Enhancement)

```python
# agentdb/meta_learning.py

class MetaLearner:
    """
    Analyze which claude.md directives lead to success,
    suggest improvements without auto-applying
    """

    def analyze_directive_effectiveness(self):
        # Track which directives correlate with success
        results = db.query("""
            SELECT
                directive,
                AVG(CASE WHEN outcome='success' THEN 1.0 ELSE 0.0 END) as success_rate,
                COUNT(*) as usage_count
            FROM patterns
            JOIN directives ON patterns.directive_used = directives.id
            GROUP BY directive
            HAVING usage_count > 10
            ORDER BY success_rate DESC
        """)

        suggestions = []
        for directive, success_rate, count in results:
            if success_rate < 0.5:
                suggestions.append({
                    "directive": directive,
                    "issue": "Low success rate",
                    "recommendation": "Consider rephrasing or removing",
                    "evidence": f"{count} uses, {success_rate*100:.1f}% success"
                })

        return suggestions

    def suggest_claude_md_improvements(self):
        # Generate diff of proposed changes
        current_content = read_file(".claude/claude.md")
        suggestions = self.analyze_directive_effectiveness()

        proposed_content = apply_suggestions(current_content, suggestions)

        return {
            "current": current_content,
            "proposed": proposed_content,
            "diff": generate_diff(current_content, proposed_content),
            "rationale": suggestions
        }
```

**Developer workflow**:
```bash
$ agentdb suggest-improvements

CLAUDE.MD IMPROVEMENT SUGGESTIONS
==================================

1. Directive: "Always use recursion for tree traversal"
   Success rate: 35% (12 uses)
   Recommendation: REMOVE - Led to stack overflow 6 times
   Proposed: "Use iterative approaches for large tree structures"

2. Directive: "Prefer async/await over promises"
   Success rate: 92% (45 uses)
   Recommendation: STRENGTHEN - Very effective pattern
   Proposed: Add to "Always Active Rules"

Apply these changes? [y/N]
```

### Skill Self-Tuning (Conceptual)

```python
# Track skill effectiveness
def analyze_skill_performance(skill_name):
    stats = db.query("""
        SELECT
            skill_invocation,
            AVG(task_success) as success_rate,
            AVG(duration_ms) as avg_duration,
            COUNT(*) as usage_count
        FROM trajectories
        WHERE skill_used = ?
        GROUP BY skill_invocation
    """, (skill_name,))

    for invocation, success_rate, duration, count in stats:
        if success_rate < 0.6:
            print(f"⚠️  {skill_name}.{invocation} has low success rate: {success_rate}")
            print(f"   Suggestion: Review implementation or remove")

        if duration > 1000 and count > 5:
            print(f"🐌 {skill_name}.{invocation} is slow: {duration}ms average")
            print(f"   Suggestion: Add caching or optimize")
```

## Choosing Your Strategy

### Decision Tree

```
Are you building a production autonomous agent?
│
├─ YES → Use Hybrid Approach
│         (Hooks + MCP + Skills + claude.md)
│
└─ NO → Is automation most important?
        │
        ├─ YES → Use Hooks-Based
        │
        └─ NO → Is explicit control important?
                │
                ├─ YES → Use MCP Server
                │
                └─ NO → Use claude.md Directives
```

### Quick Comparison

**Simplest**: claude.md directives (5 min setup)
**Most Automatic**: Hooks-based (30 min setup)
**Most Control**: MCP Server (15 min setup)
**Most Powerful**: Hybrid (2 hour setup)

## Conclusion

AgentDB offers flexible integration options from simple directives to fully automatic hooks. The right choice depends on your needs:

- **Start simple** with claude.md directives
- **Add hooks** when you want automation
- **Use MCP** when you need explicit control
- **Go hybrid** for production deployments

The innovation is that AgentDB supports **progressive enhancement**—start simple, add sophistication as needed.

---

**Related**: [Hooks Deep Dive](hooks.md) | [Anthropic Comparison](anthropic-comparison.md)
