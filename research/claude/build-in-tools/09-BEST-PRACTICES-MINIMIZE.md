# Best Practices for Token Minimization

**Aligns with**: Principle 1 - MINIMIZE (Maximum results, minimum overhead)

## Overview

Every token costs money and adds latency. This document provides concrete strategies for minimizing token usage while maintaining quality.

## Core Strategies

### 1. Search Before Read

**Anti-pattern** (wasteful):
```typescript
// Read 50 files to find the right one
for (const file of allFiles) {
  Read(file)  // 50 × 800 tokens = 40,000 tokens
}
```

**Best practice** (efficient):
```typescript
// 1. Find files first
Glob({ pattern: "**/*.ts" })  // ~230 tokens

// 2. Search content
Grep({
  pattern: "getUserId",
  glob: "*.ts",
  output_mode: "files_with_matches"
})  // ~150 tokens

// 3. Read only matches
Read("auth.ts")        // 800 tokens
Read("user-service.ts") // 800 tokens

// Total: ~2,000 tokens (95% savings!)
```

### 2. Use Offset/Limit for Large Files

**Anti-pattern**:
```typescript
// Read entire 5000-line file
Read({ file_path: "large-file.ts" })
// Truncated at 2000 lines anyway
// ~8,000 tokens
```

**Best practice**:
```typescript
// Find section first with Grep
Grep({
  pattern: "function targetFunction",
  output_mode: "content",
  "-n": true  // Show line numbers
})
// Result: Line 1247

// Read just that section
Read({
  file_path: "large-file.ts",
  offset: 1240,
  limit: 50
})
// ~200 tokens (97% savings!)
```

### 3. Batch Operations with MultiEdit

**Anti-pattern**:
```typescript
// 5 separate Edit calls
Edit({ file_path: "file.ts", old_string: "foo1", new_string: "bar1" })
Edit({ file_path: "file.ts", old_string: "foo2", new_string: "bar2" })
Edit({ file_path: "file.ts", old_string: "foo3", new_string: "bar3" })
Edit({ file_path: "file.ts", old_string: "foo4", new_string: "bar4" })
Edit({ file_path: "file.ts", old_string: "foo5", new_string: "bar5" })
// 5 × 150 tokens = 750 tokens
```

**Best practice**:
```typescript
// 1 MultiEdit call
MultiEdit({
  file_path: "file.ts",
  edits: [
    { old_string: "foo1", new_string: "bar1" },
    { old_string: "foo2", new_string: "bar2" },
    { old_string: "foo3", new_string: "bar3" },
    { old_string: "foo4", new_string: "bar4" },
    { old_string: "foo5", new_string: "bar5" }
  ]
})
// ~400 tokens (47% savings!)
```

### 4. Prefer Edit Over Write

**Anti-pattern** (for small changes):
```typescript
Read("file.ts")  // 1000 tokens
// Change 1 line
Write("file.ts", newContent)  // 1000 tokens
// Total: 2000 tokens
```

**Best practice**:
```typescript
Read("file.ts")  // 1000 tokens
Edit({
  file_path: "file.ts",
  old_string: "const x = 1;",
  new_string: "const x = 2;"
})  // 150 tokens
// Total: 1150 tokens (43% savings!)
```

### 5. Parallel Operations

**Anti-pattern** (sequential):
```typescript
// Read files one by one
const file1 = await Read("file1.ts")
const file2 = await Read("file2.ts")
const file3 = await Read("file3.ts")
// Same token cost, but slower and sequential
```

**Best practice** (parallel):
```typescript
// Read files simultaneously
Promise.all([
  Read("file1.ts"),
  Read("file2.ts"),
  Read("file3.ts")
])
// Same token cost, but faster
// Reduces overall latency
```

### 6. Use Grep Output Modes Wisely

**Anti-pattern**:
```typescript
// Always use content mode
Grep({
  pattern: "TODO",
  output_mode: "content",
  "-C": 3  // Context lines
})
// 50 matches × 7 lines × 20 tokens = ~7,000 tokens
```

**Best practice**:
```typescript
// Use files_with_matches first
Grep({
  pattern: "TODO",
  output_mode: "files_with_matches"
})
// ~150 tokens

// Then read specific files if needed
Read("file-with-todos.ts")
// ~800 tokens

// Total: ~950 tokens (86% savings!)
```

### 7. Limit Grep Results

**Anti-pattern**:
```typescript
// Get all TODOs in codebase
Grep({
  pattern: "TODO",
  output_mode: "content"
})
// 500 matches = ~10,000 tokens
```

**Best practice**:
```typescript
// Limit to first 20
Grep({
  pattern: "TODO",
  output_mode: "content",
  head_limit: 20
})
// 20 matches = ~400 tokens (96% savings!)
```

### 8. Specific Glob Patterns

**Anti-pattern**:
```typescript
// Overly broad pattern
Glob({ pattern: "**/*" })
// Returns 10,000 files = ~20,000 tokens
```

**Best practice**:
```typescript
// Specific pattern
Glob({ pattern: "src/**/*.test.ts" })
// Returns 50 files = ~1,000 tokens (95% savings!)
```

### 9. Use Subagents for Context Isolation

**Anti-pattern** (main context bloat):
```typescript
// Read 20 files into main context
for (const file of files) {
  Read(file)
}
// Main context now 16,000 tokens
// All future operations more expensive
```

**Best practice** (subagent):
```typescript
// Delegate to subagent with fresh context
Task({
  subagent_type: "Explore",
  prompt: "Analyze all test files and summarize patterns. thoroughness: medium",
  description: "Analyze tests"
})
// Main context stays clean
// Subagent returns summary (~500 tokens)
```

### 10. WebFetch vs Manual Copy-Paste

**Anti-pattern**:
```typescript
// User: "Here's the documentation: [pastes 100KB of docs]"
// Claude processes 20,000 tokens at Sonnet rates
// Cost: ~$0.06
```

**Best practice**:
```typescript
WebFetch({
  url: "https://docs.example.com",
  prompt: "What are the authentication methods?"
})
// Haiku processes 20,000 tokens (~$0.005)
// Sonnet processes ~400 tokens (~$0.001)
// Total: ~$0.006 (90% savings!)
```

## Token Cost Reference

### Tool Costs (Approximate)

| Operation | Input Tokens | Output Tokens | Total |
|-----------|-------------|---------------|-------|
| Glob | 30 | 200 | 230 |
| Grep (files_with_matches) | 50 | 100 | 150 |
| Grep (content, 10 matches) | 60 | 400 | 460 |
| Read (100-line file) | 20 | 400 | 420 |
| Read (500-line file) | 20 | 2,000 | 2,020 |
| Edit (simple) | 100 | 50 | 150 |
| MultiEdit (5 edits) | 250 | 50 | 300 |
| Write (200-line file) | 1,000 | 30 | 1,030 |
| WebFetch | 100 | 300 | 400 |
| WebSearch | 40 | 200 | 240 |
| Task (subagent spawn) | 5,000 | varies | 5,000+ |

### Model Costs (2025)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| Claude Sonnet 4 | ~$3.00 | ~$15.00 |
| Claude Haiku | ~$0.30 | ~$1.50 |

### Cost Examples

**Scenario 1: Find and modify function**

**Inefficient approach**:
```typescript
1. Read 50 files (find the right one)     40,000 tokens
2. Write entire file with change           1,000 tokens
Total: 41,000 tokens × $0.003 = $0.123
```

**Efficient approach**:
```typescript
1. Grep to find file                         150 tokens
2. Read specific file                        800 tokens
3. Edit specific function                    150 tokens
Total: 1,100 tokens × $0.003 = $0.003
Savings: 97% ($0.12)
```

**Scenario 2: Research authentication patterns**

**Inefficient approach**:
```typescript
1. User pastes docs into chat            20,000 tokens
2. Claude processes at Sonnet rates
Cost: 20,000 × $0.003 = $0.060
```

**Efficient approach**:
```typescript
1. WebSearch for docs                       240 tokens
2. WebFetch top 3 (Haiku processing)      1,200 tokens
Cost: 1,440 × $0.003 = $0.004
Savings: 93% ($0.056)
```

**Scenario 3: Build feature with 5 components**

**Inefficient approach**:
```typescript
1. Main agent creates all components
   - Reads examples                       5,000 tokens
   - Creates components                  20,000 tokens
   - Tests                               10,000 tokens
Total: 35,000 tokens × $0.003 = $0.105
Time: 5 minutes (sequential)
```

**Efficient approach**:
```typescript
1. Spawn 5 subagents in parallel
   - Each creates one component           5,000 tokens each
   - Main agent coordinates               1,000 tokens
Total: 26,000 tokens × $0.003 = $0.078
Savings: 26% ($0.027)
Time: 1 minute (parallel)
```

## Skills for Token Efficiency

### Load Only What's Needed

**Monolithic .cursorrules** (500 lines):
```
Every task loads:
- Firebase patterns (100 lines)
- Supabase patterns (100 lines)
- Testing patterns (100 lines)
- Refactoring patterns (100 lines)
- Svelte patterns (100 lines)

Total: 500 lines × 4 tokens/line = 2,000 tokens per task
```

**Skills-based approach** (80 + 100 lines):
```
Core .cursorrules (80 lines)              = 320 tokens
+ firebase-auth.skill (loaded on demand)  = 400 tokens

Total: 720 tokens per task
Savings: 64%
```

**Over 100 tasks**: 128,000 tokens saved = ~$0.38

### Skill Naming for Efficiency

**Bad** (vague triggers):
```yaml
---
name: general
triggers:
  keywords: [code, function, component]
---
```
**Problem**: Matches everything, always loaded

**Good** (specific triggers):
```yaml
---
name: firebase-auth
triggers:
  keywords: [firebase, authentication, login, signup]
  files: [firebase.config.*, auth/*.ts]
---
```
**Benefit**: Only loads when relevant

## Configuration Optimization

### Minimal .cursorrules

**Goal**: < 100 lines total

**Structure**:
```
# Core Principles (30 lines)
- Minimize boilerplate
- Test edge cases
- Clean code
[Critical principles only]

# Project Basics (30 lines)
- TypeScript + Firebase + Svelte
- File structure
- Naming conventions
[Project-specific essentials]

# Critical Rules (20 lines)
- No React/Tailwind
- Use IDE for refactoring
- Load skills on-demand
[Non-negotiable rules]

# Skills Loading (20 lines)
- When to load which skill
- Skill combinations
[Meta-instructions]

Total: 100 lines = 400 tokens
```

**vs. Monolithic**: 500 lines = 2,000 tokens

**Savings**: 80% (1,600 tokens per task)

## Measurement & Optimization

### Track Token Usage

**Add to .cursorrules**:
```markdown
After completing each task:
1. Report token usage estimate
2. Suggest optimization opportunities
3. Log to .ai-knowledge/metrics.json
```

**Example output**:
```
Task completed successfully.

Token usage estimate:
- Glob: 230 tokens
- Grep: 150 tokens
- Read (3 files): 2,400 tokens
- Edit (2 changes): 300 tokens
- Total: ~3,080 tokens

Optimization opportunities:
- ✓ Used Grep before Read (saved ~37,000 tokens)
- ✓ Used Edit instead of Write (saved ~2,000 tokens)
- Could use MultiEdit next time (save ~150 tokens)
```

### Experimental Validation

**From Phase 0 Plan**:

Create experiments to validate optimizations:

```bash
experiment create \
  --id exp-token-optimization \
  --hypothesis "Grep+Read workflow uses 90% fewer tokens than Read-all" \
  --scenario find-and-modify

experiment run --runs 10
experiment analyze
```

**Measure**:
- Token usage (input + output)
- Time to completion
- Quality (tests pass, requirements met)

**Decide based on data**: Adopt, reject, or refine

## Summary: Token Minimization Checklist

### Search & Discovery
- ✅ Glob before Read
- ✅ Grep with files_with_matches mode first
- ✅ Use head_limit for common patterns
- ✅ Specific patterns, not broad searches

### File Operations
- ✅ Read with offset/limit for large files
- ✅ Edit for small changes, not Write
- ✅ MultiEdit for multiple changes
- ✅ Parallel reads when independent

### Context Management
- ✅ Subagents for distinct phases
- ✅ Skills loaded on-demand
- ✅ Keep main context clean
- ✅ Return summaries, not full transcripts

### Web Tools
- ✅ WebFetch instead of manual paste
- ✅ Specific prompts for WebFetch
- ✅ WebSearch before WebFetch
- ✅ Leverage 15-minute cache

### Configuration
- ✅ .cursorrules < 100 lines
- ✅ Skills ~100 lines each
- ✅ Remove unused rules
- ✅ Validate experimentally

### Measurement
- ✅ Track token usage per task
- ✅ Identify optimization opportunities
- ✅ Run A/B tests (Phase 0)
- ✅ Continuous improvement

---

**Target**: 40-60% token reduction vs. unoptimized approach

**Method**: Experimental validation (Phase 0)

**Next**: [Best Practices for Separation](./10-BEST-PRACTICES-SEPARATE.md)
