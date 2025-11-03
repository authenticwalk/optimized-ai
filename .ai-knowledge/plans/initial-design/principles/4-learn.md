# Principle 4: LEARN

**Philosophy**: System continuously improves by learning from successes and failures. Every rule, skill, and pattern has its own learning history.

## Core Concept

Continuous self-optimization through:
- **Usage tracking** - What's actually being used?
- **Performance diagnostics** - Why did it work well or poorly?
- **Pattern recognition** - What works in which situations?
- **Rule-level learnings** - Each rule/skill has its own history
- **Automatic optimization** - Remove unused, promote successful

## Learning System Architecture

### Individual Learning Files

**Each rule/skill/pattern gets its own learning file:**

```
learnings/
├── rules/
│   ├── minimize-boilerplate.md
│   ├── test-edge-cases.md
│   └── use-typescript-strict.md
├── skills/
│   ├── firebase-auth.md
│   ├── testing.md
│   └── refactoring.md
└── patterns/
    ├── error-handling.md
    ├── state-management.md
    └── api-integration.md
```

**Learning File Structure:**
```markdown
# Rule: Test Edge Cases

## Usage Statistics
- Referenced: 45 times (last 100 tasks)
- High impact: 12 times (prevented bugs)
- Low impact: 30 times (routine)
- Ignored: 3 times (caused issues)

## Success Examples

### Example 1: Caught Race Condition
**Task**: Implement concurrent user updates
**Outcome**: AI tested race condition, found bug
**Impact**: HIGH - Would have been production bug
**Tokens saved vs fixing later**: ~5000
**File**: `.plan/initial-design/learnings/examples/rule-test-edge-cases-001.md`

### Example 2: Prevented Null Pointer
**Task**: Add user profile feature
**Outcome**: AI tested null case, added handling
**Impact**: MEDIUM
**File**: `.plan/initial-design/learnings/examples/rule-test-edge-cases-002.md`

## Failure Examples

### Failure 1: Over-Testing
**Task**: Fix typo in comment
**Outcome**: AI tried to write edge case tests for typo
**Impact**: NEGATIVE - Wasted 500 tokens
**Learning**: Rule too broad, needs context
**File**: `.plan/initial-design/learnings/examples/rule-test-edge-cases-fail-001.md`

## Optimizations Applied
- 2025-11-10: Added "when implementing logic" qualifier
- 2025-11-15: Refined to exclude trivial changes

## Related Rules
- use-typescript-strict (complements)
- minimize-boilerplate (sometimes conflicts)

## Current Status: KEEP (high impact)
```

## Diagnostic System

### What to Diagnose

When task performs well or poorly, identify **why**:

**Good Performance**:
- Which rules were followed?
- Which skills were loaded?
- Which patterns were used?
- What made it efficient?

**Poor Performance**:
- Which rules were ignored?
- Which rules conflicted?
- Were wrong skills loaded?
- What caused spinning/errors?

### Performance Analysis Structure

```json
{
  "taskId": "task-2025-11-03-001",
  "description": "Implement Firebase auth",
  "outcome": "SUCCESS",
  "performanceMetrics": {
    "tokens": 25000,
    "duration": 180,
    "quality": 0.95,
    "firstAttemptSuccess": true
  },
  "rulesAnalysis": {
    "followed": [
      {
        "rule": "test-edge-cases",
        "impact": "HIGH",
        "evidence": "Tested null user case, prevented bug"
      },
      {
        "rule": "use-typescript-strict",
        "impact": "MEDIUM",
        "evidence": "Caught type error at compile time"
      }
    ],
    "ignored": [],
    "conflicted": []
  },
  "skillsAnalysis": {
    "loaded": ["firebase-auth", "testing"],
    "effectiveness": {
      "firebase-auth": "HIGH - Provided exact patterns needed",
      "testing": "MEDIUM - Some patterns not applicable"
    },
    "shouldHaveLoaded": [],
    "shouldNotHaveLoaded": []
  },
  "learnings": [
    "firebase-auth + testing is effective combination",
    "test-edge-cases rule prevented bug - HIGH VALUE",
    "testing skill could be more specific to auth"
  ]
}
```

### Automatic Diagnostics

After each task:
```typescript
async function analyzeTas

k(task: CompletedTask) {
  // Rule usage analysis
  const rulesUsed = identifyRulesReferenced(task.conversation);
  for (const rule of rulesUsed) {
    await updateRuleLearnings(rule, task);
  }
  
  // Skill effectiveness analysis
  for (const skill of task.skillsLoaded) {
    const effectiveness = measureSkillEffectiveness(skill, task);
    await updateSkillLearnings(skill, effectiveness, task);
  }
  
  // Pattern detection
  const patterns = detectPatterns(task.code);
  await recordSuccessfulPatterns(patterns, task.metrics);
  
  // Conflict detection
  const conflicts = detectRuleConflicts(task.conversation);
  if (conflicts.length > 0) {
    await flagForReview(conflicts);
  }
  
  // Store diagnostic
  await storeDiagnostic({
    taskId: task.id,
    rulesUsed,
    skillsLoaded: task.skillsLoaded,
    metrics: task.metrics,
    learnings: extractLearnings(task)
  });
}
```

## Learning Loops

### Loop 1: Usage-Based Optimization

**Track what's actually used:**

```json
{
  "rule": "minimize-boilerplate",
  "usage": {
    "totalTasks": 100,
    "referenced": 65,
    "highImpact": 20,
    "mediumImpact": 35,
    "lowImpact": 10,
    "negative": 0
  },
  "decision": "KEEP - High usage, positive impact"
}
```

**Decision criteria:**
- Referenced <10%: **REMOVE** (unused)
- Referenced 10-30%: **REVIEW** (consider making skill)
- Referenced >30%: **KEEP** (core rule)
- High impact: **KEEP** (even if low frequency)
- Negative impact: **REMOVE** or **REFINE**

### Loop 2: Conflict Resolution

**Detect conflicting rules:**

```json
{
  "conflict": {
    "rule1": "minimize-boilerplate",
    "rule2": "comprehensive-error-handling",
    "frequency": 8,
    "evidence": [
      {
        "taskId": "task-001",
        "confusion": "AI unsure if error handling is boilerplate"
      }
    ]
  },
  "resolution": "Clarify: Error handling is NOT boilerplate",
  "outcome": "Conflicts reduced to 0"
}
```

### Loop 3: Skill Optimization

**Track skill effectiveness:**

```json
{
  "skill": "firebase-auth",
  "loaded": 45,
  "effectiveness": {
    "high": 40,
    "medium": 3,
    "low": 2
  },
  "avgTokens": 28000,
  "avgQuality": 0.92,
  "decision": "KEEP - Highly effective"
}
```

**Optimize based on usage:**
- High effectiveness: Promote patterns to core
- Low effectiveness: Refine or remove
- Commonly loaded together: Consider merging

### Loop 4: Pattern Promotion

**Successful patterns → Core:**

```json
{
  "pattern": "firebase-auth-error-mapping",
  "usage": 40,
  "successRate": 0.98,
  "avgTokenSaving": 500,
  "decision": "PROMOTE to core firebase-auth.skill"
}
```

## Edge Cases & Learnings

### Over-Optimization: Removed Critical Rule
**Problem**: Rule had low usage (8%) but high impact

**Analysis**:
- Rule: "Check for race conditions"
- Used rarely but caught critical bugs when used
- Automated system flagged for removal

**Human Review**: KEEP - High impact justifies low frequency

**Learning**: Don't auto-remove low-frequency, high-impact rules

**Storage**: `.plan/initial-design/learnings/learn-over-optimization.md`

### Mis-Attribution: Wrong Rule Got Credit
**Problem**: System credited wrong rule for success

**Analysis**:
- Success attributed to "use-async-await"
- Actually due to "proper-error-handling"
- Attribution algorithm was simplistic

**Solution**: Improved attribution with AST analysis

**Storage**: `.plan/initial-design/learnings/learn-attribution.md`

### Skill Merge Backfire
**Problem**: Merged two commonly-loaded skills

**Analysis**:
- firebase-auth + firebase-firestore often loaded together
- Merged into firebase.skill
- Result: Too broad (250 lines), effectiveness dropped

**Solution**: Keep separate, just note they're often co-loaded

**Storage**: `.plan/initial-design/learnings/learn-skill-merge-fail.md`

### Context Matters: Same Rule, Different Impact
**Problem**: Rule effective in some contexts, not others

**Analysis**:
- Rule: "Prefer functional style"
- Effective for data processing tasks
- Not effective for UI component tasks

**Solution**: Context-aware rules (load based on task type)

**Storage**: `.plan/initial-design/learnings/learn-context-matters.md`

## Diagnostic Examples

### Example: High Performance Task

```markdown
# Task: task-2025-11-03-042
## Description: Implement user signup with Firebase

## Performance: EXCELLENT
- Tokens: 22,000 (vs avg 35,000) - 37% better
- Duration: 150s (vs avg 220s) - 32% faster
- Quality: 0.98 (vs avg 0.88) - 11% higher
- First attempt success: YES

## Diagnosis

### Rules That Helped:
1. **firebase-auth pattern** (HIGH IMPACT)
   - Used exact pattern from firebase-auth.skill
   - Avoided common mistake (wrong auth configuration)
   - Saved ~3000 tokens vs figuring it out

2. **test-edge-cases** (HIGH IMPACT)
   - Tested duplicate email signup
   - Caught bug before manual testing
   - Saved debugging cycle

3. **typescript-strict** (MEDIUM IMPACT)
   - Caught type error in auth callback
   - Fixed at compile time vs runtime

### Skills That Helped:
1. **firebase-auth.skill** (HIGH IMPACT)
   - Provided exact initialization pattern
   - Error code mapping was perfect
   - All guidance relevant

2. **testing.skill** (MEDIUM IMPACT)
   - Auth testing patterns helpful
   - Some patterns not applicable (skipped)

### Why It Went Well:
- Right skills loaded (firebase-auth, testing)
- AI followed all relevant rules
- No conflicts or confusion
- Patterns from previous learning applied

### Learnings to Store:
- firebase-auth + testing is effective combination (record)
- firebase-auth.skill patterns are high-value (promote)
- test-edge-cases prevented bug (count as win)
```

### Example: Poor Performance Task

```markdown
# Task: task-2025-11-03-055
## Description: Add error handling to API calls

## Performance: POOR
- Tokens: 65,000 (vs avg 35,000) - 86% worse
- Duration: 480s (vs avg 220s) - 118% slower
- Quality: 0.65 (vs avg 0.88) - 26% lower
- Spin detected: YES (3 cycles)
- Manual intervention: YES

## Diagnosis

### What Went Wrong:

1. **Wrong skill loaded** (NEGATIVE IMPACT)
   - Loaded firebase-auth.skill (not relevant)
   - Added confusion with irrelevant patterns
   - Should have loaded: error-handling.skill

2. **Rule conflict** (NEGATIVE IMPACT)
   - "minimize-boilerplate" vs "comprehensive-error-handling"
   - AI confused: Is error handling boilerplate?
   - Spun trying to decide

3. **Missing pattern** (NEGATIVE IMPACT)
   - No stored pattern for "API error handling"
   - Had to figure it out from scratch
   - Tried several approaches before finding good one

### Rules Ignored:
1. **DRY principle**
   - Repeated error handling code 5 times
   - Should have extracted to utility

### Why It Went Poorly:
- Skill auto-detection failed (wrong keywords)
- Rule conflict caused confusion
- No learned patterns for this scenario
- Spinning detection didn't trigger fast enough

### Actions to Take:
1. **Fix skill detection**
   - "error handling" should trigger error-handling.skill
   - NOT firebase-auth.skill

2. **Resolve rule conflict**
   - Clarify: Error handling is NOT boilerplate
   - Update rule definitions

3. **Create pattern**
   - Store successful API error handling pattern
   - Add to error-handling.skill

4. **Improve spin detection**
   - Should have caught after 2nd cycle, not 3rd

### Learnings to Store:
- Skill detection needs keyword "error" → error-handling.skill
- Rule conflict identified - needs resolution
- API error handling pattern missing - create it
- Spin detection threshold too lenient - tighten
```

## Success Metrics

- ✅ Each rule has usage statistics
- ✅ Each skill has effectiveness data
- ✅ Successful patterns stored with examples
- ✅ Failures diagnosed and learnings extracted
- ✅ System improves measurably over time
- ✅ Automatic optimization based on data

## Learning Storage Structure

```
learnings/
├── rules/
│   ├── minimize-boilerplate.md (usage stats, examples)
│   ├── test-edge-cases.md
│   └── ...
├── skills/
│   ├── firebase-auth.md (effectiveness, examples)
│   ├── testing.md
│   └── ...
├── patterns/
│   ├── error-handling.md (when works, examples)
│   └── ...
├── examples/
│   ├── success/
│   │   ├── task-001-firebase-auth.md
│   │   └── ...
│   └── failure/
│       ├── task-055-error-handling.md
│       └── ...
└── diagnostics/
    └── task-YYYY-MM-DD-NNN.json
```

## Related Principles

- [Principle 1: MINIMIZE](1-minimize.md) - Learn what to remove
- [Principle 2: SEPARATE](2-separate.md) - Learn skill effectiveness
- [Principle 3: VALIDATE](3-validate.md) - Validate learnings work

## Detailed Documentation

- `.plan/initial-design/learnings/` - All learnings
- `.plan/initial-design/experiments/` - Validation data

## Future Work

### Predictive Learning
- Predict which skills needed for new tasks
- Suggest optimizations proactively
- Learn from similar projects

### Cross-Project Learning
- Learn patterns across projects
- Build library of proven solutions
- Share learnings (privacy-respecting)

### Meta-Learning
- Learn how to learn better
- Optimize the learning process itself
- Identify best learning strategies

## References

- [CORE-PRINCIPLES.md](../CORE-PRINCIPLES.md)
- [EXPERIMENTAL-VALIDATION.md](../EXPERIMENTAL-VALIDATION.md)

