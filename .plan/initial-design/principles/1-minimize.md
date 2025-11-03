# Principle 1: MINIMIZE

**Philosophy**: Every token counts. Every instruction adds cognitive load and potential conflicts. Less is more.

## Core Concept

Maximum results with minimum overhead:
- **Small, tight prompts** - Every word justified by experiments
- **Load only what's needed** - Don't load instructions that won't be used
- **Compressed knowledge** - Store learnings efficiently
- **Avoid repetition** - Don't say the same thing multiple ways

## Target Metrics

- Core `.cursorrules`: **< 100 lines total**
- Skills: **~100 lines each** (loaded only when needed)
- Total context: **80-180 lines** vs 500+ monolithic
- **40%+ token reduction** validated experimentally

## Why This Matters

### Bad Approach (Bloated):
```
.cursorrules: 500 lines with every possible rule
- AI reads all instructions even though most don't apply
- Conflicting rules about file organization, testing, etc.
- 50k tokens used, 20 minutes, lots of confusion
- Token cost: $$$
```

### Good Approach (Minimized):
```
.cursorrules: 80 lines (core principles + project basics)
+ firebase-auth.skill: 100 lines (loaded only for auth tasks)
= 180 lines of relevant context
- 25k tokens used, 10 minutes, clear execution
- Token cost: $
```

**Savings**: 50% tokens, 50% time, clearer focus

## Instruction Optimization Strategy

### Start Minimal
1. Begin with ~30 lines of absolute core principles
2. Add project basics (tech stack, conventions) - ~30 lines
3. Add critical rules one at a time - ~40 lines
4. **Validate each addition experimentally**

### Test Every Line
For each instruction/rule:
- Does removing it degrade results? → If no, **remove it**
- Does it conflict with other instructions? → If yes, **resolve or remove**
- Does it improve metrics? → If no, **remove it**
- Is it used in this scenario? → If no, **don't load it** (make it a skill)

### Identify Conflicts
More instructions = higher chance of conflicts:
- Rule A: "Use functional style"
- Rule B: "Use classes for encapsulation"
- **Conflict!** AI gets confused, wastes tokens deciding

**Solution**: Pick one, remove the other, or clarify when to use each

## Edge Cases & Learnings

### When Minimal Goes Too Far
**Problem**: Removed too much, quality degraded

**Example**: 
- Removed "Test edge cases" rule
- AI stopped writing edge case tests
- Bug rate increased 30%

**Learning**: Some rules are low-frequency but high-impact. Keep them even if not often referenced.

**Storage**: `.plan/initial-design/learnings/minimize-too-minimal.md`

### Conflicting Rules Discovery
**Problem**: Two rules seemed fine individually but conflicted

**Example**:
- Rule: "Minimize boilerplate"
- Rule: "Add comprehensive error handling"
- AI confused: Is error handling boilerplate?

**Learning**: Need clear definition of what counts as "boilerplate" vs "necessary code"

**Storage**: `.plan/initial-design/learnings/minimize-conflicts.md`

### Skill Boundary Optimization
**Problem**: Unclear what belongs in core vs skills

**Experiment**: Moved "testing" from core to skill
- **Result**: Token reduction but missed test coverage on simple tasks
- **Learning**: Basic testing principle stays in core, advanced testing patterns go to skill

**Storage**: `.plan/initial-design/learnings/minimize-skill-boundaries.md`

## Validation Experiments

### Exp-001: Optimal .cursorrules Size
**Hypothesis**: "80-line .cursorrules provides same quality as 500-line with 40% fewer tokens"

**Design**:
- Control: 500-line .cursorrules
- Treatment 1: 30 lines (minimal core)
- Treatment 2: 50 lines (core + some basics)
- Treatment 3: 80 lines (core + project basics)
- Treatment 4: 100 lines (includes common patterns)

**Scenarios**: 10 common tasks

**Metrics**:
- Token usage
- Time to completion
- Quality (tests pass, linter clean, requirements met)
- Confusion rate (spinning, errors)

**Expected Results**:
- 30 lines: Too minimal, quality suffers
- 50 lines: Good for simple tasks, struggles with complex
- 80 lines: Sweet spot - good quality, big token savings
- 100 lines: Marginal improvement, not worth extra tokens

**Decision Criteria**:
- If 80 lines has ≥40% token reduction with no quality loss: **ADOPT**
- If quality suffers: Try 100 lines or identify missing critical rules
- If no significant improvement: Investigate why

**Status**: Pending Phase 0 completion

**Results**: TBD

### Exp-002: Line-by-Line Validation
**Hypothesis**: "Can identify and remove 20% of rules without impact"

**Design**:
- Start with 80-line validated config
- For each rule: Run 5 tasks with and without it
- Measure impact of each rule individually

**Metrics**:
- How often is rule referenced?
- Does removing it degrade quality?
- Does it conflict with other rules?
- Token cost per rule

**Expected Results**:
- 20% of rules: Never referenced → **REMOVE**
- 60% of rules: High value → **KEEP**
- 20% of rules: Low usage but high impact when used → **KEEP but consider making skill**

**Decision Criteria**:
- Remove rules with <5% reference rate and no quality impact
- Keep rules with >20% reference rate or significant quality impact
- Consider skill-ifying rules with 5-20% reference rate

**Status**: After Phase 1

**Results**: TBD

### Exp-003: Minimal vs Monolithic A/B
**Hypothesis**: "Minimal approach performs better across diverse scenarios"

**Design**:
- Control: Monolithic 500-line config
- Treatment: 80-line core + on-demand skills
- Scenarios: 20 diverse tasks (simple to complex)
- Runs: 5 per scenario per treatment

**Metrics**:
- Token usage (average and variance)
- Time to completion
- Quality scores
- User satisfaction (manual review)

**Expected Results**:
- Minimal: 40-60% fewer tokens, same or better quality
- Minimal: More consistent performance (less variance)
- Minimal: Clearer focus, less confusion

**Decision Criteria**:
- If minimal shows >30% improvement: **FULL ADOPTION**
- If mixed results: Identify scenarios where each works better
- If monolithic wins: **RE-EVALUATE APPROACH**

**Status**: After Phase 2

**Results**: TBD

## Implementation Notes

### .cursorrules Structure (Target: 80 lines)
```
# Core Principles (30 lines)
- Minimize boilerplate
- Separation of concerns
- Test edge cases
- Clean code practices
- [Critical principles only]

# Project Basics (30 lines)
- Tech stack (TypeScript, Firebase, Svelte)
- File structure conventions
- Naming conventions
- Key architectural patterns
- [Project-specific essentials]

# Critical Rules (20 lines)
- No React/Tailwind
- Use IDE for refactoring
- Self-evaluate before commit
- Load skills on-demand
- [Non-negotiable rules]
```

### Skill Structure (Target: ~100 lines each)
```
skills/
├── firebase-auth.skill (100 lines)
│   - Firebase auth setup
│   - Login/signup patterns
│   - Error handling
│   - Security best practices
├── testing.skill (100 lines)
│   - Test structure
│   - Edge case identification
│   - Mocking patterns
│   - Integration test setup
└── refactoring.skill (100 lines)
    - When to refactor
    - Extract function patterns
    - DRY identification
    - Performance optimization
```

## Success Metrics

- ✅ Core .cursorrules < 100 lines
- ✅ 40%+ token reduction vs monolithic
- ✅ No quality degradation
- ✅ Faster execution
- ✅ Less confusion/spinning
- ✅ All validated experimentally

## Related Principles

- [Principle 2: SEPARATE](2-separate.md) - Skills and subagents
- [Principle 3: VALIDATE](3-validate.md) - Experimental validation
- [Principle 4: LEARN](4-learn.md) - Continuous optimization

## Learnings Repository

Detailed learnings stored in:
- `.plan/initial-design/learnings/minimize-*.md` - Specific learnings
- `.plan/initial-design/experiments/exp-00*.json` - Experiment results

## Questions & Future Work

### Open Questions:
1. What's the absolute minimum viable .cursorrules?
2. Should any rules be time-of-day or project-phase dependent?
3. How to handle project-specific vs global rules?
4. Can we auto-generate minimal configs from usage patterns?

### Future Experiments:
1. **Adaptive sizing**: Config size adjusts based on task complexity
2. **Context compression**: Compress frequently-used patterns
3. **Rule prioritization**: Most important rules loaded first
4. **Dynamic skills**: Skills that adapt based on usage

## References

- Main principles: [CORE-PRINCIPLES.md](../CORE-PRINCIPLES.md)
- Experimental validation: [EXPERIMENTAL-VALIDATION.md](../EXPERIMENTAL-VALIDATION.md)
- Implementation roadmap: [REVISED-ROADMAP.md](../REVISED-ROADMAP.md)

