# Agent Granularity & System Limitations

**Last Updated**: November 3, 2025
**Status**: 🔴 CRITICAL - Design decisions required

---

## Table of Contents

1. [The Agent Bloat Problem](#the-agent-bloat-problem)
2. [Critical Limitation: No Nested Subagents](#critical-limitation-no-nested-subagents)
3. [Agent vs Skill Decision Framework](#agent-vs-skill-decision-framework)
4. [Context Pollution: The Air Traffic Controller Analogy](#context-pollution-the-air-traffic-controller-analogy)
5. [Map/Reduce Patterns for Subagents](#mapreduce-patterns-for-subagents)
6. [Design Patterns Within Constraints](#design-patterns-within-constraints)
7. [Workarounds and Limitations](#workarounds-and-limitations)

---

## The Agent Bloat Problem

### Real-World Example: ruvnet/claude-flow

**Observation**: claude-flow has dozens of agents, many of which are essentially:

```markdown
---
name: some-expert
description: Expert in X
---

You are an expert in X.
[No specific guidance]
[No proven patterns]
[No concrete instructions]
```

**Problem**: Token bloat with minimal benefit

### The Issue with "Expert" Prompts

```
❌ BAD: Superficial "expert" agent
---
name: security-expert
---
You are a security expert.
Review code for security issues.

Token cost: 500 tokens (agent definition + invocation overhead)
Actual value: ~0 (Claude already knows security basics)
Benefit: Minimal to none

✅ GOOD: Skill with concrete patterns
skills/security-review/SKILL.md:
- Specific vulnerability patterns to check
- Code examples of vulnerabilities
- Remediation steps
- OWASP references
- Project-specific security rules

Token cost: 50 tokens (metadata) until needed, then 400 tokens (SKILL.md)
Actual value: High (concrete, actionable guidance)
Benefit: Significant
```

### Why Agent Proliferation Happens

**Misconceptions**:
1. "More agents = more specialized = better results" ❌
2. "Each domain needs its own agent" ❌
3. "Agents make Claude smarter in that domain" ❌

**Reality**:
1. Claude already has broad knowledge
2. What Claude needs: **Specific patterns and context**
3. Agents without substance add overhead, not value

---

## Critical Limitation: No Nested Subagents

### ⚠️ IMPORTANT CONSTRAINT

**Subagents CANNOT call other subagents**

```
❌ NOT POSSIBLE:
Main Agent
  └─→ Subagent A
       └─→ Subagent B  // NOT ALLOWED
            └─→ Subagent C  // NOT ALLOWED
```

**Source**: User research, confirmed through implementation testing

**Last Verified**: November 3, 2025

**Re-evaluation Schedule**: Check quarterly with Anthropic updates

### Why This Limitation Exists

**Likely reasons**:
1. **Complexity management**: Prevent infinite recursion
2. **Context tracking**: Difficult to manage nested context windows
3. **Cost control**: Exponential token consumption
4. **Error handling**: Cascading failures in deep hierarchies

### Architectural Implications

**Only allowed pattern**:
```
✅ VALID:
Main Agent (Orchestrator)
  ├─→ Subagent A (Worker)
  ├─→ Subagent B (Worker)
  ├─→ Subagent C (Worker)
  └─→ Subagent D (Worker)
```

**Result**: Flat hierarchy only, **maximum 2 levels**

---

## Agent vs Skill Decision Framework

### Decision Tree

```
Question 1: Does it need ISOLATED CONTEXT?
├─ YES → Consider Agent
│   └─ Question 2: Does it need to DO WORK in that context?
│       ├─ YES → Use Subagent
│       │   └─ Question 3: Is the work context-heavy (read many files, web research)?
│       │       ├─ YES → DEFINITELY use Subagent (prevents context pollution)
│       │       └─ NO → Could be either, lean toward Subagent if parallelizable
│       └─ NO → Use Skill (just needs instructions)
└─ NO → Use Skill
    └─ Question 4: Is it domain knowledge or patterns?
        ├─ Knowledge/Patterns → Skill
        └─ Actual execution needed → Reconsider if Agent needed
```

### When to Use a SKILL

**Use Skills for**:
- ✅ Domain-specific **knowledge** (Firebase patterns, security rules)
- ✅ **Instructions** and best practices
- ✅ Code **examples** and templates
- ✅ **Reference** information
- ✅ **Checklists** and procedures
- ✅ **No execution** needed, just guidance

**Example - Security Review Skill**:
```markdown
skills/security-review/SKILL.md:
- Common vulnerability patterns
- OWASP Top 10 checklist
- Secure code examples
- Testing procedures
- Remediation templates

Claude uses this knowledge IN MAIN CONTEXT
No separate execution needed
Progressive disclosure: Load only relevant sections
```

**Token Efficiency**:
```
Baseline: 5 tokens (metadata)
When activated: 400 tokens (SKILL.md)
Specific pattern: +200 tokens (as needed)
Total: 605 tokens maximum
```

### When to Use a SUBAGENT

**Use Subagents for**:
- ✅ **Context-heavy work** (reading many files, web research)
- ✅ **Isolated execution** (prevents main agent pollution)
- ✅ **Parallelizable tasks** (multiple independent operations)
- ✅ **Map/reduce operations** (explore → summarize → return)
- ✅ **High-value, complex tasks** (justifies 15× overhead)

**Example - Research Subagent**:
```markdown
Main Agent:
"Research 5 different topics"

Spawns 5 subagents (parallel):
Each subagent:
  - Reads 20+ web pages (context: 50k tokens)
  - Analyzes findings
  - Returns 1k token summary

Main Agent:
  - Receives 5 summaries (5k tokens total)
  - Context NOT polluted with 250k tokens of exploration
  - Synthesizes findings
```

**Token Efficiency**:
```
Without subagents:
Main context: 250k tokens (5 topics × 50k exploration)
Overwhelmed, can't synthesize

With subagents:
Main context: 5k tokens (5 × 1k summaries)
Clean, focused, effective synthesis
```

### Examples: Agent vs Skill

| Scenario | Use | Reasoning |
|----------|-----|-----------|
| Firebase auth patterns | **Skill** | Just needs patterns/examples, no isolated work |
| "You are a security expert" | **Skill** | Generic expertise adds no value, use skill with patterns |
| Review 50 files for security | **Subagent** | Context-heavy, prevents pollution |
| Code review pipeline | **Subagent** | Parallel execution, isolated analysis |
| Testing best practices | **Skill** | Knowledge/patterns, no execution needed |
| Implement feature in 10 files | **Subagent** | Parallelizable, high-value work |
| OWASP Top 10 checklist | **Skill** | Reference information |
| Research 5 topics on web | **Subagent** | Context-heavy exploration, map/reduce |
| Refactoring patterns | **Skill** | Techniques/examples, no isolated work |
| Coordinate sub-tasks | **Subagent (Orchestrator)** | Delegation and synthesis |

---

## Context Pollution: The Air Traffic Controller Analogy

### The Analogy

```
Main Agent = Air Traffic Controller
Subagents = Pilots

Air Traffic Controller:
- Sees high-level picture
- Coordinates traffic
- Makes strategic decisions
- Does NOT fly the planes
- Does NOT get bogged down in flight details

If controller tries to fly planes AND coordinate:
→ Overwhelmed
→ Loses strategic view
→ Makes poor decisions
→ System breaks down
```

### Context Pollution in Practice

```
❌ BAD: Main agent does context-heavy work

Main Agent context:
1. Read file1.ts (5k tokens)
2. Read file2.ts (7k tokens)
3. Read file3.ts (6k tokens)
... [repeats 47 more times]
50. Read file50.ts (4k tokens)

Total context: 250k tokens
Result: Context overwhelmed, can't think strategically
Performance: Degraded
Quality: Poor decisions

✅ GOOD: Subagent does context-heavy work

Main Agent context:
1. "Analyze these 50 files for security issues"
2. Spawn security-analyzer subagent

Subagent context (isolated):
- Read all 50 files (250k tokens in subagent)
- Analyze thoroughly
- Return: "Found 3 critical issues: [summary]" (1k tokens)

Main Agent context: 1k tokens (clean!)
Result: Can think strategically
Performance: Excellent
Quality: Good decisions
```

### Rules for Main Agent

**Main Agent should NEVER**:
- ❌ Read dozens of files into its context
- ❌ Load multiple web pages
- ❌ Perform extensive exploration
- ❌ Maintain large working memory
- ❌ Do detailed implementation work

**Main Agent SHOULD**:
- ✅ Coordinate and delegate
- ✅ Make strategic decisions
- ✅ Synthesize subagent summaries
- ✅ Maintain high-level task state
- ✅ Track progress and exceptions

### Context Budget Guidelines

```
Main Agent Context Budget:
- System prompt: 1k tokens
- Core rules: 200 tokens
- Active skills metadata: 50 tokens
- Task state: 500 tokens
- Subagent summaries: 5-10k tokens
- Strategic thinking: 5-10k tokens
TOTAL: ~20k tokens maximum

Reserved for: Coordination and decision-making

Subagent Context Budget:
- Subagent prompt: 1k tokens
- Loaded skills: 400 tokens
- Working memory: 50-150k tokens
- Exploration/implementation
TOTAL: Up to 150k tokens

Reserved for: Heavy lifting
```

---

## Map/Reduce Patterns for Subagents

### The Pattern

```
Main Agent (Coordinator)
  │
  ├─ MAP PHASE: Spawn subagents
  │   ├─→ Subagent 1: Process chunk 1 (isolated context)
  │   ├─→ Subagent 2: Process chunk 2 (isolated context)
  │   ├─→ Subagent 3: Process chunk 3 (isolated context)
  │   └─→ Subagent 4: Process chunk 4 (isolated context)
  │
  ├─ Each subagent:
  │   1. Does context-heavy work
  │   2. Returns lightweight summary
  │
  └─ REDUCE PHASE: Main agent
      - Receives all summaries
      - Synthesizes results
      - Makes decisions
```

### Example 1: Security Review of 50 Files

```typescript
// Main Agent (Orchestrator)
async function coordinateSecurityReview(files: string[]) {
  // MAP: Divide files into chunks
  const chunks = chunkArray(files, 10); // 5 chunks of 10 files

  // MAP: Spawn parallel subagents
  const results = await Promise.all(
    chunks.map((chunk, i) =>
      invokeSubagent('security-analyzer', {
        files: chunk,
        chunkId: i,
        instructions: `
          Review these 10 files for security issues.
          Focus on OWASP Top 10.
          Return ONLY critical/high findings.
          Format: { file, line, issue, severity, fix }
        `
      })
    )
  );

  // REDUCE: Synthesize results
  const allFindings = results.flatMap(r => r.findings);
  const critical = allFindings.filter(f => f.severity === 'critical');

  return {
    summary: `Reviewed ${files.length} files, found ${critical.length} critical issues`,
    criticalIssues: critical,
    recommendation: determinePriority(critical)
  };
}

// Subagent (Worker) - Each handles 10 files
// Context: Isolated, 50k tokens of file reading
// Returns: 1k token summary
// Main agent receives: 5k tokens total (5 summaries)
```

### Example 2: Multi-Topic Research

```typescript
// Main Agent
async function coordinateResearch(topics: string[]) {
  // MAP: Spawn parallel researchers
  const results = await Promise.all(
    topics.map(topic =>
      invokeSubagent('researcher', {
        topic,
        instructions: `
          Research: ${topic}
          - Web search for authoritative sources
          - Read top 10 results
          - Extract key insights
          - Return 500-word summary with sources
        `
      })
    )
  );

  // REDUCE: Synthesize into comprehensive report
  return synthesizeResearch(results);
}

// Each researcher subagent:
// - Loads 10 web pages (50k tokens)
// - Analyzes thoroughly
// - Returns 500-word summary (200 tokens)
// Main agent: 5 topics × 200 tokens = 1k tokens (clean!)
```

### Example 3: Multi-File Refactoring

```typescript
// Main Agent
async function coordinateRefactoring(files: string[], pattern: RefactorPattern) {
  // MAP: One subagent per file
  const results = await Promise.all(
    files.map(file =>
      invokeSubagent('file-refactorer', {
        file,
        pattern,
        instructions: `
          Refactor ${file} according to pattern.
          Make only specified changes.
          Return: { file, changes: summary, linesModified: number }
        `
      })
    )
  );

  // REDUCE: Verify consistency
  verifyRefactoringConsistency(results);

  return {
    filesModified: results.length,
    totalLines: results.reduce((sum, r) => sum + r.linesModified, 0),
    summary: 'Refactoring complete across all files'
  };
}

// Each file-refactorer:
// - Reads one file (2k tokens)
// - Refactors in isolation
// - Returns tiny summary (50 tokens)
// Main agent: 75 files × 50 tokens = 3.75k tokens
```

### Key Principles

1. **Chunk appropriately**: Balance parallelism vs overhead
2. **Clear instructions**: Each subagent knows exactly what to do
3. **Lightweight returns**: Summaries, not full data
4. **Synthesize in main**: Main agent makes sense of results
5. **Isolated contexts**: Subagents pollute their own context, not main

---

## Design Patterns Within Constraints

### Pattern 1: Flat Orchestrator-Worker

**The only valid pattern given nesting limitation**:

```
Main Agent (Orchestrator)
  ├─→ Worker A
  ├─→ Worker B
  ├─→ Worker C
  └─→ Worker D

✅ All workers at same level
✅ No worker calls another worker
✅ Main agent coordinates everything
```

### Pattern 2: Sequential Pipelines

```
Main Agent:
  Step 1: Spawn Planner → Receive plan
  Step 2: Spawn Implementer → Receive code
  Step 3: Spawn Reviewer → Receive feedback
  Step 4: If issues, spawn Implementer again
  Step 5: Spawn Tester → Receive test results

Each subagent disposable, returns to main, main decides next step
```

### Pattern 3: Parallel Fan-Out, Serial Synthesis

```
Main Agent:
  Phase 1: Parallel
    ├─→ Subagent 1 (parallel)
    ├─→ Subagent 2 (parallel)
    └─→ Subagent 3 (parallel)
    All return to main

  Phase 2: Main synthesizes

  Phase 3: Parallel (based on synthesis)
    ├─→ Subagent 4 (parallel)
    └─→ Subagent 5 (parallel)
    All return to main

  Phase 4: Main makes final decision
```

### Pattern 4: Iterative Refinement

```
Main Agent:
  Round 1:
    Spawn Implementer → Code v1
    Spawn Reviewer → Feedback

  If feedback requires changes:
    Round 2:
      Spawn Implementer (NEW instance) → Code v2
      Spawn Reviewer (NEW instance) → Feedback

  Repeat until approved

Each subagent is fresh instance
Main agent tracks refinement history
```

### Anti-Pattern: Trying to Nest

```
❌ WILL NOT WORK:

Main Agent:
  Spawn Coordinator Subagent
    → Coordinator tries to spawn Worker Subagent
    → ERROR: Nesting not allowed

✅ CORRECT APPROACH:

Main Agent:
  Spawn Worker 1 directly
  Spawn Worker 2 directly
  Spawn Worker 3 directly
  Main agent does coordination itself
```

---

## Workarounds and Limitations

### Workaround 1: CLI Invocation (Risky)

**Concept**: Subagent calls Claude Code CLI directly

```bash
# From within subagent
claude-code --prompt "Do something" --output result.txt
```

**Problems**:
- 🔴 **Infinite spawning risk**: Recursion loops
- 🔴 **Context isolation breaks**: New CLI instance doesn't share context
- 🔴 **Cost explosion**: Each CLI call is full agent invocation
- 🔴 **Error handling**: Cascading failures
- 🔴 **Tracking**: Difficult to monitor and debug

**Verdict**: ❌ **NOT RECOMMENDED** - Risks outweigh benefits

### Workaround 2: External Task Queue

**Concept**: Subagent adds tasks to queue, main agent processes

```typescript
// Subagent
async function subagentWork() {
  // Do work
  const needsMoreWork = analyzeResults();

  if (needsMoreWork) {
    // Add to external queue
    await taskQueue.add({
      type: 'additional-analysis',
      data: summarizedData
    });
  }

  return results;
}

// Main Agent polls queue
async function mainAgent() {
  while (true) {
    const results = await processSubagents();
    const queuedTasks = await taskQueue.getAll();

    if (queuedTasks.length > 0) {
      // Spawn new subagents for queued tasks
      processQueuedTasks(queuedTasks);
    }
  }
}
```

**Benefits**:
- ✅ Maintains flat hierarchy
- ✅ Main agent stays in control
- ✅ No infinite recursion

**Drawbacks**:
- ⚠️ Added complexity
- ⚠️ Requires external queue system
- ⚠️ More moving parts

**Verdict**: ⚠️ **Possible but complex** - Only if really needed

### Workaround 3: Iterative Coordination (Recommended)

**Concept**: Main agent does multiple rounds

```typescript
async function mainAgent(task: ComplexTask) {
  let phase = 1;
  let results = [];

  while (!isComplete(results)) {
    switch (phase) {
      case 1: // Initial exploration
        results = await spawnSubagents(explorationAgents);
        phase = 2;
        break;

      case 2: // Based on exploration, do detailed work
        const detailTasks = analyzeExploration(results);
        results = await spawnSubagents(detailAgents(detailTasks));
        phase = 3;
        break;

      case 3: // Review and refine
        const reviewResults = await spawnSubagents(reviewerAgents);
        if (reviewResults.needsWork) {
          phase = 2; // Go back to detail phase
        } else {
          break; // Done
        }
    }
  }

  return synthesize(results);
}
```

**Benefits**:
- ✅ Maintains flat hierarchy
- ✅ Main agent fully in control
- ✅ Simple to understand and debug
- ✅ No external dependencies

**Verdict**: ✅ **RECOMMENDED** - Best approach within constraints

### Workaround 4: Skills for Sub-Instructions

**Concept**: Subagents load skills for additional guidance

```typescript
// Subagent definition
{
  name: 'implementer',
  prompt: `You implement features.

  Load relevant skills as needed:
  - firebase-auth skill for auth work
  - testing skill when writing tests
  - security-review skill to self-check

  Skills provide additional patterns and guidance.`,
  tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob']
}

// Subagent can load skills (not call other subagents)
// Skills provide the "nested knowledge" without nested execution
```

**Benefits**:
- ✅ Subagents get specialized knowledge
- ✅ No nesting constraint violated
- ✅ Progressive disclosure still works

**Verdict**: ✅ **HIGHLY RECOMMENDED** - Best pattern for layered guidance

---

## Recommended Patterns for Optimized AI

### Pattern: Air Traffic Controller

```typescript
// Main Agent: Air Traffic Controller
// - Coordinates at high level
// - NEVER reads files directly (unless < 3 files)
// - NEVER does context-heavy work
// - Always delegates to subagents

async function airTrafficController(task: Task) {
  // Analyze at high level
  const strategy = analyzeTask(task);

  // Delegate context-heavy work
  if (strategy.requiresFileAnalysis && strategy.fileCount > 3) {
    // Spawn file-analyzer subagent
    const analysis = await invokeSubagent('file-analyzer', {
      files: strategy.files,
      analysis: 'security-and-quality'
    });

    // Receive lightweight summary
    // Main context stays clean
  }

  if (strategy.requiresResearch) {
    // Spawn researcher subagent
    const research = await invokeSubagent('researcher', {
      topics: strategy.topics
    });

    // Receive lightweight summary
  }

  // Synthesize and decide
  return makeDecision(analysis, research);
}
```

### Pattern: Map/Reduce Everything

```typescript
// Main Agent: Never works with raw data directly

// ❌ BAD
async function mainAgent(files: string[]) {
  for (const file of files) {
    const content = await read(file); // Pollutes context!
    analyze(content); // Context grows unbounded
  }
}

// ✅ GOOD
async function mainAgent(files: string[]) {
  const chunks = chunkArray(files, 10);

  // MAP: Parallel subagents
  const analyses = await Promise.all(
    chunks.map(chunk =>
      invokeSubagent('analyzer', {
        files: chunk,
        returnFormat: 'summary-only' // Key!
      })
    )
  );

  // REDUCE: Synthesize summaries
  return synthesize(analyses);
}
```

### Pattern: Skills in Subagents

```typescript
// Subagents load skills for domain knowledge

agents: {
  'implementer': {
    description: 'Implements features',
    prompt: `You implement features following project patterns.

Load relevant skills based on task:
- firebase-auth: For authentication work
- supabase-rls: For database security
- testing: For test creation
- refactoring: For code improvements

Skills provide specific patterns and examples.
Follow the patterns, don't reinvent.`,
    tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob'],
    model: 'sonnet'
  }
}

// Result: Subagent gets layered guidance without nesting subagents
```

---

## Tools for Subagents

### Tool: Extract Relevant Lines

**Problem**: Subagent reads large file, only needs specific sections

**Solution**: Tool to extract line ranges

```typescript
// MCP Server: Optimized File Operations
server.tool({
  name: 'read_file_with_line_numbers',
  description: 'Read file with line numbers prepended',
  handler: async ({ filePath }) => {
    const content = await readFile(filePath);
    const lines = content.split('\n');
    return lines
      .map((line, i) => `${i + 1}: ${line}`)
      .join('\n');
  }
});

server.tool({
  name: 'extract_line_range',
  description: 'Extract specific line range from file',
  handler: async ({ filePath, startLine, endLine }) => {
    const content = await readFile(filePath);
    const lines = content.split('\n');
    return lines
      .slice(startLine - 1, endLine)
      .map((line, i) => `${startLine + i}: ${line}`)
      .join('\n');
  }
});

server.tool({
  name: 'extract_function',
  description: 'Extract specific function from file',
  handler: async ({ filePath, functionName }) => {
    const content = await readFile(filePath);
    // Parse and extract function
    return extractedFunction;
  }
});
```

**Usage in Subagent**:

```typescript
// Subagent workflow
async function analyzeFunction() {
  // 1. Read with line numbers
  const fileWithLines = await readFileWithLineNumbers('auth.ts');

  // 2. Identify relevant section
  const relevantLines = identifySection(fileWithLines);
  // e.g., "Lines 142-167 contain the login function"

  // 3. Extract only that section
  const section = await extractLineRange('auth.ts', 142, 167);

  // 4. Analyze just the relevant section
  const analysis = analyzeSection(section);

  // 5. Return reference to line numbers
  return {
    findings: [
      { file: 'auth.ts', lines: '142-167', issue: '...' }
    ]
  };
}

// Context saved: Only relevant 25 lines loaded, not entire 500-line file
```

**Note**: Check if Claude already provides line numbers in Read tool output. If yes, we may not need custom tool.

---

## Design Checklist

### Before Creating an Agent

- [ ] Does this need isolated context? (If no → Skill)
- [ ] Does it do context-heavy work? (If no → Skill)
- [ ] Is it more than "you are an expert"? (If no → Skill)
- [ ] Does it provide concrete, specific guidance? (If no → Make it a Skill)
- [ ] Is the task parallelizable? (If no → Consider if Agent needed)
- [ ] Is the task high-value enough to justify 15× overhead? (If no → Skill)
- [ ] Will this agent need to spawn sub-agents? (If yes → Redesign, not possible)

### Before Creating a Skill

- [ ] Is this domain knowledge or patterns? (If yes → Skill)
- [ ] Can this be progressive disclosure? (If yes → Skill)
- [ ] Will this be reused across tasks? (If yes → Skill)
- [ ] Is execution in main context acceptable? (If no → Need Agent)

### Designing Workflows

- [ ] Main agent acts as coordinator only?
- [ ] All context-heavy work delegated to subagents?
- [ ] Subagents return lightweight summaries?
- [ ] No nested subagent calls assumed?
- [ ] Flat hierarchy maintained?
- [ ] Map/reduce pattern where appropriate?

---

## Key Takeaways

1. **Most "agents" should be Skills** - Only use Agents when isolated context needed

2. **"You are an expert" is worthless** - Provide concrete patterns and examples

3. **No nested subagents** - Critical limitation, design around it

4. **Main agent = Air traffic controller** - Never does context-heavy work

5. **Context pollution is the enemy** - Subagents protect main agent context

6. **Map/reduce everything** - Subagents explore, main agent synthesizes

7. **Flat hierarchy only** - Main agent → Subagents, that's it

8. **Skills in subagents** - Layered guidance without nesting

9. **Lightweight summaries** - Subagents return 1-2k tokens, not 50k

10. **15× overhead threshold** - Only use subagents for high-value, context-heavy work

---

## Action Items

### Immediate

1. **Audit existing agent designs** against decision framework
2. **Convert "expert" agents to skills** where appropriate
3. **Document nesting limitation** in all orchestration docs
4. **Design all workflows** with flat hierarchy
5. **Create line extraction tools** for efficient subagent work

### Phase 0

1. **Experiment: Agent vs Skill** for same scenario
2. **Measure context pollution** with/without subagent delegation
3. **Test map/reduce patterns** for multi-file operations
4. **Validate air traffic controller** model

### Documentation Updates

1. **Update 05-ORCHESTRATION-PATTERNS.md** with nesting limitation
2. **Update 01-SUBAGENTS-DEEP-DIVE.md** with context pollution analogy
3. **Update 06-BEST-PRACTICES.md** with decision framework
4. **Add this document** to reading order

---

## Re-evaluation Schedule

**Check quarterly**: Have Anthropic's capabilities changed?

- ✅ November 2025: Nesting limitation confirmed
- ⏳ February 2026: Re-check with new Claude releases
- ⏳ May 2026: Re-check
- ⏳ August 2026: Re-check

**What to look for**:
- Nested subagent support announced
- New orchestration patterns in docs
- Community workarounds that prove stable

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Subagent fundamentals
- [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md) - When to use Skills
- [05-ORCHESTRATION-PATTERNS.md](05-ORCHESTRATION-PATTERNS.md) - Coordination patterns
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive guidelines
