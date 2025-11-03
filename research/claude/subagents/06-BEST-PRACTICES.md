# Best Practices - Comprehensive Guide

**Last Updated**: November 3, 2025
**Source**: Anthropic documentation, community implementations, research findings

---

## Quick Reference

| Topic | Key Principle | Reference |
|-------|--------------|-----------|
| **Subagents** | One clear job per agent | [Section 1](#subagent-design) |
| **Skills** | Progressive disclosure, lean SKILL.md | [Section 2](#skills-design) |
| **Tokens** | Output optimization first (4× cost) | [Section 3](#token-management) |
| **Orchestration** | Keep coordinator pure | [Section 4](#orchestration) |
| **Tools** | Minimize overlap, clear descriptions | [Section 5](#tool-design) |
| **Context** | Summaries, not dumps | [Section 6](#context-management) |
| **Security** | Least privilege, validate inputs | [Section 7](#security) |
| **Testing** | Start small (20 queries), iterate | [Section 8](#testing-validation) |

---

## 1. Subagent Design

### One Job Per Agent

```typescript
// ❌ BAD: Agent does too much
{
  name: 'code-helper',
  description: 'Helps with code - reviews, writes, tests, documents',
  // Unclear when to use, does everything poorly
}

// ✅ GOOD: Focused, single-purpose
{
  name: 'security-reviewer',
  description: 'Reviews code for security vulnerabilities',
  // Clear purpose, specialized expertise
}
```

### Clear Description

**Purpose**: Help Claude decide when to invoke

```typescript
// ❌ BAD: Vague
description: "Helps with authentication"

// ✅ GOOD: Specific, keyword-rich
description: "Expert security reviewer for authentication code, focusing on SQL injection, XSS, CSRF, auth bypasses, and session management vulnerabilities"
```

**Tips**:
- Include key terms users might mention
- Specify domain/technology
- List main capabilities
- Be action-oriented

### Minimal Tool Access

**Principle**: Least privilege

```typescript
// Read-only reviewer
tools: ['Read', 'Grep', 'Glob']

// Test executor
tools: ['Read', 'Bash', 'Grep']

// Code modifier
tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob']

// Full implementer
tools: ['Read', 'Edit', 'Write', 'Bash', 'Grep', 'Glob']
```

### Return Summaries

```typescript
// ❌ BAD: Return everything
return {
  all_files_read: [...],  // 10k tokens
  full_analysis: [...],   // 20k tokens
  every_detail: [...]     // 15k tokens
};

// ✅ GOOD: Return summary
return {
  summary: "Reviewed 47 files, found 3 critical issues",
  critical_issues: [
    { file: "auth.ts:142", issue: "SQL injection", fix: "..." }
  ]
};
```

---

## 2. Skills Design

### Keep SKILL.md Lean

**Target**: ~100 lines (~400 tokens)

```markdown
❌ BAD: Everything in one file (1,000+ lines)
---
name: firebase
---
# Firebase Skill
[500 lines of auth patterns]
[300 lines of Firestore patterns]
[200 lines of Functions patterns]

✅ GOOD: Split into resources (100 lines)
---
name: firebase
---
# Firebase Skill

## Authentication
See [patterns/auth.md](patterns/auth.md)

## Firestore
See [patterns/firestore.md](patterns/firestore.md)

## Functions
See [patterns/functions.md](patterns/functions.md)
```

### Progressive Disclosure

**Three tiers**:
1. **Metadata** (5 tokens): Name + description
2. **SKILL.md** (400 tokens): Overview + navigation
3. **Resources** (variable): Loaded as needed

### Split Mutually Exclusive Contexts

```
❌ BAD: Monolithic
skills/
└── backend/
    └── SKILL.md  # Auth + DB + API + Deploy (rarely all needed)

✅ GOOD: Separated
skills/
├── auth/SKILL.md
├── database/SKILL.md
├── api/SKILL.md
└── deployment/SKILL.md
```

### Use Code as Documentation

```python
# scripts/init-firebase.py
"""
Firebase initialization script.

Can be executed directly OR referenced as documentation.

Usage:
    python scripts/init-firebase.py

Configuration:
    Set FIREBASE_API_KEY in .env
"""

def init_firebase():
    # Implementation here
    pass
```

**Benefits**:
- Executable tool
- Clear documentation
- No context loading needed for execution

### Descriptive Names

```yaml
❌ BAD
name: auth
description: Authentication stuff

✅ GOOD
name: firebase-authentication
description: Firebase authentication patterns for email/password signup, login, logout, session management, and error handling with security best practices
```

---

## 3. Token Management

### Priority Order

1. **Output tokens** (4× cost) - Optimize first
2. **Prompt caching** (75% savings) - Design for caching
3. **Progressive loading** - Load only what's needed
4. **Tool filtering** - Reduce tool definitions

### Output Optimization

```typescript
// Every output token = 4× cost!

// ❌ BAD: Verbose output
return "I have analyzed the authentication file and found that there is a SQL injection vulnerability on line 142 where user input is directly concatenated into a SQL query without proper sanitization or parameterization.";
// 41 tokens × 4 = 164 token-cost-equivalent

// ✅ GOOD: Concise output
return "SQL injection: auth.ts:142 - Use parameterized queries";
// 11 tokens × 4 = 44 token-cost-equivalent (73% savings!)
```

### Prompt Caching Design

**Structure**:
```
[CACHEABLE - Static content at top]
- System prompt
- Project context
- Core rules
- Tool definitions (stable set)

[DYNAMIC - Variable content at bottom]
- Current conversation
- Active skills
- User messages
```

### Progressive Loading

```
Baseline: Metadata only (50 tokens)
Task match: Load skill (+ 400 tokens)
Specific need: Load resource (+ 200 tokens)

vs Monolithic: 3,000 tokens always

Savings: 97.8% → 78.3% depending on task
```

### Tool Filtering

```typescript
// ❌ BAD: All 50 tools always included
tools: ALL_TOOLS  // 2,500 tokens

// ✅ GOOD: Filter based on role
tools: ['Read', 'Grep', 'Glob']  // 150 tokens (94% savings!)
```

---

## 4. Orchestration

### Keep Orchestrator Pure

```typescript
// ✅ GOOD: Orchestrator coordinates
async function orchestrate(task) {
  const plan = analyzePlan(task);
  const results = await Promise.all([
    delegateToWorker('security', plan.security_scope),
    delegateToWorker('quality', plan.quality_scope)
  ]);
  return synthesize(results);
}

// ❌ BAD: Orchestrator executes
async function orchestrate(task) {
  // Shouldn't be doing detailed work!
  const findings = await analyzeEveryFile(allFiles);
  // ...
}
```

### Precise Task Decomposition

```
❌ BAD: Vague
"Review the authentication code"

✅ GOOD: Precise
"Review files: auth.ts, login.ts, register.ts
Focus: SQL injection only
Return: file:line, issue, severity, fix
Max 10 findings, critical/high only"
```

### Parallel Execution

```typescript
// Identify independent tasks

// ✅ GOOD: Parallel
const [security, quality, performance] = await Promise.all([
  reviewSecurity(files),
  reviewQuality(files),
  reviewPerformance(files)
]);
// Time: max of 3 (1× duration)

// ❌ SLOW: Sequential
const security = await reviewSecurity(files);
const quality = await reviewQuality(files);
const performance = await reviewPerformance(files);
// Time: sum of 3 (3× duration)
```

### Lightweight Communication

```typescript
// Workers save detailed work externally
await writeFile('.ai-working/findings.md', detailedFindings);

// Return reference, not content
return {
  summary: "Analysis complete",
  details: ".ai-working/findings.md",
  key_findings: [...]  // Top 3-5 only
};
```

---

## 5. Tool Design

### Self-Contained & Unambiguous

```typescript
// ❌ BAD: Unclear what it does
{
  name: 'process_data',
  description: 'Processes data'
}

// ✅ GOOD: Clear and specific
{
  name: 'query_firestore_collection',
  description: 'Queries Firestore collection with optional filters, returns array of documents. Supports where clauses, ordering, and limits.'
}
```

### Minimize Overlap

```typescript
// ❌ BAD: Overlapping tools
- getUserById(id)
- getUserByEmail(email)
- getUserByUsername(username)

// ✅ GOOD: One flexible tool
- getUser({ id?, email?, username? })
```

### Return Token-Efficient Data

```typescript
// ❌ BAD: Dump everything
async function getUsers() {
  return await db.users.findAll();
  // Returns 50k tokens!
}

// ✅ GOOD: Controlled output
async function getUsers({ limit = 10, fields = ['id', 'email'] }) {
  const users = await db.users.find().limit(limit);
  return users.map(u => pick(u, fields));
  // Returns ~1k tokens (98% savings!)
}
```

---

## 6. Context Management

### External Memory

```markdown
## .plan/current-task.md
- Objective: Implement Firebase auth
- Status: In progress
- Decisions: Using email/password (not OAuth)
- Next: Implement signup endpoint

Agent context: "See .plan/current-task.md for current state"
Cost: 20 tokens (reference) vs 500 tokens (inline)
```

### Context Compaction

```
After every 3-5 turns OR when approaching limit:

Summarize:
- Key decisions made
- Current state
- Critical information

Discard:
- Intermediate exploration
- Tool call details
- Redundant information

Result: 70% context reduction
```

### Just-in-Time Loading

```typescript
// ❌ BAD: Pre-load everything
loadAllPatterns();  // 5,000 tokens upfront

// ✅ GOOD: Load as needed
// Start with references only: 50 tokens
// Load specific pattern when needed: +200 tokens
```

---

## 7. Security

### Least Privilege

```typescript
// Grant minimum necessary tools

// Reviewer (read-only)
tools: ['Read', 'Grep', 'Glob']

// Tester (execute but don't modify)
tools: ['Read', 'Bash', 'Grep']

// Implementer (full access)
tools: ['Read', 'Edit', 'Write', 'Bash', 'Grep', 'Glob']
```

### Input Validation

```typescript
// Validate all inputs

if (!ALLOWED_COLLECTIONS.includes(collection)) {
  throw new Error('Invalid collection');
}

if (collection.includes('..') || collection.includes('/')) {
  throw new Error('Invalid collection name');
}

// Prevent injection
if (query.includes('DROP') || query.includes('DELETE')) {
  throw new Error('Dangerous query detected');
}
```

### Secrets Management

```typescript
// ❌ BAD: Hardcoded secrets
const apiKey = 'sk-abc123...';

// ✅ GOOD: Environment variables
const apiKey = process.env.API_KEY;

// ❌ BAD: Secrets in code or prompts
prompt: "Use API key sk-abc123 to connect";

// ✅ GOOD: Reference to secure storage
prompt: "Use API key from environment variable FIREBASE_API_KEY";
```

---

## 8. Testing & Validation

### Start Small

**Anthropic's approach**:
- Start with **20 test queries**
- Iterate rapidly
- Add more tests as you refine

**Don't**:
- Start with 1,000 test cases
- Try to be comprehensive initially
- Spend weeks planning tests

### Use Statistical Analysis

```
Experiment design:
- Control group: 10 runs
- Treatment group: 10 runs
- Calculate: Mean, std dev, confidence intervals
- T-test for significance
- Document: Results + decision
```

### Track Key Metrics

```
Per Task:
- Time to completion
- Token usage (input + output)
- Tool calls
- Success rate
- Quality score

Per Agent:
- Activation frequency
- Average tokens
- Success rate
- Failure modes

Per Skill:
- Load frequency
- Tokens when loaded
- Impact on success
```

### Iterate Based on Data

```
After 10 tasks:
- Which instructions were used?
- Which skills loaded most?
- What failed and why?
- What can be removed?
- What should be added?

Optimize and measure again.
```

---

## 9. Prompt Engineering

### Clear Role Definition

```
You are a [SPECIFIC ROLE] with [EXPERTISE].

Your responsibilities:
- [Primary responsibility]
- [Secondary responsibility]

You do NOT:
- [What agent shouldn't do]
```

### Step-by-Step Process

```
For each task:
1. [First step]
2. [Second step]
3. [Third step]

Return format:
[Specific structure]
```

### Quality Standards

```
Success criteria:
- [Measurable criterion 1]
- [Measurable criterion 2]

Quality bars:
- [Standard to meet]
- [Standard to meet]
```

---

## 10. Error Handling

### Graceful Degradation

```typescript
try {
  return await primaryApproach();
} catch (error) {
  console.log('Primary failed, trying fallback');
  return await fallbackApproach();
}
```

### Retry Logic

```typescript
async function withRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(Math.pow(2, i) * 1000);  // Exponential backoff
    }
  }
}
```

### Structured Errors

```typescript
// ❌ BAD: Generic error
throw new Error('Failed');

// ✅ GOOD: Actionable error
throw new Error(`
Failed to query Firestore collection: ${collection}

Possible causes:
- Collection doesn't exist
- Insufficient permissions
- Invalid query syntax

Suggestions:
- Verify collection name
- Check Firebase rules
- Review query structure
`);
```

---

## Common Pitfalls

### ❌ Agent Does Too Much
**Issue**: Agent has multiple responsibilities
**Fix**: One clear job per agent

### ❌ Vague Task Delegation
**Issue**: "Review the code" (too vague)
**Fix**: Precise scope, format, constraints

### ❌ Context Pollution
**Issue**: Main agent context grows unbounded
**Fix**: Use subagents, external memory, compaction

### ❌ Monolithic Prompts
**Issue**: 3,000 line .cursorrules always loaded
**Fix**: Minimal core + progressive skills

### ❌ Returning Full Data
**Issue**: Subagent returns 50k token exploration
**Fix**: Return 1-2k token summary

### ❌ Sequential Execution
**Issue**: Tasks run one after another
**Fix**: Identify parallelizable tasks, use Promise.all

### ❌ No Token Tracking
**Issue**: Don't know what's expensive
**Fix**: Measure tokens per task, optimize

### ❌ Skipping Validation
**Issue**: Assume optimizations work
**Fix**: A/B test with statistical analysis

---

## Checklist for New Implementations

### Subagents
- [ ] One clear job per agent
- [ ] Specific, keyword-rich description
- [ ] Minimal tool access (least privilege)
- [ ] Returns lightweight summaries

### Skills
- [ ] SKILL.md < 100 lines
- [ ] Split into resource files
- [ ] Progressive disclosure implemented
- [ ] Descriptive name and description

### Orchestration
- [ ] Orchestrator coordinates (doesn't execute)
- [ ] Precise task decomposition
- [ ] Parallel execution where possible
- [ ] Lightweight communication

### Tokens
- [ ] Output optimization (4× cost)
- [ ] Prompt designed for caching
- [ ] Progressive loading implemented
- [ ] Tools filtered by role

### Security
- [ ] Least privilege tool access
- [ ] Input validation
- [ ] No hardcoded secrets
- [ ] Secure external integrations

### Testing
- [ ] Start with small test set (20 queries)
- [ ] Track key metrics
- [ ] A/B testing for optimizations
- [ ] Iterate based on data

---

## Key Principles Summary

1. **MINIMIZE**: Every token must justify its existence
2. **SEPARATE**: Different contexts for different tasks
3. **VALIDATE**: Prove everything with data
4. **ITERATE**: Continuous improvement based on usage

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md)
- [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md)
- [03-MODEL-CONTEXT-PROTOCOL.md](03-MODEL-CONTEXT-PROTOCOL.md)
- [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md)
- [05-ORCHESTRATION-PATTERNS.md](05-ORCHESTRATION-PATTERNS.md)
