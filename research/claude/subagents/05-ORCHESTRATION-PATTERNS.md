# Multi-Agent Orchestration Patterns

**Last Updated**: November 3, 2025

---

## Core Pattern: Orchestrator-Worker

From Anthropic's research system and community implementations, the **orchestrator-worker pattern** is the proven standard for multi-agent systems.

---

## Architecture

### Basic Structure

```
┌─────────────────────────────┐
│   Orchestrator (Lead Agent) │
│   - Plans strategy          │
│   - Delegates tasks         │
│   - Synthesizes results     │
│   - Makes decisions         │
└──────────┬──────────────────┘
           │
           ├─────────────┬──────────────┬──────────────┐
           │             │              │              │
           ▼             ▼              ▼              ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ Worker 1 │  │ Worker 2 │  │ Worker 3 │  │ Worker 4 │
    │ (Planner)│  │(Implement│  │(Reviewer)│  │ (Tester) │
    └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

### Roles

#### Orchestrator (Lead Agent)

**Responsibilities**:
- ✅ Analyze overall task
- ✅ Break into subtasks
- ✅ Delegate to appropriate workers
- ✅ Track progress
- ✅ Synthesize worker outputs
- ✅ Make final decisions
- ✅ Handle exceptions

**Does NOT**:
- ❌ Perform detailed implementation
- ❌ Execute low-level tasks
- ❌ Maintain large working contexts

**Model**: Claude Opus 4 (best reasoning)

#### Workers (Subagents)

**Responsibilities**:
- ✅ Execute assigned subtask
- ✅ Work in isolated context
- ✅ Use specialized knowledge
- ✅ Return concise results

**Does NOT**:
- ❌ Make global decisions
- ❌ Coordinate with other workers
- ❌ Understand full task context

**Model**: Claude Sonnet 3.5 (good balance)

---

## Coordination Topologies

### 1. Hierarchical (Recommended for Most Cases)

```
        Orchestrator (Opus)
             │
   ┌─────────┼─────────┐
   │         │         │
Worker 1  Worker 2  Worker 3
(Sonnet)  (Sonnet)  (Sonnet)
```

**When to use**:
- Clear task decomposition
- Independent subtasks
- Centralized decision-making needed

**Benefits**:
- Simple coordination
- Clear responsibilities
- Easy to debug
- Proven pattern

**Example**: Code review pipeline
- Orchestrator: Coordinates review
- Worker 1: Security review
- Worker 2: Quality review
- Worker 3: Performance review

### 2. Pipeline (Sequential Processing)

```
Worker 1 → Worker 2 → Worker 3 → Worker 4
(Plan)     (Implement) (Test)     (Document)
```

**When to use**:
- Tasks must be sequential
- Output of one feeds next
- Handoff points are clear

**Benefits**:
- Clear workflow
- Each stage isolated
- Easy to optimize each step

**Example**: Feature implementation
- Planner: Creates specification
- Implementer: Writes code
- Tester: Creates tests
- Documenter: Writes docs

### 3. Star (Parallel Execution)

```
         Orchestrator
         │
    ┌────┼────┬────┬────┐
    │    │    │    │    │
    W1   W2   W3   W4   W5
    │    │    │    │    │
    └────┼────┴────┴────┘
         │
    Orchestrator
```

**When to use**:
- Tasks are fully independent
- Parallel execution beneficial
- Need aggregate results

**Benefits**:
- Maximum parallelization
- Fast execution
- Clean isolation

**Example**: Research task
- Orchestrator: Defines research directions
- W1-W5: Research different aspects in parallel
- Orchestrator: Synthesizes findings

### 4. Iterative (Worker-Coordinator Loop)

```
Orchestrator ←→ Worker
     ↓
  Refine
     ↓
Orchestrator ←→ Worker
     ↓
  Complete
```

**When to use**:
- Iterative refinement needed
- Feedback loops required
- Quality improvement through cycles

**Benefits**:
- Progressive improvement
- Feedback integration
- Adaptive approach

**Example**: Code implementation with review
- Implementer: Writes code v1
- Reviewer: Provides feedback
- Implementer: Refines code v2
- Reviewer: Approves or requests changes
- Repeat until accepted

---

## Communication Patterns

### 1. Lightweight Summaries

**Pattern**: Workers return condensed results

```typescript
// ❌ BAD: Worker returns everything
{
  "exploration": "...(20,000 tokens)...",
  "all_findings": "...(15,000 tokens)...",
  "full_analysis": "...(10,000 tokens)..."
}

// ✅ GOOD: Worker returns summary
{
  "summary": "Analyzed 47 files, found 3 critical issues",
  "critical_issues": [
    { "file": "auth.ts:142", "issue": "SQL injection", "priority": "critical" }
  ],
  "recommendation": "Fix critical issues before deployment"
}
```

### 2. External Memory

**Pattern**: Workers store detailed work externally

```typescript
// Worker performs extensive exploration
// Instead of returning all to orchestrator:

// 1. Worker saves detailed findings to file
writeFile('.ai-working/research-findings.md', detailedFindings);

// 2. Worker returns reference to orchestrator
return {
  "summary": "Research complete, 47 sources analyzed",
  "details_location": ".ai-working/research-findings.md",
  "key_insights": [...]
};

// 3. Orchestrator can retrieve details if needed
// but doesn't have them in active context by default
```

### 3. Structured Handoffs

**Pattern**: Clear task specifications for workers

```typescript
// Orchestrator creates precise task for worker
const task = {
  "objective": "Review authentication code for security issues",
  "scope": {
    "files": ["src/auth/*.ts"],
    "focus": ["SQL injection", "XSS", "CSRF", "Auth bypass"]
  },
  "output_format": {
    "required": ["file", "line", "issue", "severity", "fix"],
    "max_items": 10,
    "priority": "critical_and_high_only"
  },
  "constraints": {
    "max_tokens": 2000,
    "time_budget": "5_minutes"
  }
};

// Worker receives clear boundaries
// Returns exactly what's specified
```

---

## Best Practices

### 1. Keep Orchestrator Pure

**Orchestrator = Coordinator, not Executor**

```typescript
// ✅ GOOD: Orchestrator delegates
async function orchestrate(task: Task) {
  // Analyze and plan
  const plan = await analyzTask(task);

  // Delegate to workers
  const results = await Promise.all([
    delegateToWorker('security-reviewer', plan.security_scope),
    delegateToWorker('quality-reviewer', plan.quality_scope),
    delegateToWorker('performance-reviewer', plan.performance_scope)
  ]);

  // Synthesize
  return synthesize(results);
}

// ❌ BAD: Orchestrator does detailed work
async function orchestrate(task: Task) {
  // This should be delegated!
  const securityIssues = await analyzeEveryFileForSecurity(allFiles);
  const qualityIssues = await analyzeEveryFileForQuality(allFiles);
  // ...
}
```

**Tool access**: Orchestrator should have minimal tools

```typescript
orchestrator: {
  tools: ['Task', 'Read', 'Grep'],  // Can delegate and gather info
  // NOT: Edit, Write, Bash (leave to workers)
}
```

### 2. Clear Task Decomposition

**Give workers precise objectives**

```
❌ BAD: Vague delegation
"Review the authentication system"

✅ GOOD: Precise delegation
"Review these 5 files for SQL injection vulnerabilities:
- src/auth/login.ts
- src/auth/register.ts
- src/database/queries.ts
- src/api/users.ts
- src/api/auth.ts

Focus only on SQL injection. Ignore other security issues.

Return format:
- File path and line number
- Vulnerability description
- Severity (Critical/High only)
- Remediation steps

Maximum 10 findings, prioritized by severity."
```

### 3. Parallel When Possible

**Identify independent subtasks**

```typescript
// ❌ SLOW: Sequential execution
const securityReview = await reviewSecurity(files);
const qualityReview = await reviewQuality(files);
const performanceReview = await reviewPerformance(files);
// Total time: sum of all three

// ✅ FAST: Parallel execution
const [securityReview, qualityReview, performanceReview] = await Promise.all([
  reviewSecurity(files),
  reviewQuality(files),
  reviewPerformance(files)
]);
// Total time: max of the three (up to 67% faster!)
```

### 4. Limit Coordination Overhead

**Minimize back-and-forth**

```
❌ BAD: Heavy coordination
Orchestrator → Worker A: "Do X"
Worker A → Orchestrator: "Done"
Orchestrator → Worker B: "Do Y with result from A"
Worker B → Orchestrator: "Need clarification"
Orchestrator → Worker B: "Here's clarification"
Worker B → Orchestrator: "Done"
... 10 round trips, lots of overhead

✅ GOOD: Clear upfront specification
Orchestrator → Worker A: "Do X, save result to file A"
Orchestrator → Worker B: "Do Y using input from file A"
Workers execute independently in parallel
Workers → Orchestrator: Results
... 1 round trip, minimal overhead
```

### 5. Error Handling & Resilience

**Plan for worker failures**

```typescript
async function orchestrateWithErrorHandling(task: Task) {
  try {
    const results = await Promise.allSettled([
      workerTask1(),
      workerTask2(),
      workerTask3()
    ]);

    // Handle partial success
    const successes = results.filter(r => r.status === 'fulfilled');
    const failures = results.filter(r => r.status === 'rejected');

    if (failures.length > 0) {
      // Retry failed tasks
      const retries = await Promise.all(
        failures.map(f => retryTask(f))
      );
    }

    return synthesize(successes, retries);
  } catch (error) {
    // Fallback strategy
    return fallbackApproach(task);
  }
}
```

---

## Real-World Patterns

### Pattern 1: Feature Development Pipeline

```
User Request
    ↓
Orchestrator (Opus)
    ↓
    ├─→ Planner Agent (Sonnet)
    │   ├─ Creates detailed spec
    │   ├─ Identifies edge cases
    │   └─ Returns plan
    ↓
Orchestrator reviews plan
    ↓
    ├─→ Implementer Agent (Sonnet)
    │   ├─ Writes code
    │   ├─ Follows spec
    │   └─ Returns implementation
    ↓
Orchestrator checks implementation
    ↓
    ├─→ Test Writer Agent (Sonnet)
    │   ├─ Creates unit tests
    │   ├─ Creates integration tests
    │   └─ Returns tests
    ↓
    ├─→ Reviewer Agent (Sonnet)
    │   ├─ Reviews code quality
    │   ├─ Reviews security
    │   └─ Returns feedback
    ↓
If feedback requires changes:
    ├─→ Implementer Agent (refinement)
Else:
    ├─→ Document Writer Agent (Sonnet)
    │   └─ Creates documentation
    ↓
Orchestrator creates PR
```

### Pattern 2: Comprehensive Code Review

```
PR Created
    ↓
Review Orchestrator (Opus)
    ↓
    ├─→ Security Reviewer (Sonnet) ─┐
    ├─→ Quality Reviewer (Sonnet)  ─┤ Parallel
    ├─→ Performance Reviewer (Sonnet)┤ Execution
    └─→ Test Coverage Reviewer ─────┘
           │
           ↓
     All return findings
           │
           ↓
  Orchestrator synthesizes
           │
           ↓
   Prioritized feedback
           │
           ↓
       PR comment
```

### Pattern 3: Multi-File Refactoring

```
Deprecate function used in 75 files
    ↓
Refactor Coordinator (Opus)
    ↓
Step 1: Analysis
    ├─→ Grep: Find all usages
    └─→ Identify dependencies
    ↓
Step 2: Create safe refactoring plan
    ├─ Group files by category
    ├─ Identify test files
    └─ Create rollback strategy
    ↓
Step 3: Parallel file refactoring
    ├─→ File Refactorer 1 (Sonnet) → file1.ts
    ├─→ File Refactorer 2 (Sonnet) → file2.ts
    ├─→ ...
    └─→ File Refactorer 75 (Sonnet) → file75.ts
    │
    └─ Each refactors single file in isolated context
    ↓
Step 4: Coordinator verifies consistency
    ├─→ Run tests
    ├─→ Check compilation
    └─→ Verify no breakage
    ↓
Complete
```

### Pattern 4: Research & Documentation

```
Research Task: "Document Claude subagents"
    ↓
Research Lead (Opus)
    ↓
Step 1: Break into research areas
    ├─ Architecture & design
    ├─ Implementation details
    ├─ Best practices
    ├─ Real-world examples
    └─ Performance considerations
    ↓
Step 2: Parallel research
    ├─→ Researcher 1 (Sonnet) → Architecture
    ├─→ Researcher 2 (Sonnet) → Implementation
    ├─→ Researcher 3 (Sonnet) → Best practices
    ├─→ Researcher 4 (Sonnet) → Examples
    └─→ Researcher 5 (Sonnet) → Performance
    │
    └─ Each explores independently, returns summary
    ↓
Step 3: Lead synthesizes findings
    ├─ Identify common themes
    ├─ Remove duplicates
    ├─ Organize logically
    └─ Create outline
    ↓
Step 4: Delegate documentation
    └─→ Document Writer (Sonnet)
        ├─ Receives synthesized findings
        ├─ Creates comprehensive docs
        └─ Returns formatted documentation
    ↓
Complete
```

---

## Performance Considerations

### Token Overhead

From research:
- **Multi-agent = 15× token usage** vs single agent
- **But 90.2% better results** on complex tasks

**When worth it**:
- ✅ Complex, high-value tasks
- ✅ Heavily parallelizable work
- ✅ When quality > cost

**When not worth it**:
- ❌ Simple tasks
- ❌ High-volume, low-value operations
- ❌ Cost-sensitive scenarios

### Time Savings

With parallel execution:
- **Up to 90% time reduction** for parallelizable tasks
- Anthropic's research system achieved this

**Example**:
```
10 sequential tasks @ 2 min each = 20 minutes
10 parallel subagents @ 2 min each = 2 minutes
```

---

## Key Takeaways

1. **Orchestrator-worker is the proven pattern**

2. **Orchestrator = Coordinator** (not executor)

3. **Clear task decomposition** is critical

4. **Parallel execution** saves massive time

5. **Lightweight summaries** prevent context bloat

6. **External memory** for detailed work

7. **15× token overhead** - use for high-value tasks

8. **90.2% better results** on complex tasks

9. **Minimize coordination** overhead

10. **Error handling** is essential

---

## Implementation Checklist

- [ ] Define orchestrator with coordination-only tools
- [ ] Create specialized worker agents
- [ ] Implement clear task decomposition
- [ ] Use lightweight summary returns
- [ ] Enable parallel execution where possible
- [ ] Add external memory for detailed work
- [ ] Implement error handling and retries
- [ ] Monitor token usage and performance
- [ ] Track success rates
- [ ] Iterate based on measurements

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Subagent fundamentals
- [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md) - Managing token overhead
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive guidelines
- [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md) - Code samples
