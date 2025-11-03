# Learning: Token Budget Management

## Discovery Context

**Source File:** `.tmp/pheromind/taskbp.md` (lines 1-408)
**Context:** Pheromind's comprehensive task planning template includes explicit token budget management for 128k context windows.
**Relevance:** Directly supports our **Principle 1: MINIMIZE** (`.plan/initial-design/principles/1-minimize.md`)

## The Core Insight

**Explicit token budgeting prevents context overflow and ensures AI agents stay within limits.**

Pheromind allocates 128,000 tokens with explicit percentages:
- **System Instructions:** 2,000 tokens (1.5%)
- **Task Definition:** 8,000 tokens (6.25%)
- **Critical Dependencies:** 12,000 tokens (9.4%)
- **Core Code Context:** 50,000 tokens (39%)
- **Documentation:** 15,000 tokens (11.7%)
- **Test Context:** 10,000 tokens (7.8%)
- **Output Buffer:** 20,000 tokens (15.6%)
- **Safety Buffer:** 11,000 tokens (8.6%)

**Key Principle:** Always reserve minimum output buffer (15,000 tokens) to ensure AI can respond.

## Deep Analysis

### Why This Approach Was Taken

**Problem Solved:**
- AI agents frequently exceed context windows when loading all potentially relevant information
- Lack of explicit budgeting leads to truncated context and degraded performance
- No clear strategy for what to include vs. omit when approaching limits

**Design Decision:**
- Percentage-based allocation provides flexibility across different task complexities
- Minimum thresholds ensure critical elements always included
- Buffer zones prevent hard limits from being hit

### How It Compares to Alternatives

**Alternative 1: No Explicit Budgeting (Current Common Practice)**
- ❌ Leads to unpredictable context overflow
- ❌ No systematic approach to prioritization
- ❌ Often results in truncated critical context
- ❌ Performance degrades silently

**Alternative 2: Hard Token Limits Per Section**
- ✅ Prevents any section from dominating
- ❌ Inflexible; simple tasks waste budget on unused allocations
- ❌ Complex tasks may need different distributions

**Pheromind's Approach: Dynamic Percentage-Based Allocation**
- ✅ Adapts to task complexity
- ✅ Clear prioritization framework
- ✅ Guaranteed minimum reserves
- ✅ Predictable, trackable, optimizable

### Edge Cases and Considerations

**Edge Case 1: Simple Tasks**
```
Simple Task (fixing typo):
- Reduce: Code context (don't need 50k tokens)
- Increase: Output buffer (may need more explanation)
- Result: Still fits well within 128k
```

**Edge Case 2: Complex System-Wide Changes**
```
Complex Task (refactor across 10 files):
- Increase: Code context (need more files)
- Reduce: Documentation (reference by ID)
- May still exceed: Use context compression
- Result: Requires careful prioritization
```

**Edge Case 3: Heavy Documentation Task**
```
Documentation Generation:
- Increase: Documentation allocation
- Reduce: Code context (just API surfaces)
- Increase: Output buffer (generating lots of text)
```

## Implementation Details

### Priority-Based Token Allocation

```markdown
CONTEXT_PRIORITY_MATRIX:

Priority 1 (Always Include - Critical):
- Task definition and acceptance criteria
- Direct code files being modified
- Immediate dependencies
- Critical error handling patterns

Priority 2 (Include if Space - Important):
- Related code files for reference
- Comprehensive documentation
- Extended dependencies
- Design patterns and examples

Priority 3 (Include Only if Extra Space - Nice-to-Have):
- Historical context
- Peripheral documentation
- Nice-to-have examples
- Tangential code files

Priority 4 (Omit if Necessary - Low Value):
- Deprecated patterns
- Unrelated code
- Verbose comments
- Redundant documentation
```

### Context Overflow Strategy

```markdown
If token limit approached:
1. First remove: [Priority 4 items] - Least critical context
2. Then compress: [Priority 3 items] - Summarize instead of full text
3. Then abstract: [Priority 2 items] - Reference by ID/path
4. Finally truncate: [Priority 1 items] - Partial inclusion with clear boundaries
5. Never compromise: Output buffer minimum (15,000 tokens)
```

### Dynamic Allocation Rules

From `taskbp.md`:
```
Dynamic Allocation Rules:
1. If task is simple: Reduce code context, increase output buffer
2. If task is complex: Increase code context, reduce documentation
3. If dependencies are heavy: Increase dependency allocation
4. Always preserve minimum output buffer of 15,000 tokens
```

## Applicability to Our Project

### How We Currently Handle This

**Current State (Optimized AI):**
- Core `.cursorrules`: Target <100 lines (~3-4k tokens)
- Skills: Target ~100 lines each (~3-4k tokens)
- Total context: 80-180 lines (~10-25k tokens)
- **No explicit token tracking or budgeting**

**Gap Identified:**
- We have line targets but no explicit token budgets
- No systematic approach to overflow handling
- No priority matrix for context inclusion
- Skills don't declare their token requirements

### Specific Recommendations

**Recommendation 1: Add Token Metadata to Skills**

```json
// skills/firebase-auth.skill.json
{
  "name": "firebase-auth",
  "description": "Firebase Authentication patterns",
  "estimatedTokens": 3500,
  "maxTokens": 5000,
  "priority": 1,
  "compressible": false,
  "content": "..."
}
```

**Benefit:** Skill loader can sum tokens before loading and make informed decisions.

**Recommendation 2: Implement Token Budget Tracking**

```typescript
// In skill loading system
interface TokenBudget {
  total: 128000,
  allocated: {
    core: 4000,
    skills: 0,  // Calculated dynamically
    taskContext: 8000,
    codeContext: 50000,
    documentation: 12000,
    outputBuffer: 20000,
    safetyBuffer: 10000
  },
  remaining: number,
  warnings: string[]
}

function calculateBudget(coreRules: string, skills: Skill[], task: Task): TokenBudget {
  // Estimate tokens for each component
  // Track remaining budget
  // Warn if approaching limits
  // Suggest compressions if needed
}
```

**Recommendation 3: Context Prioritization in Skill Loading**

```typescript
// Prioritize skills by relevance and token cost
interface SkillPriority {
  skill: Skill,
  relevanceScore: number,  // 0-10
  tokenCost: number,
  priorityLevel: 1 | 2 | 3 | 4
}

function selectSkills(
  availableSkills: Skill[],
  task: Task,
  remainingBudget: number
): Skill[] {
  // Score each skill by relevance
  // Sort by priority and token efficiency
  // Load until budget exhausted
  // Return optimal set
}
```

**Recommendation 4: Overflow Handling Strategy**

```markdown
When approaching token limit:
1. Check current allocation against budget
2. If >90% budget used:
   - Remove Priority 4 context (deprecated patterns)
   - Compress Priority 3 context (examples → summaries)
   - Reference Priority 2 by path (full docs → links)
3. If >95% budget used:
   - Truncate code context (show key sections only)
   - Skip non-critical documentation
4. Never compromise:
   - Core task definition
   - Immediate dependencies
   - Output buffer (min 15k tokens)
```

### Integration with Our Architecture

**Skill Definition Enhancement:**
```yaml
# firebase-auth.skill
name: firebase-auth
description: Firebase Authentication patterns
tokens:
  estimated: 3500
  maximum: 5000
  compressible: false
priority: 1
content: |
  # Firebase auth patterns...
```

**Skill Loader Logic:**
```typescript
async function loadSkills(task: Task): Promise<LoadedSkills> {
  const budget = new TokenBudget(128000)
  budget.allocate('core', estimateTokens(coreRules))
  budget.allocate('task', estimateTokens(task.definition))

  const candidateSkills = identifyRelevantSkills(task)
  const prioritizedSkills = prioritizeByRelevance(candidateSkills)

  const loadedSkills = []
  for (const skill of prioritizedSkills) {
    if (budget.canAllocate(skill.tokens.estimated)) {
      loadedSkills.push(skill)
      budget.allocate('skills', skill.tokens.estimated)
    } else {
      budget.warnings.push(`Skipped ${skill.name}: Insufficient budget`)
    }
  }

  if (budget.getRemaining() < 15000) {
    throw new Error('Insufficient output buffer')
  }

  return { skills: loadedSkills, budget }
}
```

## Experimental Validation Needed

**Experiment: Token Budget Impact**

**Hypothesis:** Explicit token budgeting reduces context overflow incidents by 60% and improves task completion quality by 20%.

**Design:**
- **Control:** Current approach (no explicit budgeting)
- **Treatment:** With explicit token budgeting and priority matrix
- **Scenarios:** 20 tasks of varying complexity
- **Runs:** 5 per scenario per treatment

**Metrics:**
- Context overflow incidents
- Token usage efficiency (% of budget used productively)
- Task completion quality score
- Time to completion
- Skill loading decisions (appropriate vs inappropriate)

**Success Criteria:**
- ≥50% reduction in overflow incidents
- ≥15% improvement in quality
- No significant increase in completion time
- More appropriate skill loading decisions

**Decision:**
- If validated: Adopt with full implementation
- If partial: Refine and re-test
- If failed: Analyze why and consider alternatives

## Related Learnings

**Links to Related Concepts:**
- [Context Prioritization Pattern](./pattern-context-prioritization.md) - How to prioritize what to load
- [Task Breakdown Learning](./learning-task-breakdown.md) - Keep tasks within token bounds
- [Principle 1: MINIMIZE](../.plan/initial-design/principles/1-minimize.md) - Our core principle this supports

## Conclusion

Token budget management is a **critical missing piece** in our current architecture. Pheromind's explicit percentage-based allocation with priority matrices provides a proven, systematic approach we should adopt.

**Key Adaptations:**
1. Add token metadata to all skills
2. Implement budget tracking in skill loader
3. Use priority matrix for context inclusion decisions
4. Never compromise output buffer minimum
5. Validate through controlled experiments

**Implementation Priority:** HIGH - Should be part of Phase 1 (Foundation)

**Status:** ✅ Ready for experimental validation
