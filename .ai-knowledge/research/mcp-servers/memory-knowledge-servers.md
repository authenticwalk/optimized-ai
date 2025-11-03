# Memory & Knowledge Graph MCP Servers

## Core Insight

Memory/knowledge graph servers are **CRITICAL** for our learning system (Principle 4: LEARN). This is the **ONLY MCP server we absolutely need** because there's no suitable CLI alternative for knowledge graph operations.

---

## Official Anthropic Memory Server

**Package:** `@modelcontextprotocol/server-memory`
**Status:** ✅ Official, actively maintained (Nov 2024)
**Language:** TypeScript
**Repository:** Part of official MCP servers

### What It Does

Enables Claude to remember information about users and projects across conversations by storing data in a **local knowledge graph**.

**Core Concepts:**
- **Entities** - Things (users, projects, patterns, skills)
- **Relations** - Connections between entities
- **Observations** - Facts about entities

### Why We Need It

**From our SPEC.md:**
> "`.ai-knowledge/` - Local learning database (git-tracked)"

**From Principle 4 (LEARN):**
> "System continuously improves by learning from successes and failures. Every rule, skill, and pattern has its own learning history."

**Our Use Cases:**
1. **Store learned patterns** - Successful code patterns, architectures
2. **Track failures** - What didn't work and why
3. **Remember preferences** - User's coding preferences, style
4. **Skill effectiveness** - Which skills work for which tasks
5. **Rule usage** - Which rules are referenced, high-impact
6. **Task history** - Successes, failures, diagnostics

### Installation

```bash
# Install globally
npm install -g @modelcontextprotocol/server-memory

# Configure in Claude Code
# Add to MCP settings
```

### Configuration Example

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "env": {
        "MEMORY_FILE_PATH": ".ai-knowledge/memory.db"
      }
    }
  }
}
```

### Integration with Our System

**Storage Structure:**
```
.ai-knowledge/
├── memory.db              # Knowledge graph (MCP server)
├── plans/                 # Task plans
├── research/              # Research findings
└── patterns.json          # Quick-access patterns
```

**Example Knowledge Graph Entries:**

```typescript
// Entity: Successful pattern
{
  type: "entity",
  name: "firebase-auth-initialization",
  entityType: "pattern",
  observations: [
    "Used in 15 tasks with 100% success rate",
    "Tokens saved: avg 3000 per use",
    "Common with: testing.skill",
    "Prevents mistake: wrong auth configuration"
  ]
}

// Relation: Skill effectiveness
{
  type: "relation",
  from: "firebase-auth.skill",
  to: "testing.skill",
  relationType: "commonly_loaded_with",
  observations: [
    "Loaded together 45 times",
    "Combined success rate: 92%",
    "Avg token usage: 28000"
  ]
}

// Entity: Failed approach
{
  type: "entity",
  name: "api-error-handling-attempt-1",
  entityType: "failure",
  observations: [
    "Task: task-2025-11-03-055",
    "Approach: Generic try-catch",
    "Failure reason: Didn't map API error codes",
    "Spun for 3 cycles",
    "Token waste: 30000",
    "Fixed by: Explicit error code mapping"
  ]
}
```

### Operations We'll Use

**Store learnings:**
```typescript
// After successful task
storeEntity({
  name: "task-2025-11-03-042-firebase-auth",
  type: "success",
  observations: [
    "Task: Implement user signup",
    "Rules used: firebase-auth pattern, test-edge-cases",
    "Skills: firebase-auth, testing",
    "Tokens: 22000 (37% below average)",
    "Quality: 0.98",
    "Key: Used exact pattern from skill"
  ]
});

// Create relation
createRelation({
  from: "firebase-auth.skill",
  to: "testing.skill",
  type: "effective_combination",
  observation: "Success rate 92% when loaded together"
});
```

**Query learnings:**
```typescript
// Before starting task
const patterns = queryEntities({
  type: "pattern",
  relatedTo: "firebase-auth"
});

// Check for similar failures
const failures = queryEntities({
  type: "failure",
  keywords: ["API", "error handling"]
});
```

### Why MCP Over CLI?

**Knowledge Graph Operations Need:**
- ✅ Semantic relationships between entities
- ✅ Graph traversal (find related patterns)
- ✅ Context-aware queries
- ✅ Stateful session across conversations

**CLI Limitations:**
- ❌ No graph database CLI suitable for this
- ❌ Would need to build our own (reinventing MCP)
- ❌ Stateless, requires complex scripting
- ❌ No semantic understanding

**Verdict:** MCP is **correct choice** for this use case.

---

## Community Alternatives

### 1. Memento MCP (Neo4j-powered)

**Repository:** gannonh/memento-mcp
**Status:** Active development (2025)
**Backend:** Neo4j graph database

**Features:**
- ✅ Semantic search with vector embeddings
- ✅ Advanced graph operations
- ✅ Production-grade graph database

**Why We Don't Need It (Yet):**
- ❌ Requires Neo4j setup (more complexity)
- ❌ Overkill for initial needs
- ❌ Official Anthropic server sufficient

**When to Consider:**
- Multi-project learning (Phase 2)
- Team collaboration (Phase 2)
- Advanced graph queries needed
- Scale beyond local knowledge graph

---

### 2. mcp-knowledge-graph (shaneholloman)

**Repository:** shaneholloman/mcp-knowledge-graph
**Status:** Active (2025)
**Storage:** Local JSONL

**Features:**
- ✅ AIM (AI Memory) system
- ✅ Project-local memory
- ✅ Named databases
- ✅ Simple JSONL storage

**Comparison to Official:**
- ✅ Simpler storage (JSONL vs graph DB)
- ❌ Less sophisticated queries
- ❌ No official support

**When to Consider:**
- If official server has performance issues
- Need simpler, human-readable storage
- Want to inspect memory files directly

---

### 3. Kuzu-powered Knowledge Graph

**Package:** `@deanacus/knowledge-graph-mcp`
**Status:** Released Sept 2025
**Backend:** Kuzu embedded graph database

**Features:**
- ✅ Embedded graph database
- ✅ No external dependencies
- ✅ Good performance

**Comparison to Official:**
- ✅ Embedded (no external DB)
- ❌ Less mature than Anthropic server
- ❌ Smaller community

---

## Recommendation

### Phase 0-1: Official Anthropic Server

**Install:** `@modelcontextprotocol/server-memory`

**Why:**
- ✅ Official support
- ✅ Well-tested
- ✅ Integrates seamlessly with Claude Code
- ✅ Sufficient for our needs
- ✅ Knowledge graph capabilities
- ✅ Active maintenance

**Storage:** `.ai-knowledge/memory.db`

### Phase 2-3: Evaluate Alternatives

**If we need:**
- Multi-project learning → Consider **Memento MCP** (Neo4j)
- Simpler storage → Consider **shaneholloman/mcp-knowledge-graph**
- Advanced queries → Consider **Neo4j-based** solutions

**Decision criteria (Principle 3: VALIDATE):**
- Performance benchmarks
- Query complexity needs
- Storage size
- Team collaboration requirements

---

## Integration with Learning System

### From Principle 4 (LEARN):

**Learning Loops:**

1. **Usage-Based Optimization**
   ```typescript
   // Track rule usage
   await memory.observe({
     entity: "rule-minimize-boilerplate",
     observation: `Referenced in task-${taskId}, impact: HIGH`
   });

   // Query to make decisions
   const usage = await memory.query({
     entity: "rule-minimize-boilerplate",
     observationType: "usage"
   });
   // If <10% usage → REMOVE
   ```

2. **Conflict Resolution**
   ```typescript
   // Detect conflict
   await memory.createRelation({
     from: "rule-minimize-boilerplate",
     to: "rule-comprehensive-error-handling",
     type: "conflicts_with",
     observation: "AI confused in 8 tasks"
   });
   ```

3. **Pattern Promotion**
   ```typescript
   // Track successful pattern
   await memory.observe({
     entity: "pattern-firebase-auth-initialization",
     observation: "Used in task-042, success, tokens-saved: 3000"
   });

   // Query for promotion decision
   const pattern = await memory.query({
     entity: "pattern-firebase-auth-initialization",
     observationType: "usage"
   });
   // If usage > 40 && successRate > 0.98 → PROMOTE to core
   ```

4. **Skill Effectiveness**
   ```typescript
   // Track skill performance
   await memory.observe({
     entity: "skill-firebase-auth",
     observation: "Loaded in task-042, effectiveness: HIGH, tokens: 28000, quality: 0.92"
   });
   ```

---

## Success Metrics

**How we'll know it's working:**

1. **Learning stored** - Patterns, failures, preferences in graph
2. **Queries working** - Can retrieve relevant learnings before tasks
3. **Decisions improved** - System makes better choices over time
4. **Context preserved** - Knowledge persists across sessions
5. **Performance acceptable** - Query latency < 100ms

---

## Experiments to Run

### Experiment: Memory Server Impact

**Hypothesis:** "Memory server improves task quality by 15% and reduces token usage by 20% for similar tasks."

**Design:**
- Control: No memory, fresh context each time
- Treatment: Memory server with stored learnings

**Scenarios:**
- 10 Firebase auth tasks
- 10 Supabase RLS tasks
- 10 API integration tasks

**Metrics:**
- Token usage (avg, variance)
- Quality scores
- Time to completion
- Repetition of mistakes

**Expected:**
- Treatment shows improvement on repeated task types
- First task: no difference
- Subsequent tasks: significant improvement

**Status:** Phase 1 (after memory server installed)

---

## Alternatives Considered & Rejected

### 1. Plain JSON Files

**Approach:** Store learnings in `.ai-knowledge/patterns.json`, etc.

**Why rejected:**
- ❌ No graph relationships
- ❌ Hard to query complex patterns
- ❌ No semantic understanding
- ❌ Manual relationship management

**Verdict:** Keep JSON for simple quick-access, use graph for complex queries.

---

### 2. SQLite with Custom Schema

**Approach:** Design schema for entities/relations, use `sqlite3`.

**Why rejected:**
- ❌ Reinventing knowledge graph
- ❌ Complex SQL for graph queries
- ❌ No built-in graph operations
- ❌ More work than using MCP

**Verdict:** MCP memory server is designed for this, don't reinvent.

---

### 3. No Persistence

**Approach:** Rely on Claude Code's context window only.

**Why rejected:**
- ❌ Learnings lost between sessions
- ❌ Can't improve over time (Principle 4)
- ❌ No cross-task learning
- ❌ Defeats self-learning goal

**Verdict:** Persistence is core to our system.

---

## Applicability to Our Project

**From SPEC.md Architecture:**

### 2. Learning Database
> "`.ai-knowledge/` - Local learning database (git-tracked)"

**Memory server enables:**
- ✅ patterns.json → Graph entities
- ✅ failures.json → Graph entities with relations
- ✅ preferences.json → User entity observations
- ✅ corrections.json → Improvement relations
- ✅ metrics.json → Performance observations

### 6. Self-Evaluation Loop
> "Review changes against learned patterns"

**Memory server provides:**
- ✅ Query patterns before evaluation
- ✅ Check against known failures
- ✅ Validate against preferences

### 9. Hooks & Agents - Learner Agent
> "learner - Updates knowledge base"

**Memory server is storage backend for learner agent:**
- ✅ Learner agent writes observations
- ✅ Creates relations between learnings
- ✅ Updates entity observations
- ✅ Maintains knowledge graph

---

## Implementation Notes

### Storage Location

```bash
.ai-knowledge/
├── memory.db          # ← Knowledge graph (MCP server manages this)
├── patterns.json      # Quick-access patterns (redundant with memory.db)
├── failures.json      # Quick-access failures (redundant with memory.db)
└── preferences.json   # Quick-access preferences (redundant with memory.db)
```

**Question:** Should we keep JSON files or rely entirely on memory server?

**Answer:**
- **Hybrid approach:**
  - Memory server = source of truth (rich queries)
  - JSON files = generated snapshots (quick reference, git diffs)
  - Scripts to sync: `memory.db → JSON`

**Why:**
- ✅ JSON files human-readable (good for git)
- ✅ Memory server queryable (good for AI)
- ✅ Best of both worlds

---

## Decision

**ADOPT:** Official Anthropic Memory Server (`@modelcontextprotocol/server-memory`)

**Rationale:**
1. **Critical for learning system** - No CLI alternative
2. **Official support** - Maintained by Anthropic
3. **Proven technology** - Part of official MCP reference
4. **Aligns with goals** - Enables Principle 4 (LEARN)
5. **Minimal overhead** - Single MCP server

**Next Steps:**
1. Install memory server
2. Configure in Claude Code
3. Design knowledge schema (entities, relations)
4. Integrate with learner agent
5. Test with initial learnings
6. Measure effectiveness (Principle 3: VALIDATE)

---

## Related Research

- [GitHub Integration](./github-integration.md)
- [VSCode Integration](./vscode-integration.md)
- [Database Servers](./database-servers.md)

**Research Date:** 2025-11-03
**Status:** ✅ Decision made - Install official Anthropic memory server
