# Deep Dive: Letta (MemGPT) - Stateful Agents with Advanced Memory

## What It Is

Letta is a platform for building stateful agents: open AI with advanced memory that can learn and self-improve over time. It's made by the creators of MemGPT, a research paper that introduced the concept of the "LLM Operating System" for memory management.

## Repository

https://github.com/letta-ai/letta

## Key Innovation

**LLM Operating System Concept**: Manage LLM memory like an OS manages computer memory—with hierarchies, paging, caching, and persistence.

## Core Concepts

### 1. Memory Hierarchies

Just like computer memory (registers → cache → RAM → disk), LLMs need memory hierarchies:

**Analogy**:
```
CPU Registers  → Context Window (immediate access, very limited)
CPU Cache      → Conversation Summary (fast, moderately sized)
RAM            → Session Memory (medium speed, larger capacity)
Hard Disk      → Long-term Knowledge (slow, unlimited capacity)
```

**LLM Application**:
```
Context Window → Current conversation (8k-200k tokens, expensive)
Summary        → Compressed recent context (fast to generate/load)
Session Store  → This conversation's facts (queryable)
Long-term DB   → All learned knowledge (searchable, persistent)
```

### 2. Persistent Editable Memory Blocks

Memory is organized into **blocks** that can be:
- **Read** by the agent at any time
- **Edited** by the agent when learning
- **Persisted** across sessions
- **Searched** semantically

**Memory Block Types**:
1. **Core Memory** - Always in context (persona, task context)
2. **Recall Memory** - Recent conversation history
3. **Archival Memory** - Long-term facts and learnings

### 3. Perpetual Self-Improving Agents

Agents that:
- Run indefinitely (not just single sessions)
- Learn from every interaction
- Update their own memory
- Improve their responses over time
- Maintain context across days/weeks/months

## How It Works

### Memory Management Functions

Agents have tools to manage their own memory:

```python
# Core memory (always loaded)
agent.edit_core_memory("user_preferences", "Prefers TypeScript over JavaScript")

# Recall memory (recent conversation)
agent.recall_recent("What did user say about Firebase?")

# Archival memory (long-term storage)
agent.archival_insert("Firebase auth pattern: always use onAuthStateChanged")
agent.archival_search("Firebase patterns")
```

### Automatic Memory Paging

Like OS virtual memory, Letta automatically:
1. Keeps important info in context
2. Summarizes less important info
3. Pages out old info to archival storage
4. Retrieves from archival when needed

## Features

### 1. Memory Hierarchies

```
┌─────────────────────────────────────┐
│     Context Window (Active)         │
│  ┌───────────────────────────────┐  │
│  │   Core Memory (Persona)       │  │ Always loaded
│  │   - User preferences          │  │
│  │   - Current task              │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │   Recall Memory (Recent)      │  │ Recent history
│  │   - Last 10-20 messages       │  │
│  │   - Current conversation      │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
         ↓ Paging ↑
┌─────────────────────────────────────┐
│   Archival Memory (Long-term)       │
│   - All past conversations          │ Searchable
│   - Learned patterns                │ Persistent
│   - Historical context              │ Unlimited
└─────────────────────────────────────┘
```

### 2. Persistent Editable Memory

Agents can **edit** their own memory:
- Update preferences as they learn
- Correct mistakes in stored facts
- Refine understanding over time

### 3. Cross-Session Learning

Agent retains memory across:
- Multiple conversations
- Days or weeks
- Project phases
- Different tasks

### 4. Self-Improvement

Agent improves by:
- Analyzing past failures
- Updating strategies
- Refining patterns
- Learning user preferences

## Alignment with Our Project

### 🔥 HIGH Alignment - LEARN Principle

**Memory Architecture**:
✅ Validates our `.ai-knowledge/` structured storage
✅ Shows need for memory hierarchy (not just flat files)
✅ Demonstrates persistent learning across sessions

**Our Implementation Needs**:
```
Current plan:
.ai-knowledge/
├── patterns.json (all patterns in one file)
├── failures.json (all failures in one file)
└── preferences.json (all preferences in one file)

Letta-inspired enhancement:
.ai-knowledge/
├── knowledge.db (SQLite)
│   ├── core_memory (always loaded - user prefs, project context)
│   ├── recall_memory (recent tasks, last 20 sessions)
│   └── archival_memory (all historical learnings)
└── embeddings/ (vector search for semantic retrieval)
```

### ✅ MEDIUM Alignment - MINIMIZE Principle

**Context Management**:
✅ Automatic summarization reduces token usage
✅ Load only necessary memory into context
✅ Hierarchies prevent context bloating

**Application**:
- Don't load all learnings - query and load top 5 relevant
- Summarize long contexts automatically
- Page out old info, retrieve when needed

### ✅ MEDIUM Alignment - VALIDATE Principle

**Self-Improvement**:
✅ Agents can validate and correct their own memory
✅ Errors in memory can be edited
✅ Learning from past mistakes

## Key Innovations to Adapt

### 1. Memory Hierarchy Architecture (HIGH PRIORITY)

**Innovation**: Structured memory levels like OS memory

**Our Application**:
```sql
-- knowledge.db schema

-- Core Memory (always loaded, ~1KB)
CREATE TABLE core_memory (
  key TEXT PRIMARY KEY,
  value TEXT,
  last_updated TIMESTAMP
);
-- Examples: user_name, project_type, tech_stack, coding_style

-- Recall Memory (recent context, ~100 entries)
CREATE TABLE recall_memory (
  id INTEGER PRIMARY KEY,
  task_id TEXT,
  timestamp TIMESTAMP,
  summary TEXT,  -- Compressed version of task
  outcome TEXT,  -- Success/failure
  patterns_used TEXT[]
);
-- Last 20-50 tasks for quick context

-- Archival Memory (unlimited, searchable)
CREATE TABLE archival_memory (
  id INTEGER PRIMARY KEY,
  category TEXT,  -- pattern, failure, preference, learning
  content TEXT,
  embedding BLOB,  -- For semantic search
  metadata JSON,
  created TIMESTAMP
);
-- All historical learnings
```

**Benefits**:
- Fast access to core info (user preferences)
- Quick context from recent tasks
- Deep search when needed
- Prevents context bloat

### 2. Editable Memory (MEDIUM PRIORITY)

**Innovation**: Agents can update their own memory

**Our Application**:
```markdown
# Learner agent can update memory after each task

After task completion:
1. Review what worked/didn't work
2. Update relevant patterns in archival_memory
3. Correct any wrong assumptions in core_memory
4. Add new learnings to recall_memory
5. Summarize and archive old recall_memory

Example:
- Learns: Firebase auth pattern X works better than Y
- Action: Update archival_memory with new pattern
- Action: Mark old pattern as deprecated
- Result: Next time, uses better pattern automatically
```

### 3. Memory Paging/Summarization (MEDIUM PRIORITY)

**Innovation**: Automatic summarization to manage context

**Our Application**:
```markdown
# After 50 tasks in recall_memory:

1. Identify patterns across tasks
2. Promote common patterns to archival_memory
3. Summarize old recall_memory entries
4. Keep summaries, archive full details
5. Free up recall_memory for recent tasks

Example:
- 20 Firebase auth tasks in recall_memory
- Common pattern identified: "Always use onAuthStateChanged"
- Pattern promoted to archival_memory
- Old tasks summarized: "Firebase auth tasks (20) - pattern learned"
- Recall_memory freed for new tasks
```

### 4. Perpetual Agent (LOW PRIORITY - Future)

**Innovation**: Agents that run continuously, learning over time

**Our Application** (Long-term):
```markdown
# Phase 1-3: Session-based (current plan)
- Agent activated per task
- Loads memory for task
- Completes task
- Updates memory
- Exits

# Phase 4+: Perpetual agent (future)
- Agent always running in background
- Monitors repo for changes
- Learns from all Git activity (not just its own)
- Proactively suggests improvements
- Continuous learning

Example:
- User manually fixes a bug
- Agent observes the fix (git diff)
- Agent learns: "This is the correct pattern"
- Agent updates memory with correction
- Next time, agent uses correct pattern
```

## Comparisons to Other Memory Systems

### vs Mem0
**Letta (MemGPT)**:
- ✅ Hierarchical memory (core/recall/archival)
- ✅ Editable memory blocks
- ✅ LLM OS metaphor
- ⚠️ More complex architecture

**Mem0**:
- ✅ Simpler API (one line to enable)
- ✅ MCP integration
- ✅ Universal layer for any LLM
- ⚠️ Less sophisticated hierarchy

**Our Choice**: Borrow Letta's hierarchy, Mem0's simplicity

### vs Memori
**Letta (MemGPT)**:
- ✅ Memory hierarchies
- ✅ Automatic paging
- ⚠️ More opinionated structure

**Memori**:
- ✅ SQL-based (familiar queries)
- ✅ Transparent and portable
- ✅ Entity extraction + relationships
- ⚠️ Requires structured extraction

**Our Choice**: SQL for storage (Memori), hierarchies for organization (Letta)

### vs Cognee
**Letta (MemGPT)**:
- ✅ Hierarchical structure
- ✅ Persistent memory blocks
- ⚠️ Mostly vector-based search

**Cognee**:
- ✅ Hybrid: Vector + Graph
- ✅ Meaning AND relationships
- ✅ Knowledge graph visualization
- ⚠️ More complex setup

**Our Choice**: Start simple (Letta-style hierarchy), add graph later (Cognee)

## Implementation Recommendations

### Phase 1: Basic Hierarchy (Immediate)

```sql
-- Simple 3-tier hierarchy

-- Tier 1: Core (always loaded)
CREATE TABLE core_memory (
  key TEXT PRIMARY KEY,
  value TEXT
);

-- Tier 2: Recall (recent tasks)
CREATE TABLE recall_memory (
  id INTEGER PRIMARY KEY,
  task_id TEXT,
  summary TEXT,
  timestamp TIMESTAMP
);

-- Tier 3: Archival (all learnings)
CREATE TABLE archival_memory (
  id INTEGER PRIMARY KEY,
  category TEXT,
  content TEXT,
  timestamp TIMESTAMP
);
```

### Phase 2: Add Editability (Short-term)

```typescript
// Learner agent can update memory

async function updateMemory(category: string, oldContent: string, newContent: string) {
  await db.execute(`
    UPDATE archival_memory
    SET content = ?, timestamp = ?
    WHERE category = ? AND content = ?
  `, [newContent, Date.now(), category, oldContent]);
}

// Example usage after task:
if (task.outcome === 'success' && improvedPattern) {
  await updateMemory('pattern', oldPattern, newPattern);
}
```

### Phase 3: Add Semantic Search (Medium-term)

```typescript
// Add vector embeddings for semantic search

async function archivalSearch(query: string, limit: number = 5) {
  const queryEmbedding = await generateEmbedding(query);

  const results = await db.execute(`
    SELECT content,
           cosine_similarity(embedding, ?) as similarity
    FROM archival_memory
    WHERE category = 'pattern'
    ORDER BY similarity DESC
    LIMIT ?
  `, [queryEmbedding, limit]);

  return results;
}

// Example usage:
const relevantPatterns = await archivalSearch("Firebase authentication", 5);
// Returns top 5 most semantically similar patterns
```

### Phase 4: Add Automatic Paging (Long-term)

```typescript
// Periodically summarize and archive old recall memory

async function pageRecallMemory() {
  // If recall memory > 50 entries
  const count = await db.count('recall_memory');

  if (count > 50) {
    // Identify old entries (>30 days)
    const old = await db.query(`
      SELECT * FROM recall_memory
      WHERE timestamp < ?
      LIMIT 20
    `, [Date.now() - 30 * 24 * 60 * 60 * 1000]);

    // Summarize old entries
    const summary = await summarizeEntries(old);

    // Move summary to archival
    await db.insert('archival_memory', {
      category: 'summarized_tasks',
      content: summary,
      timestamp: Date.now()
    });

    // Delete old entries from recall
    await db.delete('recall_memory',
      old.map(e => e.id)
    );
  }
}
```

## Experiments to Run

### Experiment 1: Memory Hierarchy Effectiveness
**Hypothesis**: "3-tier hierarchy reduces token usage 50% vs flat storage"
**Design**:
- Control: Load all patterns for every task
- Treatment: Load core + relevant patterns only
- Measure: Token usage, task success rate

### Experiment 2: Editable Memory Impact
**Hypothesis**: "Editable memory improves pattern quality 30% over time"
**Design**:
- Track pattern success rate over 100 tasks
- Allow agent to update patterns after failures
- Measure: Pattern effectiveness over time
- Expected: Gradual improvement as patterns refined

### Experiment 3: Recall vs Archival
**Hypothesis**: "Recent tasks (recall) are 10x more relevant than old tasks (archival)"
**Design**:
- Measure how often recall memory is used vs archival
- Track relevance scores
- Determine optimal recall memory size

## Questions for Further Research

1. **Optimal Hierarchy Sizes**
   - How much core memory? (1KB? 5KB?)
   - How many recall entries? (20? 50? 100?)
   - When to page to archival?

2. **Summarization Strategy**
   - What to keep when summarizing?
   - How to compress without losing critical details?
   - Automatic or manual summarization?

3. **Editing Strategy**
   - When should agent edit vs append?
   - How to prevent incorrect edits?
   - Audit trail for memory changes?

4. **Search Strategy**
   - When to use semantic search vs SQL queries?
   - Hybrid approaches?
   - Caching frequently accessed memory?

## Key Takeaways

1. ✅ **Memory hierarchies are essential** - Not all memory is equally important
2. ✅ **Editable memory enables improvement** - Agents should update their knowledge
3. ✅ **Paging prevents bloat** - Automatic summarization maintains context size
4. ✅ **LLM OS is powerful metaphor** - Think about memory like computer memory
5. ⚠️ **Complexity tradeoff** - More sophisticated = more to implement
6. ✅ **Validates our approach** - `.ai-knowledge/` is right direction, needs hierarchy

## Integration with Our Principles

### MINIMIZE
- ✅ Hierarchies prevent loading all memory
- ✅ Summarization compresses context
- ✅ Supports 40% token reduction goal

### SEPARATE
- ✅ Memory as separate concern
- ✅ Clear boundaries between memory tiers
- ✅ Specialized learner agent for memory management

### VALIDATE
- ✅ Editable memory allows correction
- ✅ Can measure memory effectiveness over time
- ✅ Audit trail for memory changes

### LEARN
- ✅ Core concept - continuous learning across sessions
- ✅ Self-improvement through memory updates
- ✅ Pattern promotion and refinement

## Status

**Research Complete**: ✅
**Priority**: HIGH - Critical for our LEARN principle
**Impact**: HIGH - Fundamentally improves our memory architecture
**Next Actions**:
1. Update `.ai-knowledge/` schema with hierarchy
2. Implement basic 3-tier system (Phase 1)
3. Add editability (Phase 2)
4. Experiment with effectiveness

## Related Research

- [Mem0](./mem0-memory-layer.md) - Alternative memory system
- [Memori](./memori-sql-memory.md) - SQL-based approach
- [Cognee](./cognee-memory.md) - Vector + Graph hybrid
- [Anthropic Best Practices](./anthropic-best-practices.md) - Memory as architecture

## References

- GitHub: https://github.com/letta-ai/letta
- MemGPT Paper: (introduced LLM OS concept)
- OS Memory Management: (inspiration for hierarchy)
