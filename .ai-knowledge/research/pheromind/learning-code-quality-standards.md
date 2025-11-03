# Learning: Code Quality Standards

## Discovery Context

**Source File:** `.tmp/pheromind/goodcodeguide.md`
**Context:** Comprehensive code quality guide with industry-backed metrics for function size, file size, and cyclomatic complexity.
**Relevance:** Supports our **Principle 3: VALIDATE** - provides measurable criteria for self-evaluation loop.

## The Core Insight

**Quantitative quality constraints prevent code bloat and maintain maintainability.**

**Key Standards:**
- **Functions:** 20-50 lines (industry consensus), Martin Fowler suggests <6 lines for simple cases
- **Files:** <500 lines with 2-10 cohesive functions
- **Cyclomatic Complexity:** ≤10 (maintainable), 11-20 (needs refactor plan), >20 (blocked), >50 (untestable)
- **Design Principles:** SOLID, DRY, KISS, YAGNI

## Deep Analysis

### Industry-Backed Standards

**Function Length Research:**
- **Code Complete (Steve McConnell):** Complex algorithms may extend to 100-200 lines, but should be exceptional
- **Martin Fowler:** Any function >6 lines "starts to smell" and should be considered for refactoring
- **Uncle Bob (Robert Martin):** Functions should be small, then smaller than that
- **Academic Guidelines:** Traditional recommendation is ≤30 lines per function
- **Industry Consensus:** 20-50 lines is the sweet spot

**File Size Analysis:**
- **Uncle Bob's Research:** Successful projects average 20-50 lines per file, most under 100 lines
- **Cognitive Load Research:** Files with >500 lines become difficult to comprehend
- **Industry Practice:** 2-10 functions per file maintains cohesion

**Cyclomatic Complexity Science:**
- **Formula:** CC = E - N + 2P (edges - nodes + 2 * connected components)
- **Research Findings:**
  - CC ≤10: Considered maintainable and testable
  - CC 11-20: Moderate risk, should have refactor plan
  - CC >20: High risk, difficult to test
  - CC >50: Untestable, must be refactored
- **Each decision point** (if, while, for, case) increases CC by 1

### Why These Numbers Matter

**Cognitive Load:**
- Humans can hold ~7 ± 2 items in working memory
- Short functions fit in mental model
- Small files reduce context switching

**Testability:**
- Lower complexity = fewer paths to test
- Smaller functions = easier to mock/stub
- CC ≤10 means ≤1024 test paths (manageable)
- CC >50 means >1,000,000,000,000,000 paths (impossible)

**Maintainability:**
- Smaller units = easier to understand
- Lower complexity = fewer bugs
- Clear boundaries = easier refactoring

## Implementation Details

### Quality Constraints from Pheromind

```markdown
**Size and Complexity Constraints:**

Functions:
- Target: 20-50 lines (industry consensus)
- Martin Fowler approach: Consider refactoring >6 lines
- Complex algorithms exception: May extend to 100-200 with justification
- Review trigger: >50 lines

Files:
- Target: <500 lines with 2-10 cohesive functions
- Average (successful projects): 20-50 lines
- Refactor proactively to minimize cognitive load

Cyclomatic Complexity:
- Maintain: ≤10 (normal code, considered maintainable)
- Document: 11-20 (requires refactor plan)
- Block: >20 (unless mitigation approved)
- Untestable: >50 (must be refactored)
```

### Design Principles (SOLID + Supporting)

From `goodcodeguide.md`:

**SOLID:**
- **S**ingle Responsibility: One reason to change
- **O**pen-Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes substitutable for base types
- **I**nterface Segregation: Many specific interfaces > one general
- **D**ependency Inversion: Depend on abstractions, not concretions

**Supporting Principles:**
- **DRY:** Don't Repeat Yourself - eliminate duplication through abstraction
- **KISS:** Keep It Simple, Stupid - favor simple over complex
- **YAGNI:** You Aren't Gonna Need It - don't build until needed

### Separation of Intention from Implementation

**Key Quote from Martin Fowler:**
> "If you need to spend effort figuring out what a code fragment does, extract it into a well-named function."

**Example:**

```typescript
// ❌ Poor: Implementation mixed with intention
function processUser(user: User) {
  if (user.age >= 18 && user.status === 'active' && user.emailVerified) {
    // 50 lines of processing logic...
  }
}

// ✅ Good: Intention separated from implementation
function processUser(user: User) {
  if (!canProcessUser(user)) return

  validateUserData(user)
  updateUserAccount(user)
  sendNotification(user)
}

function canProcessUser(user: User): boolean {
  return user.age >= 18 &&
         user.status === 'active' &&
         user.emailVerified
}
```

## Applicability to Our Project

### Current State (Optimized AI)

**What We Have:**
- General principles about code quality in SPEC.md
- User philosophy: "100% coverage is misleading" (Integration tests > unit tests)
- Preference for TypeScript with clean, simple configs

**What We're Missing:**
- Concrete, measurable quality constraints
- Cyclomatic complexity monitoring
- Function/file size enforcement
- Automated quality checks in self-evaluation

### Specific Recommendations

**Recommendation 1: Add Quality Constraints to `.cursorrules`**

```markdown
# Code Quality Constraints (Validated - Pheromind Research)

Functions:
- Target: 20-50 lines
- Review required: >50 lines
- Justification needed: >100 lines (complex algorithms only)
- Cyclomatic complexity: ≤10

Files:
- Maximum: 500 lines
- Target: 2-10 cohesive functions per file
- Cognitive load: Should be comprehensible in single read

Design Principles (Always apply):
- SOLID: Single responsibility, Open-closed, Liskov substitution, Interface segregation, Dependency inversion
- DRY: Eliminate duplication through abstraction
- KISS: Favor simple solutions
- YAGNI: Don't build until needed

Self-Evaluation Checks:
- Before committing, verify all functions ≤50 lines
- Check cyclomatic complexity ≤10
- Confirm no files >500 lines
- Validate SOLID principles applied
```

**Recommendation 2: Self-Evaluation Checklist Extension**

Add to the self-evaluation loop (ADR-007 in DECISIONS.md):

```markdown
## Code Quality Validation

- [ ] All functions ≤50 lines (or justified)
- [ ] All files ≤500 lines
- [ ] Cyclomatic complexity ≤10
- [ ] No code duplication (DRY)
- [ ] SOLID principles applied
- [ ] Meaningful naming (self-documenting)
- [ ] Proper error handling
- [ ] Security validated (input validation, output encoding)
```

**Recommendation 3: Quality Metrics Tracking**

Store in `.ai-knowledge/metrics.json`:

```json
{
  "codeQuality": {
    "functions": {
      "averageLines": 25.4,
      "maxLines": 48,
      "violations": [],
      "trend": "improving"
    },
    "files": {
      "averageLines": 180,
      "maxLines": 420,
      "violations": [],
      "trend": "stable"
    },
    "complexity": {
      "averageCC": 6.2,
      "maxCC": 12,
      "violations": ["src/auth.ts:login (CC=12)"],
      "trend": "stable"
    },
    "designPrinciples": {
      "solidCompliance": 0.95,
      "dryViolations": 2,
      "trend": "improving"
    }
  }
}
```

**Recommendation 4: IDE Integration Tool**

From our SPEC.md (ADR-010), we plan a VSCode extension. Add quality checks:

```typescript
// IDE Operations
ide.getComplexity(file)       // Calculate cyclomatic complexity
ide.getFunctionSizes(file)    // Get all function line counts
ide.getFileSizes()            // Get all file sizes
ide.checkQualityViolations()  // Run all quality checks
```

### Integration with Our Principles

**Principle 1: MINIMIZE**
- Small functions/files align with minimal context
- Lower complexity = less to understand
- Clear constraints prevent bloat

**Principle 3: VALIDATE**
- Measurable criteria for self-evaluation
- Objective quality gates
- Can be experimentally validated

**Principle 4: LEARN**
- Track quality metrics over time
- Learn which patterns lead to violations
- Automatically suggest refactoring when needed

## Experimental Validation

**Experiment: Code Quality Constraints Impact**

**Hypothesis:** Enforcing size and complexity constraints improves maintainability by 30% and reduces bug rates by 25%.

**Design:**
- **Control:** No explicit constraints enforced
- **Treatment:** Constraints enforced in self-evaluation
- **Scenarios:** 15 tasks across varying complexity
- **Runs:** 10 per scenario

**Metrics:**
- **Maintainability:** Code review scores, refactoring frequency, comprehension time
- **Bug Rate:** Bugs found in testing, production incidents
- **Quality Scores:** Function sizes, file sizes, cyclomatic complexity
- **Time Impact:** Development time with vs without constraints

**Success Criteria:**
- ≥20% improvement in maintainability scores
- ≥15% reduction in bug rates
- No >20% increase in development time
- Quality metrics within target ranges (functions ≤50, files ≤500, CC ≤10)

**Decision Criteria:**
- If validated: Adopt fully and enforce strictly
- If partial: Refine thresholds based on data
- If no improvement: Analyze why and reconsider approach

## Related Learnings

- [Testing Standards](./learning-testing-standards.md) - Testability depends on low complexity
- [Task Breakdown](./learning-task-breakdown.md) - Breaking tasks keeps code modular
- [Principle 3: VALIDATE](../.plan/initial-design/principles/3-validate.md) - Measurable validation criteria

## Conclusion

Pheromind's code quality standards provide **concrete, measurable criteria** backed by industry research. Adopting these constraints in our self-evaluation loop will:

1. Prevent code bloat and complexity creep
2. Improve testability and maintainability
3. Provide objective quality gates
4. Support our minimize and validate principles

**Key Standards to Adopt:**
- Functions: 20-50 lines (target), >50 requires review, >100 requires justification
- Files: <500 lines with 2-10 functions
- Cyclomatic Complexity: ≤10 (>20 blocked)
- SOLID + DRY + KISS + YAGNI

**Implementation Priority:** HIGH - Add to Phase 1 (Self-Evaluation Loop)

**Status:** ✅ Ready for experimental validation
