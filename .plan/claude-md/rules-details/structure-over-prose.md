# Rule: Structure Over Prose

## The Rule
Use tables, lists, and structured formats instead of prose paragraphs.

## Research Backing

### Source: Token Efficiency Best Practices (.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:109-148)

**Format comparison** (same information):

```markdown
Prose (verbose):
"Our technology stack consists of several key components. For the programming language, we use TypeScript with strict mode enabled to ensure type safety. For authentication, we rely on Firebase Authentication, which provides secure user management out of the box. Our database is Firestore, a NoSQL cloud database that scales automatically. For the frontend, we use Svelte as our primary framework because it compiles to vanilla JavaScript and has excellent performance characteristics."

Tokens: ~85

Table (structured):
| Component | Technology | Reason |
|-----------|------------|--------|
| Language | TypeScript (strict) | Type safety |
| Auth | Firebase | Secure, managed |
| Database | Firestore | NoSQL, scalable |
| Frontend | Svelte | Compiled, fast |

Tokens: ~38
Reduction: 55%

List (compact):
Stack:
- Language: TypeScript (strict)
- Auth: Firebase
- DB: Firestore
- Frontend: Svelte

Tokens: ~20
Reduction: 76%
```

**Conclusion**: Lists saved 76% tokens vs prose for identical information.

## Why This Rule Exists

### Cognitive Load Reduction

**Prose requires parsing:**
```markdown
"When implementing new features, please make sure that you always write comprehensive unit tests that cover all edge cases, and also ensure that you update the documentation to reflect the changes you have made, and don't forget to run the test suite before committing."

Claude must:
1. Parse long sentence
2. Extract individual requirements
3. Hold in working memory
4. Identify priorities
```

**Structured format is immediate:**
```markdown
New features:
- Write unit tests (cover edge cases)
- Update documentation
- Run tests before commit

Claude can:
1. Scan list instantly
2. Process items independently
3. No parsing overhead
```

### Token Density

**Information per token:**
```
Prose: 3-4 pieces of info per 10 tokens (30-40% density)
Lists: 6-7 pieces of info per 10 tokens (60-70% density)
Tables: 7-8 pieces of info per 10 tokens (70-80% density)

Structured formats pack 2x more information per token.
```

## How to Apply This Rule

### When to Use Each Format

From research (.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:144-148):

- **Prose**: Nuanced explanations, complex relationships
- **Lists**: Simple enumerations, sequential steps
- **Tables**: Multi-attribute data, comparisons
- **XML**: Hierarchical data, clear sections
- **Math**: Logical rules, constraints

### Format: Bullet Lists

**Best for**: Sequential steps, simple enumerations

```markdown
Before (prose):
"The build process involves several steps. First, you need to run the linter to check code quality. Then compile TypeScript to JavaScript. After that, run the test suite to ensure nothing broke. Finally, create a production build with optimizations enabled."

After (list):
Build process:
1. Run linter
2. Compile TypeScript
3. Run tests
4. Create production build

Tokens: ~42 → ~12 (71% reduction)
```

### Format: Tables

**Best for**: Multi-attribute comparisons, structured data

```markdown
Before (prose):
"For development we use localhost on port 3000. The staging environment is at staging.example.com using HTTPS. Production runs at example.com also using HTTPS. Each environment has different API keys configured."

After (table):
| Env | URL | Protocol | API Key |
|-----|-----|----------|---------|
| Dev | localhost:3000 | HTTP | dev-key |
| Staging | staging.example.com | HTTPS | staging-key |
| Prod | example.com | HTTPS | prod-key |

Tokens: ~48 → ~24 (50% reduction)
```

### Format: Nested Lists

**Best for**: Hierarchical information

```markdown
Before (prose):
"The project has three main directories. The src directory contains all source code including components in src/components and utilities in src/utils. The tests directory mirrors the src structure with unit tests in tests/unit and integration tests in tests/integration. The docs directory has API documentation and user guides."

After (nested list):
Project structure:
- src/
  - components/
  - utils/
- tests/
  - unit/
  - integration/
- docs/
  - api/
  - guides/

Tokens: ~58 → ~18 (69% reduction)
```

### Format: XML Tags

**Best for**: Hierarchical sections, clear boundaries

```markdown
<stack>
  <language>TypeScript</language>
  <frontend>Svelte</frontend>
  <backend>Firebase</backend>
  <database>Firestore</database>
</stack>

<rules>
  <critical>Never merge to main</critical>
  <critical>Tests must pass before commit</critical>
  <preferred>Use IDE refactoring tools</preferred>
</rules>
```

### Format: Abbreviations + Tables

**Best for**: Frequently repeated terms

```markdown
Abbreviations:
- TS: TypeScript
- FB: Firebase
- FS: Firestore

| Component | Tech | Notes |
|-----------|------|-------|
| Language | TS (strict) | Type safety |
| Auth | FB Auth | Managed |
| DB | FS | NoSQL |

Tokens: ~15 (extremely compact)
```

## Prose vs Structure: Decision Tree

```
Is the information:

Multi-attribute data (X has property Y, Z)?
└─ YES → Use TABLE

Sequential steps?
└─ YES → Use NUMBERED LIST

Simple enumeration?
└─ YES → Use BULLET LIST

Hierarchical structure?
└─ YES → Use NESTED LIST or XML

Nuanced explanation with qualifiers?
└─ YES → Use PROSE (exception)

Logical rules / constraints?
└─ YES → Use MATH NOTATION

Key-value pairs?
└─ YES → Use DEFINITION LIST
```

## What NOT to Do

### Anti-Pattern 1: Lists for Complex Explanations

```markdown
Bad:
- Context compaction may lose instructions
- Progressive disclosure loads skills on-demand
- Token efficiency reduces costs
- Skills use 30-50 tokens until loaded

What's the relationship between these? Unclear.

Good (structured prose):
Context compaction may lose CLAUDE.md instructions. To mitigate:
- Use progressive disclosure (load skills on-demand)
- Skills cost 30-50 tokens until loaded vs full CLAUDE.md upfront
- Result: Better token efficiency and lower costs
```

### Anti-Pattern 2: Tables for Single-Attribute Data

```markdown
Bad:
| Stack Component |
|-----------------|
| TypeScript |
| Firebase |
| Svelte |

Just use a list:

Stack:
- TypeScript
- Firebase
- Svelte

Simpler and fewer tokens.
```

### Anti-Pattern 3: Over-Structured

```markdown
Bad:
<project>
  <stack>
    <language>
      <name>TypeScript</name>
      <version>5.0</version>
    </language>
  </stack>
</project>

Over-engineered. Just use:

Stack: TypeScript 5.0
```

## Validation Method

### A/B Test: Prose vs Structured

**Hypothesis**: "Structured formats reduce tokens by 50% without quality loss"

**Design:**
- Control: CLAUDE.md written in prose
- Treatment: Same information in structured formats
- Test: 10 representative tasks
- Measure: Tokens, comprehension (Claude's understanding), task success

**Success Criteria:**
- ✅ ≥40% token reduction
- ✅ No degradation in Claude's understanding
- ✅ Equal or better task completion quality

### Measurement

```typescript
// Count tokens for each format
const proseVersion = `
Our technology stack consists of TypeScript with strict mode for type safety...
`;

const structuredVersion = `
Stack:
- TypeScript (strict mode)
- Firebase Auth
- Firestore DB
`;

const proseTokens = countTokens(proseVersion);  // 85
const structuredTokens = countTokens(structuredVersion);  // 20
const savings = (proseTokens - structuredTokens) / proseTokens;  // 76%

assert(savings > 0.4);  // ✅ >40% reduction
```

## Examples: Before/After Transformations

### Example 1: Project Overview

**Before (prose):**
```markdown
This project is called Optimized AI and it's a self-learning coding assistant. We use TypeScript for type safety, Firebase for authentication and database, and Svelte for the frontend. The project follows four core principles: minimize, separate, validate, and learn. We store learned patterns in .ai-knowledge and work in .plan folders.
```
Tokens: ~62

**After (structured):**
```markdown
# Optimized AI - Self-Learning Coding Assistant

## Stack
- TypeScript (strict)
- Firebase (auth + DB)
- Svelte (frontend)

## Principles
1. MINIMIZE: < 100 line configs
2. SEPARATE: Skills for domains
3. VALIDATE: Experiments required
4. LEARN: Patterns in .ai-knowledge

## Folders
- .ai-knowledge/: Learned patterns
- .plan/: Working directory
```
Tokens: ~35 (44% reduction)

### Example 2: Workflow Instructions

**Before (prose):**
```markdown
When you start a task, first check the .ai-knowledge directory to see if there are similar patterns from past work. Then read the task from .plan/current-task.md. Create a plan in .plan/approach.md before implementing anything. After implementation, self-evaluate your work. If everything passes, create a pull request. Never merge to main automatically.
```
Tokens: ~58

**After (structured):**
```markdown
## Workflow
1. Check .ai-knowledge/ for patterns
2. Read .plan/current-task.md
3. Create .plan/approach.md
4. Implement
5. Self-evaluate
6. Create PR (never auto-merge to main)
```
Tokens: ~22 (62% reduction)

### Example 3: Tool Constraints

**Before (prose):**
```markdown
You should use IDE operations instead of manual refactoring because they're safer and faster. When working with Firebase, always use the official CLI rather than MCP servers unless you specifically need the memory MCP server. For testing, use Vitest and make sure all tests pass before committing.
```
Tokens: ~52

**After (structured):**
```markdown
## Tools
| Task | Use | Not |
|------|-----|-----|
| Refactor | IDE ops | Manual |
| Firebase | CLI | MCP* |
| Testing | Vitest | - |

*Exception: Memory MCP for knowledge graph

Critical: Tests must pass before commit
```
Tokens: ~26 (50% reduction)

## Integration with Other Rules

### Synergy with Token Efficiency
- Token Efficiency: Minimize every word
- Structure Over Prose: Use most efficient format
- Combined: Maximum token density

### Synergy with Reference Not Duplicate
- Structure Over Prose: Compact format in CLAUDE.md
- Reference Not Duplicate: Link to external docs for details
- Combined: Ultra-minimal core config

### Synergy with Progressive Disclosure
- Structure Over Prose: Compact skill metadata in CLAUDE.md
- Progressive Disclosure: Full details in skills
- Combined: Efficient skill discovery

## Cost-Benefit Analysis

### Investment
- Time: 1-2 hours to restructure existing CLAUDE.md
- Effort: Low (mostly formatting changes)
- Risk: Very low (easy to reverse)

### Return
- Token savings: 40-76% (research-backed)
- Improved scannability: Claude processes faster
- Maintainability: Easier to update structured data
- Consistency: Less ambiguity

### ROI
```
Time to restructure CLAUDE.md: 2 hours
Token savings: 50% average
Tasks per month: 200
Savings: 200 tasks × 1,000 tokens × 0.5 = 100,000 tokens saved/month

Cost savings: 100k × $0.005/1k = $0.50/month = $6/year

Qualitative benefits:
- Faster Claude processing ✅
- Easier to maintain ✅
- Less ambiguous ✅
- More scannable ✅
```

## Continuous Improvement

### Quarterly Audit

Every 3 months, review CLAUDE.md:

1. **Identify prose sections**
2. **Check if can be structured**
3. **Calculate potential token savings**
4. **Restructure if ≥20% savings available**

### Structure Checklist

Before committing CLAUDE.md changes:

- [ ] All multi-attribute data in tables? ✅
- [ ] All sequential steps in numbered lists? ✅
- [ ] All enumerations in bullet lists? ✅
- [ ] All hierarchies in nested lists or XML? ✅
- [ ] Prose only where truly necessary? ✅
- [ ] Measured token count? ✅
- [ ] Tested Claude's understanding? ✅

## References

1. **Token Efficiency BP**: `.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md` (lines 109-148)
2. **Claude Research**: `.ai-knowledge/research/claude/CLAUDE.md`
3. **Master Synthesis**: `.ai-knowledge/research/MASTER-SYNTHESIS.md` (Principle 1: MINIMIZE)

## Status
- **Research-Backed**: ✅ 76% savings proven
- **Validated**: ✅ Multiple sources
- **Implementation**: Ready (easy transformation)
- **Priority**: HIGH (quick wins available)
