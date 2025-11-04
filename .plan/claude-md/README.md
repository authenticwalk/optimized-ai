# Claude.md Structure

## Philosophy

**Minimal core + Detailed backing files**

The root `claude.md` file is kept to an absolute minimum (< 100 lines, ~500 tokens). Each rule references a detailed backing file in `./rules-details/` that contains:
- Full research citations
- Comprehensive explanations
- Examples and anti-patterns
- Validation methods
- Cost-benefit analysis
- Integration with other rules

## Why This Approach?

From `.ai-knowledge/research/`:

1. **Token Efficiency** (69% savings)
   - Core CLAUDE.md: Always loaded (~500 tokens)
   - Detailed rules: Load only when needed via @imports
   - Research: 80-98% cost reduction achievable

2. **Progressive Disclosure**
   - Minimal info upfront
   - Deep context on-demand
   - Skills system: 30-50 tokens until loaded

3. **Validation Required**
   - Every rule backed by research
   - Citations to `.ai-knowledge/research/`
   - Empirical evidence, not assumptions

4. **Maintainability**
   - Easy to update: Change once in rules-details/
   - Single source of truth
   - Clear separation of concerns

## Structure

```
/claude.md                          # Minimal core (< 100 lines)
├─ Stack overview
├─ Core principles (summary)
├─ Critical rules (summary)
├─ Workflow (summary)
└─ References to detailed files

/.plan/claude-md/
├─ README.md                        # This file
└─ rules-details/                   # Detailed backing files
   ├─ token-efficiency.md           # Why minimize tokens
   ├─ progressive-disclosure.md     # Why use Skills
   ├─ structure-over-prose.md       # Why use tables/lists
   ├─ reference-not-duplicate.md    # Why link, not copy
   ├─ react-patterns.md             # Reason → Act → Observe
   └─ validation-required.md        # Experiment first

Each rule file contains:
- The Rule (1-2 sentences)
- Research Backing (with citations)
- Why This Rule Exists (problem solved)
- How to Apply This Rule (practical steps)
- What NOT to Do (anti-patterns)
- Validation Method (how to test)
- Integration with Other Rules (synergies)
- Examples from Research (real cases)
- Cost-Benefit Analysis (ROI)
- References (citations)
```

## Token Budget

### Current Allocation

```
claude.md:                     ~500 tokens
Rules metadata (in claude.md): ~100 tokens
Skills metadata (inactive):    ~300 tokens (10 skills × 30 tokens)
───────────────────────────────────────────
Baseline:                      ~900 tokens

When detailed rule needed:     +2,500 tokens (one rule file)
When skill loaded:             +1,500 tokens (one skill)
───────────────────────────────────────────
Typical task:                  ~2,900 tokens

vs Monolithic (5,000 tokens):  42% savings
```

### Validation

From `.ai-knowledge/research/claude/CLAUDE.md`:
- Official max: 5,000 tokens
- Recommended: 2,000 tokens
- Our target: 500 tokens (4x better than recommended)

**Status:** ✅ Under budget

## When to Load Detailed Rules

### Always in Memory
- `claude.md` core (500 tokens)

### Load When Needed
- Optimizing token usage? → Read `@.plan/claude-md/rules-details/token-efficiency.md`
- Creating skills? → Read `@.plan/claude-md/rules-details/progressive-disclosure.md`
- Formatting CLAUDE.md? → Read `@.plan/claude-md/rules-details/structure-over-prose.md`
- Testing optimization? → Read `@.plan/claude-md/rules-details/validation-required.md`
- Implementing workflow? → Read `@.plan/claude-md/rules-details/react-patterns.md`

**Trigger**: Only load when that specific rule is relevant to current task.

## Scientific Validation

Each rule has been validated through:

1. **Research Review**
   - 87 files in `.ai-knowledge/research/`
   - Multiple independent sources
   - Industry best practices

2. **Evidence Cited**
   - Token efficiency: 80-98% savings (industry reports)
   - Skills system: 69% savings (research-backed)
   - ReAct pattern: 40% error reduction (Anthropic guidance)

3. **Experimental Design**
   - Each rule includes validation method
   - A/B testing criteria
   - Success metrics defined

4. **Ready for Testing**
   - Hypothesis clearly stated
   - Experiment designed
   - Metrics identified
   - Decision criteria set

## Maintenance

### When to Update

**Monthly:**
- Review `claude.md` token count
- Check if any prose can be converted to structure
- Verify references are still accurate

**Quarterly:**
- Full audit of all rules
- Check for new research findings
- Update rules-details/ with new evidence
- Remove invalidated practices

**On New Research:**
- Add research findings to rules-details/
- Update citations
- Refine recommendations based on new evidence

### How to Add New Rules

1. **Create rule file:** `.plan/claude-md/rules-details/new-rule.md`
2. **Follow template:**
   - The Rule (1-2 sentences)
   - Research Backing (citations)
   - Why This Rule Exists (problem/evidence)
   - How to Apply (practical steps)
   - What NOT to Do (anti-patterns)
   - Validation Method (experiment)
   - Integration (synergies)
   - Examples (real cases)
   - Cost-Benefit (ROI)
   - References (sources)

3. **Add to claude.md:** One-line summary + reference
4. **Validate:** Run experiment to verify rule effectiveness
5. **Document:** Update this README

### How to Remove Rules

If rule is invalidated:
1. **Document why** in rules-details/[rule].md
2. **Mark deprecated** in claude.md
3. **Keep file** for historical reference
4. **Update references** in other rules

Never delete - maintain history for learning.

## Comparison to Alternatives

### Alternative 1: Monolithic CLAUDE.md
```
Single file: 5,000 tokens
Always loaded: 5,000 tokens
Detailed? YES
Efficient? NO

Our approach:
Core: 500 tokens (90% reduction)
Load details on-demand
```

### Alternative 2: Subdirectory CLAUDE.md
```
/claude.md
/lib/firebase/CLAUDE.md
/tests/CLAUDE.md

Problem: Subdirectory loading is broken (bug #2571, #3529, #4275)
Workaround: Use Skills instead
```

### Alternative 3: No Structure
```
Just prompt Claude each time

Problem: Inconsistent, no learned patterns
Solution: Structured CLAUDE.md + Skills + .ai-knowledge/
```

## Research Sources

All rules backed by research in `.ai-knowledge/research/`:

### Primary Sources
- `claude/CLAUDE.md` - Official Claude Code best practices
- `similar-projects/anthropic-best-practices.md` - Anthropic official guidance
- `synthlang-prompt-optimization/best-practices-token-efficiency.md` - Token optimization
- `pheromind/learning-token-budget-management.md` - Budget management
- `MASTER-SYNTHESIS.md` - Comprehensive review

### Key Findings
- **Token efficiency**: 80-98% reduction possible (industry reports)
- **Skills system**: 69% savings vs monolithic (research-backed)
- **CLAUDE.md limit**: < 5,000 tokens (official), < 2,000 recommended, < 500 our target
- **Progressive disclosure**: 30-50 tokens per skill until loaded
- **ReAct pattern**: 40% error reduction (Anthropic guidance)

## Status

### Implementation Status
- [✅] Minimal claude.md created (< 100 lines)
- [✅] 6 detailed rule files created
- [✅] All rules backed by research
- [✅] References properly linked
- [✅] Token budget under target
- [⚠️] Experimental validation pending

### Next Steps
1. **Create Skills** (`.claude/skills/`)
   - firebase-auth.skill
   - testing-patterns.skill
   - refactoring.skill
   - performance.skill

2. **Run Experiments**
   - Exp-001: Validate token savings
   - Exp-002: Validate quality maintained
   - Exp-003: Measure skill load frequency

3. **Monitor Metrics**
   - Track token usage
   - Measure task success rate
   - Document learnings in `.ai-knowledge/`

4. **Iterate**
   - Refine based on data
   - Update rules as needed
   - Continuous improvement

## Questions?

See:
- Project principles: `@.plan/initial-design/principles/`
- Research findings: `.ai-knowledge/research/MASTER-SYNTHESIS.md`
- Validation framework: `@.plan/initial-design/principles/3-validate.md`
- Token optimization: `.ai-knowledge/research/synthlang-prompt-optimization/`

---

**Version:** 1.0
**Created:** 2025-11-04
**Status:** ✅ Ready for validation
**Token Budget:** 500/5000 (10% of max, 25% of recommended)
