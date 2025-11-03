# Token Optimization Techniques

**Last Updated**: November 3, 2025

---

## Key Finding

From Anthropic's multi-agent research:

> **Token usage explains 80% of performance variance**

Translation: **More strategic token usage = Better results**

---

## Core Token Economics

### Cost Structure

- **Input tokens**: 1× cost (baseline)
- **Output tokens**: ~4× cost (more expensive)
- **Cached input tokens**: 0.25× cost (75% savings!)

**Implication**: Optimize outputs first, cache inputs where possible

### Usage Multipliers

From research:
- **Chat interaction**: 1× baseline
- **Single agent**: 4× baseline
- **Multi-agent system**: 15× baseline

**Implication**: Use multi-agent only for high-value tasks

---

## Optimization Techniques

### 1. Prompt Caching

**Strategy**: Move unchanging content to the top

```
❌ BAD: No caching structure
User query: "Implement Firebase auth"
System prompt: "You are a helpful assistant..."
Project context: "This is a TypeScript project..."
Skills loaded: [firebase-auth skill content]
---
Nothing cached, full token cost every time

✅ GOOD: Designed for caching
[CACHEABLE STATIC CONTENT - TOP]
System prompt: "You are a helpful assistant..."
Project context: "This is a TypeScript project..."
Core rules: [.cursorrules content]
---
[DYNAMIC CONTENT - BOTTOM]
Skills loaded: [firebase-auth skill content]
User query: "Implement Firebase auth"
---
Static content cached at 75% discount!
```

**Impact**: 75% cost reduction on static content

### 2. Progressive Disclosure (Skills)

**Strategy**: Load only what's needed, when needed

```
Without Progressive Disclosure:
All patterns loaded: 3,000 tokens (always)

With Progressive Disclosure:
Metadata: 50 tokens (always)
Skill activated: +400 tokens (when needed)
Resource loaded: +200 tokens (when needed)
Total: 650 tokens (78% savings!)
```

**Implementation**:
- Keep SKILL.md lean (100 lines)
- Split into referenced files
- Load files progressively

### 3. Tool Filtering

**Strategy**: Only include relevant tools in context

```
❌ BAD: All 50 tools always included
- Every tool definition → ~50 tokens
- 50 tools × 50 tokens = 2,500 tokens
- Included even when not needed

✅ GOOD: Filter tools based on task
User: "Review code for security"
Relevant tools: Read, Grep, Glob (3 tools)
Cost: 3 × 50 = 150 tokens (94% savings!)
```

**Implementation**:
```typescript
// Filter tools based on agent role
agents: {
  'security-reviewer': {
    tools: ['Read', 'Grep', 'Glob'],  // Read-only
    // Not including: Edit, Write, Bash, etc.
  }
}
```

### 4. Output Token Management

**Strategy**: Return summaries, not raw data

```
❌ BAD: Full data dump
{
  "files_analyzed": 47,
  "full_content_of_each_file": "...(45,000 tokens)...",
  "every_single_finding": "...(15,000 tokens)..."
}
Total output: 60,000 tokens × 4 = 240,000 token-cost equivalent!

✅ GOOD: Condensed summary
{
  "files_analyzed": 47,
  "critical_issues": [
    { "file": "auth.ts:142", "issue": "SQL injection", "fix": "Use parameterized queries" },
    { "file": "api.ts:67", "issue": "Missing auth check", "fix": "Add middleware" }
  ],
  "summary": "2 critical issues found requiring immediate attention"
}
Total output: 200 tokens × 4 = 800 token-cost equivalent (99.7% savings!)
```

### 5. Context Compaction

**Strategy**: Summarize conversation history

```
Without Compaction:
Turn 1: 1,000 tokens
Turn 2: 1,500 tokens
Turn 3: 2,000 tokens
Turn 4: 2,500 tokens
Cumulative: 7,000 tokens in context

With Compaction (after turn 3):
Turn 1-3 Summary: 500 tokens
Turn 4: 2,500 tokens
Total: 3,000 tokens (57% savings!)
```

**When to compact**:
- After every 3-5 turns
- When approaching context limit (150k/200k tokens)
- Before delegating to subagent

**What to preserve**:
- Key decisions made
- Current task state
- Critical information

**What to summarize**:
- Tool call details
- Intermediate exploration
- Redundant information

### 6. Just-in-Time Context

**Strategy**: Retrieve on-demand instead of pre-loading

```
❌ BAD: Pre-load everything
Load all patterns: 5,000 tokens
Load all examples: 3,000 tokens
Load all docs: 2,000 tokens
Total: 10,000 tokens (most unused!)

✅ GOOD: Load as needed
Start: Lightweight identifiers only (50 tokens)
When needed: Load specific pattern (200 tokens)
Total: 250 tokens for typical task (97.5% savings!)
```

**Implementation**:
- Store lightweight references
- Use tools to fetch on-demand
- Cache recently accessed items

### 7. Subagent Isolation

**Strategy**: Return summaries to orchestrator

```
Subagent Context (isolated):
Task: Analyze 50 files for security issues
Working context: 50,000 tokens
(Explores files, runs analysis, finds issues)

Returns to Orchestrator:
Summary: "Found 3 critical issues: [list]"
Return size: 500 tokens (99% reduction!)

Orchestrator Context:
Doesn't include 50,000 token exploration
Only receives 500 token summary
Stays focused and efficient
```

### 8. Conditional System Prompt Sections

**Strategy**: Include sections only when relevant tools present

```typescript
// Dynamic system prompt construction
function buildSystemPrompt(toolsAvailable: string[]): string {
  let prompt = BASE_PROMPT;

  if (toolsAvailable.includes('Bash')) {
    prompt += BASH_GUIDELINES;
  }

  if (toolsAvailable.includes('Edit')) {
    prompt += EDITING_GUIDELINES;
  }

  // Only include guidelines for available tools
  return prompt;
}

// Security reviewer (Read-only)
toolsAvailable: ['Read', 'Grep', 'Glob']
System prompt: 1,000 tokens (no Bash or Edit guidelines)

// Implementer (Full access)
toolsAvailable: ['Read', 'Edit', 'Write', 'Bash', 'Grep', 'Glob']
System prompt: 1,500 tokens (includes all guidelines)
```

### 9. Task Sequencing and Context Accumulation 🔴 CRITICAL

**Problem**: Each conversation turn includes ALL previous context

```
Context Accumulation:
Turn 1: System (1k) + User (100) = 1.1k tokens
Turn 2: Previous (1.1k) + AI response (5k) + User (100) = 6.2k tokens
Turn 3: Previous (6.2k) + AI response (10k) + User (100) = 16.3k tokens
Turn 4: Previous (16.3k) + AI response (8k) + User (100) = 24.4k tokens

Each message carries forward ALL previous tokens!
```

**Implication**: **Order of operations matters significantly**

#### Bad Order: Research First

```
Task: 1) Research topic (generates 10k token report)
      2) Change 5 files (generates 3k tokens)
      3) More research (generates 8k tokens)

❌ BAD SEQUENCE:

Turn 1: Do research 1
  Context: 1k (baseline)
  Output: 10k tokens (research results)
  Cost: 1k input + 40k output-equivalent = 41k

Turn 2: Change files (WITH 10k research in context)
  Context: 11k (baseline + research)
  Output: 3k tokens
  Cost: 11k input + 12k output-equivalent = 23k

Turn 3: Do research 2 (WITH everything in context)
  Context: 14k (baseline + research1 + file changes)
  Output: 8k tokens
  Cost: 14k input + 32k output-equivalent = 46k

TOTAL COST: 41k + 23k + 46k = 110k token-equivalents
```

#### Good Order: Defer Heavy Context

```
✅ GOOD SEQUENCE:

Turn 1: Change files (lightweight operation first)
  Context: 1k (baseline)
  Output: 3k tokens
  Cost: 1k input + 12k output-equivalent = 13k

Turn 2: Do research 1 (WITH only file changes in context)
  Context: 4k (baseline + file changes)
  Output: 10k tokens
  Cost: 4k input + 40k output-equivalent = 44k

Turn 3: Do research 2 (WITH everything in context)
  Context: 14k (all previous)
  Output: 8k tokens
  Cost: 14k input + 32k output-equivalent = 46k

TOTAL COST: 13k + 44k + 46k = 103k token-equivalents
SAVINGS: 7k token-equivalents (6.4% reduction)
```

#### Best Order: Use Subagents for Heavy Operations

```
✅ OPTIMAL SEQUENCE:

Turn 1: Change files (lightweight)
  Context: 1k
  Output: 3k tokens
  Cost: 1k input + 12k output-equivalent = 13k

Turn 2: Spawn research subagent 1 (isolated context)
  Subagent context: 50k tokens (isolated, then discarded)
  Returns to main: 1k token summary
  Main context: 4k + 1k = 5k
  Cost: 4k input + 4k output-equivalent = 8k

Turn 3: Spawn research subagent 2 (isolated context)
  Subagent context: 40k tokens (isolated, then discarded)
  Returns to main: 1k token summary
  Main context: 5k + 1k = 6k
  Cost: 5k input + 4k output-equivalent = 9k

TOTAL COST (main context): 13k + 8k + 9k = 30k token-equivalents
SAVINGS: 80k token-equivalents (73% reduction!)

Note: Subagents have their own costs (15× overhead)
But context stays clean for main agent
```

#### Task Classification

**Heavy Context Operations** (defer or use subagents):
- 📚 **Research**: Web searches, reading many sources
- 📁 **File exploration**: Reading dozens of files
- 🔍 **Analysis**: Complex code analysis across files
- 📊 **Data processing**: Large dataset operations
- 📝 **Report generation**: Long-form content

**Lightweight Operations** (do early):
- ✏️ **File edits**: Changing specific files
- 🔧 **Configuration**: Updating configs
- 🚀 **Simple implementations**: Single-file changes
- ✅ **Quick validations**: Running tests
- 📋 **List operations**: Getting file lists

#### Sequencing Strategy

**Rule 1: Light before Heavy**

```
Order: Lightweight ops → Heavy ops
Reason: Heavy ops generate lots of tokens that accumulate
```

**Rule 2: Use Subagents for Heavy**

```
Heavy operation? → Subagent (isolated context, return summary)
Reason: Main context stays clean
```

**Rule 3: Use External Memory**

```
Heavy result needed later? → Save to file, reference later
Reason: Don't keep in active context
```

#### Example: Complex Feature Implementation

```
Task: Implement auth system
  - Research Firebase auth patterns
  - Research security best practices
  - Implement login/signup
  - Implement session management
  - Write tests
  - Write documentation

❌ BAD ORDER:
1. Research Firebase (10k tokens) → Context: 10k
2. Research security (8k tokens) → Context: 18k
3. Implement login (5k tokens) → Context: 23k
4. Implement session (4k tokens) → Context: 27k
5. Write tests (3k tokens) → Context: 30k
6. Write docs (5k tokens) → Context: 35k
Cumulative context: 143k token-turns

✅ GOOD ORDER:
1. Implement login (5k tokens) → Context: 5k
2. Implement session (4k tokens) → Context: 9k
3. Write tests (3k tokens) → Context: 12k
4. Research Firebase via subagent → Returns 1k summary → Context: 13k
5. Research security via subagent → Returns 1k summary → Context: 14k
6. Write docs (5k tokens) → Context: 19k
Cumulative context: 72k token-turns (50% savings!)
```

#### Decision Framework

```
For each task in workflow:
  1. Classify: Heavy or Lightweight?
  2. Ask: Will results be needed in later steps?
  3. Ask: Can this run in isolated context?

Decision:
  If Heavy + Needed Later + Can Isolate:
    → Use Subagent (return summary only)

  If Heavy + Not Needed Later:
    → Defer to end of workflow

  If Lightweight:
    → Do early in workflow

  If Heavy + Results Must Stay in Context:
    → Defer as late as possible
```

#### External Memory Pattern

**Problem**: Some heavy operations produce results needed throughout

**Solution**: Save to external file, reference as needed

```typescript
// Instead of keeping in context:

// ❌ BAD: Keep research in context
Turn 1: Research Firebase patterns
  Output: 10k tokens (stays in context forever)
Turn 2: Use research to implement
  Context: 10k + implementation
Turn 3: Use research for tests
  Context: 10k + implementation + tests

// ✅ GOOD: Save to external file
Turn 1: Research Firebase patterns (via subagent)
  Subagent: 50k tokens (isolated)
  Returns: 1k summary + "Saved to .ai-working/firebase-research.md"
  Main context: 1k

Turn 2: Implement
  Main: "See .ai-working/firebase-research.md for patterns"
  Load relevant section as needed: +2k tokens
  Context: 1k + 2k + implementation

Turn 3: Write tests
  Main: "See .ai-working/firebase-research.md for patterns"
  Load relevant section: +1k tokens
  Context: Previous + 1k + tests

Result: Research not permanently in context
Load only relevant parts as needed
Massive context savings
```

#### Practical Guidelines

**1. Plan Sequence Before Starting**

```typescript
// Analyze task workflow
function planSequence(tasks: Task[]): Task[] {
  const lightweight = tasks.filter(t => t.tokenEstimate < 5000);
  const heavy = tasks.filter(t => t.tokenEstimate >= 5000);

  return [
    ...lightweight.sort(byPriority),  // Do lightweight first
    ...heavy.sort(byTokenDescending)  // Then heavy (biggest last)
  ];
}
```

**2. Use Subagents for Research**

```
Research task?
  → ALWAYS use subagent
  → Return summary only
  → Save details to file if needed later
```

**3. Batch Similar Operations**

```
// ✅ GOOD: Batch file changes
Change files 1-10 in one operation
Then move to next phase

// ❌ BAD: Interleave operations
Change file 1
Research
Change file 2
Research
Change file 3
```

**4. Monitor Context Growth**

```typescript
// Track context size per turn
Turn 1: 5k tokens
Turn 2: 12k tokens (+7k)
Turn 3: 35k tokens (+23k) ⚠️ Heavy growth!
  → Consider if Turn 3 should be subagent

Threshold: If single turn adds >10k tokens, consider subagent
```

---

## Advanced Techniques

### Parallel Tool Calling

**Strategy**: Execute multiple tools simultaneously

```
Sequential:
Step 1: Read file A (5 seconds)
Step 2: Read file B (5 seconds)
Step 3: Read file C (5 seconds)
Total: 15 seconds

Parallel:
Step 1: Read A, B, C simultaneously (5 seconds)
Total: 5 seconds (67% time savings!)

Same token cost, much faster
```

**When to use**:
- Multiple independent reads
- Multiple API calls
- Multiple file searches

### GraphQL for Data Fetching

**Strategy**: Request only needed fields

```typescript
// ❌ BAD: REST API returns everything
GET /api/users
Returns: { id, email, name, address, phone, preferences, history, ... }
Token cost: ~100 tokens per user × 50 users = 5,000 tokens

// ✅ GOOD: GraphQL returns only what's needed
query {
  users {
    id
    email
  }
}
Token cost: ~20 tokens per user × 50 users = 1,000 tokens (80% savings!)
```

### Structured Note-Taking

**Strategy**: Maintain external memory files

```
Instead of keeping everything in context:

Create NOTES.md:
## Current Task
Implementing Firebase authentication

## Decisions Made
- Using email/password auth (not OAuth)
- Storing tokens in httpOnly cookies
- Session timeout: 30 days

## Next Steps
1. Implement signup endpoint
2. Add validation middleware
3. Test error handling

Agent context:
"See NOTES.md for current state"
Cost: ~20 tokens (reference) vs 500 tokens (full content)
```

---

## Measurement & Monitoring

### Metrics to Track

```
Per Task:
- Input tokens
- Output tokens
- Cached tokens
- Tool calls
- Context window usage %
- Time to completion

Per Agent/Subagent:
- Average tokens per invocation
- Token efficiency (results per token)
- Context window utilization
- Frequency of activation

Per Skill:
- Load frequency
- Tokens when loaded
- Success rate when used
- Time in context
```

### Optimization Targets

From our project goals:

- **60%+ token reduction** vs monolithic approach
- **< 50k tokens** for typical task
- **< 150k tokens** for complex multi-agent task
- **75%+ prompt caching** hit rate for static content

---

## Token Budgets

### Budget Allocation

```
Total Context Window: 200k tokens

Reserved (always):
- System prompt: 1,000 tokens
- Core rules: 200 tokens
- Project context: 300 tokens
- Tool definitions: 500 tokens
Subtotal: 2,000 tokens (1%)

Dynamic (as needed):
- Skill metadata: 50 tokens
- Active skills: 400-800 tokens
- Conversation: 5,000-10,000 tokens
- Working memory: 10,000-20,000 tokens
Subtotal: ~30,000 tokens (15%)

Available for work: 168,000 tokens (84%)
```

### Budget Guidelines

- **Simple tasks**: Target < 20k tokens total
- **Medium tasks**: Target < 50k tokens total
- **Complex tasks**: Target < 100k tokens total
- **Multi-agent**: Budget 15× baseline (e.g., 50k → 750k)

---

## Practical Examples

### Example 1: Code Review Optimization

```
❌ UNOPTIMIZED:
1. Load all review guidelines (2,000 tokens)
2. Load all code files (50,000 tokens)
3. Perform comprehensive analysis
4. Return full analysis (20,000 tokens output)
Total: 72,000 tokens input + 80,000 token-cost-equivalent output

✅ OPTIMIZED:
1. Load review metadata (50 tokens)
2. Activate security-review skill (400 tokens)
3. Use Grep to find security-relevant files (10 files, 5,000 tokens)
4. Perform targeted analysis
5. Return critical findings only (500 tokens output)
Total: 5,450 tokens input + 2,000 token-cost-equivalent output
Savings: 93% tokens, 96% cost!
```

### Example 2: Multi-File Refactoring

```
❌ SEQUENTIAL (Single Agent):
For each of 50 files:
  - Load file (500 tokens)
  - Analyze dependencies (in context)
  - Make changes
  - Context grows with each file
Total: 25,000+ tokens cumulative context

✅ PARALLEL (Multi-Agent):
Coordinator:
  - Creates refactoring plan (2,000 tokens)
  - Spawns 50 subagents
Each subagent (isolated context):
  - Receives specific file + instructions (500 tokens)
  - Makes changes
  - Returns result (100 tokens)
Coordinator receives summaries: 50 × 100 = 5,000 tokens

Coordinator context: 2,000 + 5,000 = 7,000 tokens (72% savings!)
```

### Example 3: Research Task

```
❌ SINGLE CONTEXT:
- Explore topic A (10,000 tokens)
- Explore topic B (10,000 tokens)
- Explore topic C (10,000 tokens)
- Explore topic D (10,000 tokens)
Cumulative context: 40,000 tokens
All explorations competing for attention

✅ SUBAGENT ISOLATION:
Orchestrator (2,000 tokens):
  - Breaks into 4 research directions
  - Spawns 4 subagents

Each Subagent (separate context):
  - Explores assigned topic (10,000 tokens)
  - Returns 1,000 token summary

Orchestrator receives: 4,000 tokens summaries
Orchestrator context: 2,000 + 4,000 = 6,000 tokens
Each subagent: 10,000 tokens (isolated, then discarded)

Result: Better focused orchestrator (85% smaller context)
```

---

## Key Takeaways

1. **Token usage = 80% of performance variance**

2. **Prompt caching = 75% savings** on static content

3. **Progressive disclosure (skills) = 60-70% savings** vs monolithic

4. **Output tokens = 4× cost** - Always optimize outputs first

5. **Multi-agent = 15× tokens** - Use judiciously

6. **Subagent isolation** prevents context pollution

7. **Tool filtering** reduces unnecessary definitions

8. **Just-in-time loading** beats pre-loading

9. **Parallel execution** same cost, much faster

10. **Task sequencing matters** 🔴 - Light operations before heavy (6-73% savings)

11. **Context accumulation is real** - Each turn carries ALL previous tokens

12. **Measure everything** - Track tokens per task

---

## Implementation Checklist

- [ ] Design prompts for caching (static content at top)
- [ ] Implement progressive disclosure for skills
- [ ] Filter tools based on agent role
- [ ] Return summaries, not full dumps
- [ ] Implement context compaction after N turns
- [ ] Use just-in-time resource loading
- [ ] Ensure subagents return lightweight summaries
- [ ] Conditional system prompt sections
- [ ] **Plan task sequences** (lightweight before heavy)
- [ ] **Classify tasks** as heavy or lightweight
- [ ] **Use subagents for heavy operations** (research, file exploration)
- [ ] **Use external memory** for results needed throughout
- [ ] Track token usage per task AND cumulative per conversation
- [ ] Set token budgets for different task types
- [ ] Monitor context growth per turn (threshold: >10k = consider subagent)

---

## Further Reading

- [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md) - Progressive disclosure
- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Context isolation
- [05-ORCHESTRATION-PATTERNS.md](05-ORCHESTRATION-PATTERNS.md) - Multi-agent efficiency
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive guidelines
