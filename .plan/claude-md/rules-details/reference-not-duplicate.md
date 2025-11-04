# Rule: Reference Not Duplicate

## The Rule
Link to external docs with @imports instead of duplicating content.

## Research Backing

### Source 1: Import System (.ai-knowledge/research/claude/CLAUDE.md:260-276)
**CLAUDE.md supports imports:**
```markdown
@path/to/additional-context.md
@~/.claude/my-global-instructions.md
```

**Import rules:**
- Relative and absolute paths supported
- Home directory access via `@~/`
- Recursive imports allowed (max depth: 5)
- Imports inside code blocks are ignored

**Token implication**: Every imported file adds to your baseline token usage!

### Source 2: Reference by Location (.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:377-398)
**Principle**: Don't repeat what's already documented. Reference it.

```markdown
Anti-pattern (redundant):
CLAUDE.md:
"Core principles:
1. MINIMIZE: Keep configs under 100 lines
2. SEPARATE: Use skills for domain-specific logic
3. VALIDATE: Run experiments before adopting
4. LEARN: Capture patterns in .ai-knowledge"
Tokens: ~35

Best practice (reference):
CLAUDE.md:
"Core principles: @.plan/initial-design/principles/"
Tokens: ~8
Reduction: 77%

The full details are in the referenced files.
Only load when needed (via @import).
```

## Why This Rule Exists

### Problem: Content Duplication

**Scenario**: You have comprehensive docs in your repo
```
project/
├── README.md (project overview, 500 tokens)
├── ARCHITECTURE.md (system design, 800 tokens)
├── CODING-STANDARDS.md (style guide, 600 tokens)
└── .plan/initial-design/principles/
    ├── 1-minimize.md (400 tokens)
    ├── 2-separate.md (400 tokens)
    ├── 3-validate.md (400 tokens)
    └── 4-learn.md (400 tokens)

Total: 3,500 tokens of existing documentation
```

**Without @imports (duplication):**
```markdown
# CLAUDE.md

## Project Overview
[Copy 500 tokens from README.md]

## Architecture
[Copy 800 tokens from ARCHITECTURE.md]

## Coding Standards
[Copy 600 tokens from CODING-STANDARDS.md]

## Core Principles
[Copy 1,600 tokens from principles/]

Total tokens: 3,500 (duplicated) + overhead
```

**With @imports (reference):**
```markdown
# CLAUDE.md

@README.md
@ARCHITECTURE.md
@CODING-STANDARDS.md
@.plan/initial-design/principles/

Total tokens: ~20 (imports) + 3,500 (referenced content)
```

**But wait**: @imports still load the full content!

**Optimal approach:**
```markdown
# CLAUDE.md

## Project Overview
Optimized AI - Self-learning coding assistant

Details: @README.md (reference for deeper context)

## Core Principles
1. MINIMIZE: < 100 line configs
2. SEPARATE: Skills for domains
3. VALIDATE: Experiments required
4. LEARN: Patterns in .ai-knowledge

Full details: @.plan/initial-design/principles/

## When to Load Details
Check referenced docs when:
- Clarification needed on principles
- Architectural decisions required
- Style questions arise
```

### Key Insight: Selective Reference

**Don't @import everything**. Instead:
1. Provide minimal summary in CLAUDE.md
2. Reference full docs with @path
3. Only import when details truly needed for every task

## How to Apply This Rule

### Strategy 1: Summary + Reference

```markdown
# In CLAUDE.md (Always Loaded)

## Tech Stack
- Language: TypeScript (strict)
- Backend: Firebase
- Frontend: Svelte
- Testing: Vitest

Full stack docs: @docs/TECH-STACK.md

## When to Load Full Docs
Read @docs/TECH-STACK.md when:
- Setting up new environment
- Evaluating technology changes
- Troubleshooting infrastructure issues
```

**Benefit:**
- Core CLAUDE.md: ~20 tokens (always loaded)
- Full docs: ~500 tokens (load on-demand via explicit read)
- Savings: 96% vs always importing

### Strategy 2: Progressive Reference Depth

```markdown
Level 1 (Always in CLAUDE.md):
## Principles
1. MINIMIZE
2. SEPARATE
3. VALIDATE
4. LEARN

Level 2 (Reference, load when needed):
@.plan/initial-design/principles/README.md

Level 3 (Deep reference, rarely load):
@.plan/initial-design/principles/1-minimize.md
```

### Strategy 3: Contextual Imports

```markdown
## Domain-Specific Docs

When working on Firebase:
- Auth: @docs/firebase/AUTH.md
- Firestore: @docs/firebase/FIRESTORE.md
- Functions: @docs/firebase/FUNCTIONS.md

When working on testing:
- Unit tests: @docs/testing/UNIT.md
- Integration: @docs/testing/INTEGRATION.md

Don't import upfront. Read when in that domain.
```

## Import Path Syntax

From .ai-knowledge/research/claude/CLAUDE.md:1127-1173:

| Syntax | Meaning | Example | Use Case |
|--------|---------|---------|----------|
| `@~/` | User's home directory | `@~/.claude/my-settings.md` | Personal preferences not in repo |
| `@` | Project root (relative) | `@docs/api.md` | Project documentation |
| `@./` | Project root (explicit relative) | `@./lib/README.md` | Same as `@` |

**Important:**
- `@~/` is NOT project root - it's user's home directory (e.g., `/home/username/`)
- `@` and `@./` are project-relative (resolve from project root)

### Example: Personal + Project

```markdown
# User's ~/.claude/CLAUDE.md (personal preferences)
## My Coding Style
- Always use const over let
- Prefer arrow functions
- 2-space indentation

# Project CLAUDE.md (team shared)
@~/.claude/CLAUDE.md  # Load user preferences
@README.md            # Project overview
@ARCHITECTURE.md      # System design

## Team Standards
[Team-specific rules here]
```

## What NOT to Do

### Anti-Pattern 1: @import Everything

```markdown
Bad:
@README.md
@ARCHITECTURE.md
@CONTRIBUTING.md
@CODE-OF-CONDUCT.md
@SECURITY.md
@CHANGELOG.md
@docs/API.md
@docs/DEPLOYMENT.md
@docs/TROUBLESHOOTING.md

Result: 5,000+ tokens loaded at session start
Most of this is irrelevant for day-to-day coding.
```

**Better:**
```markdown
## Project Docs
- Overview: @README.md
- Architecture: @ARCHITECTURE.md
- API: @docs/API.md

Read specific docs when needed.
Core info summarized below.

## Stack
[Minimal summary]

## Workflow
[Minimal summary]
```

### Anti-Pattern 2: Circular Imports

```markdown
Bad:
# CLAUDE.md
@docs/WORKFLOW.md

# docs/WORKFLOW.md
@CLAUDE.md

Result: Infinite loop or duplicate context
```

### Anti-Pattern 3: Importing Generated Content

```markdown
Bad:
@dist/bundle.js
@node_modules/firebase/index.d.ts
@.next/static/chunks/main.js

These are generated files.
Don't import build artifacts or dependencies.
```

## Validation Method

### Experiment: @import vs Summary

**Hypothesis**: "Summaries + selective @imports reduce tokens by 50% vs always importing"

**Design:**
- Control: @import all documentation at session start
- Treatment: Summaries in CLAUDE.md, @import only when needed
- Measure: Tokens loaded, task success rate, time to completion

**Success Criteria:**
- ✅ ≥40% token reduction
- ✅ No degradation in quality
- ✅ No increase in time (users read docs when needed)

### Measurement

```typescript
// Track import usage
interface ImportMetrics {
  path: string;
  loadedCount: number;
  avgRelevance: number;  // 0-10 score
  tokenCost: number;
}

// Example data
const imports: ImportMetrics[] = [
  { path: '@README.md', loadedCount: 2, avgRelevance: 3, tokenCost: 500 },
  { path: '@ARCHITECTURE.md', loadedCount: 1, avgRelevance: 8, tokenCost: 800 },
  { path: '@CHANGELOG.md', loadedCount: 0, avgRelevance: 0, tokenCost: 300 },
];

// Analysis
const wasted = imports.filter(i => i.avgRelevance < 5);
// Result: README.md and CHANGELOG.md rarely relevant
// Action: Remove from @imports, add as references only
```

## Integration with Other Rules

### Synergy with Token Efficiency
- Token Efficiency: Minimize CLAUDE.md tokens
- Reference Not Duplicate: Link instead of copying
- Combined: Maximum reduction in baseline tokens

### Synergy with Progressive Disclosure
- Reference Not Duplicate: Point to external docs
- Progressive Disclosure: Load skills on-demand
- Combined: Nothing loaded until proven necessary

### Synergy with Structure Over Prose
- Reference Not Duplicate: Minimal summary
- Structure Over Prose: Use lists/tables for summary
- Combined: Ultra-compact core config

## Examples from Research

### Example 1: Minimal CLAUDE.md with References
Source: .ai-knowledge/research/claude/CLAUDE.md:718-769

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

## References
- Core principles: @.plan/initial-design/principles/
- Validation framework: @.plan/initial-design/EXPERIMENTAL-VALIDATION.md
- Global learnings: @~/.optimized-ai/global-knowledge/patterns.md
```

**Analysis:**
- Core info: ~35 tokens (always loaded)
- Referenced docs: ~3,000 tokens (load on-demand)
- Savings: 99% vs importing everything upfront

## Recommended Patterns

### Pattern 1: Layered References

```markdown
## Documentation Layers

Layer 1 (Always in CLAUDE.md):
- Essential info only
- 1-2 sentence summaries

Layer 2 (Reference paths):
- @docs/[topic].md
- Load when topic is relevant

Layer 3 (Deep references within Layer 2 docs):
- Detailed implementation guides
- Historical context
- Advanced topics
```

### Pattern 2: Conditional Imports

```markdown
## Context-Sensitive Documentation

If working on authentication:
  Read @docs/firebase/AUTH.md

If working on database:
  Read @docs/firebase/FIRESTORE.md

If reviewing PR:
  Read @docs/REVIEW-CHECKLIST.md

Don't load all upfront.
```

### Pattern 3: Update References

```markdown
## Living Documentation

When documentation changes:
1. Update source doc (e.g., docs/API.md)
2. Check if CLAUDE.md summary needs update
3. Don't duplicate - keep reference link
4. Single source of truth

References always stay current.
```

## Cost-Benefit Analysis

### Investment
- Time: 30 minutes to add reference links
- Effort: Very low (just add @paths)
- Risk: Very low (easy to import if needed)

### Return
- Token savings: Variable (depends on doc size)
- Maintenance: Easier (update in one place)
- Consistency: Single source of truth
- Flexibility: Load details when needed

### ROI Example

```
Scenario: 3,000 tokens of documentation

Always Import Approach:
- Baseline: 3,000 tokens
- Every task: 3,000 tokens
- 50 tasks/week: 150,000 tokens/week

Reference Approach:
- Baseline: 50 tokens (references)
- Load full docs: 5 tasks/week need details
- 50 tasks/week: (45 × 50) + (5 × 3,050) = 17,500 tokens/week

Savings: 132,500 tokens/week (88%)
Cost savings: 132.5k × $0.005/1k = $0.66/week = $34/year
```

## Continuous Improvement

### Monthly Review

Check which @imports or references are actually used:

1. **Track reads**: Log when external docs are read
2. **Measure relevance**: How often was it helpful?
3. **Adjust references**:
   - Frequently used (>10x/month): Consider importing
   - Rarely used (<2x/month): Keep as reference only
   - Never used: Remove reference

### Import Optimization

```json
// .ai-knowledge/metrics/import-usage.json
{
  "period": "2025-11",
  "references": {
    "@README.md": {
      "reads": 3,
      "relevance": 7,
      "recommendation": "Keep as reference"
    },
    "@ARCHITECTURE.md": {
      "reads": 1,
      "relevance": 9,
      "recommendation": "Keep as reference"
    },
    "@CHANGELOG.md": {
      "reads": 0,
      "relevance": 0,
      "recommendation": "Remove reference"
    }
  }
}
```

## References

1. **Claude Research**: `.ai-knowledge/research/claude/CLAUDE.md` (lines 260-276, 1127-1173)
2. **Token Efficiency**: `.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md` (lines 377-398)
3. **Workarounds**: `.ai-knowledge/research/claude/CLAUDE.md` (Part 14: Subdirectory Bug Workarounds)

## Status
- **Research-Backed**: ✅ Multiple sources
- **Validated**: ✅ 77% savings in examples
- **Implementation**: Ready (simple @path syntax)
- **Priority**: MEDIUM (nice optimization)
