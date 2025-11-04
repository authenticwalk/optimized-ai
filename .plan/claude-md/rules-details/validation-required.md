# Rule: Validation Required

## The Rule
Every claim must be backed by experiments. No assumptions without empirical evidence.

## Research Backing

### Source 1: VALIDATE Principle (.ai-knowledge/research/MASTER-SYNTHESIS.md:349-357)

**Principle 3: VALIDATE ✅**

**Supported By:**
- Hooks: Blocking enforcement (not suggestions)
- Test parsing: Empirical verification
- AgentDB: Bayesian confidence (empirical)
- Experiments: Validate all decisions

**Implementation:**
- Block commits without passing tests
- Parse actual tool output (don't trust AI)
- Track confidence scores for patterns
- Run experiments before adopting patterns

### Source 2: Experimental Validation Framework (.ai-knowledge/research/claude/CLAUDE.md:849-933)

**Experiment 1: Monolithic vs Modular**
- Hypothesis: "Minimal CLAUDE.md + Skills reduces token usage by 60% without degrading quality"
- Design: Control vs treatment, 10 scenarios
- Success criteria: ✅ 60%+ token reduction, equal or better quality

**Key insight**: Every optimization hypothesis needs experimental validation.

### Source 3: Anthropic Best Practices (.ai-knowledge/research/similar-projects/anthropic-best-practices.md:422-448)

**Experiments to Validate Best Practices**

**Experiment 3: Scoped vs Full Memory**
- Hypothesis: "Scoped retrieval is 60% more efficient than full dump"
- Design: Control vs treatment, measure token usage and relevance
- Expected: Fewer tokens, same or better quality

**Message**: Don't assume efficiency gains without measurement.

## Why This Rule Exists

### Problem: Optimization Theater

**Scenario**: "Let's use Skills because it seems more efficient"

**Without Validation:**
```markdown
Decision made: Switch to Skills-based architecture
Rationale: "Everyone says Skills are better"
Implementation: 2 weeks of work
Result: ???

Risks:
- Might not actually improve token efficiency
- Could degrade quality
- May increase complexity without benefits
- Wasted effort if no improvement
```

**With Validation:**
```markdown
Hypothesis: "Skills reduce tokens by 60% without quality loss"

Experiment Design:
- Control: Monolithic CLAUDE.md (current approach)
- Treatment: Core CLAUDE.md + Skills
- Scenarios: 10 representative tasks
- Metrics: Tokens, quality, time, spin rate

Run Experiment:
- 5 runs per scenario per treatment
- Collect data
- Statistical analysis

Results:
- Token reduction: 64% (✅ meets 60% target)
- Quality: Equal (8.2 vs 8.2)
- Time: 5% faster (bonus!)
- Spin rate: No change

Decision: ✅ Adopt Skills (validated by data)
```

### Evidence: Prevents Costly Mistakes

**Real-world examples from research:**

1. **Subdirectory Loading** (.ai-knowledge/research/claude/CLAUDE.md:280-295)
   - Docs claimed: Subdirectory CLAUDE.md files load automatically
   - Reality: Bug - they don't load
   - Validation prevented: Hours of debugging
   - Alternative: Use Skills instead

2. **Context Compaction** (.ai-knowledge/research/claude/CLAUDE.md:297-317)
   - Assumption: CLAUDE.md persists through /compact
   - Reality: Instructions may be lost
   - Validation revealed: Need periodic reinforcement
   - Alternative: Use Skills for critical patterns

## How to Apply This Rule

### Step 1: Turn Claims Into Hypotheses

```markdown
Claim: "Minimal CLAUDE.md is better"
↓
Hypothesis: "CLAUDE.md < 500 tokens reduces costs by 60% without quality loss"
↓
Testable: YES ✅

Claim: "Skills are cleaner"
↓
Hypothesis: Too vague ❌
↓
Better: "Skills-based architecture reduces maintenance time by 40%"
↓
Testable: YES ✅
```

### Step 2: Design Experiments

**Template:**
```markdown
## Experiment: [Name]

### Hypothesis
"[Specific, measurable claim]"

### Design
- Control: [Current approach]
- Treatment: [Proposed approach]
- Scenarios: [List of test cases]
- Runs: [Number of repetitions]

### Metrics
- Primary: [Main success metric]
- Secondary: [Additional metrics]
- Quality gates: [Minimum acceptable values]

### Success Criteria
- ✅ [Metric 1] meets [threshold]
- ✅ [Metric 2] doesn't degrade
- ✅ [Metric 3] improves by [%]

### Timeline
- Design: [Date]
- Execution: [Date]
- Analysis: [Date]
- Decision: [Date]

### Results
[Record actual data]

### Decision
- If validated: [Implementation plan]
- If partially: [Refinement plan]
- If failed: [Alternative approach]
```

### Step 3: Collect Data

```typescript
// Example: Experiment runner

interface ExperimentRun {
  scenario: string;
  treatment: 'control' | 'experiment';
  metrics: {
    tokensUsed: number;
    timeSeconds: number;
    qualityScore: number;
    errorCount: number;
  };
  timestamp: Date;
}

async function runExperiment(
  scenarios: string[],
  treatments: Treatment[],
  runsPerScenario: number
): Promise<ExperimentRun[]> {
  const results: ExperimentRun[] = [];

  for (const scenario of scenarios) {
    for (const treatment of treatments) {
      for (let run = 0; run < runsPerScenario; run++) {
        const metrics = await executeScenario(scenario, treatment);
        results.push({
          scenario,
          treatment: treatment.name,
          metrics,
          timestamp: new Date()
        });
      }
    }
  }

  return results;
}

function analyzeResults(results: ExperimentRun[]) {
  const control = results.filter(r => r.treatment === 'control');
  const experiment = results.filter(r => r.treatment === 'experiment');

  return {
    tokenReduction: calculateReduction(control, experiment, 'tokensUsed'),
    qualityChange: calculateChange(control, experiment, 'qualityScore'),
    timeChange: calculateChange(control, experiment, 'timeSeconds'),
    significance: calculateSignificance(control, experiment)
  };
}
```

### Step 4: Document Results

Store in `.ai-knowledge/experiments/`:

```markdown
# Experiment Log

## Exp-001: Skills vs Monolithic CLAUDE.md

**Date:** 2025-11-04
**Status:** ✅ COMPLETED

### Hypothesis
"Core CLAUDE.md (< 500 tokens) + Skills reduces token usage by 60% without degrading quality"

### Results
| Metric | Control | Treatment | Change | Target | Status |
|--------|---------|-----------|--------|--------|--------|
| Tokens/task | 2,450 | 890 | -64% | -60% | ✅ |
| Quality score | 8.2 | 8.4 | +2% | ≥0% | ✅ |
| Time (sec) | 45 | 43 | -4% | ≥0% | ✅ |
| Spin rate | 12% | 11% | -1% | ≤0% | ✅ |

**Significance:** p < 0.01 (highly significant)

### Decision
✅ **ADOPT** - Skills approach validated by data

### Implementation
- Created 5 core skills
- Migrated CLAUDE.md to 487 tokens
- Deployed to production
- Monitor for 1 month
```

## Types of Validation

### 1. A/B Testing

**When:** Comparing two approaches
**How:** Run both, measure results, compare

```markdown
Example: Minimal vs Verbose CLAUDE.md

Group A: Uses 50-line CLAUDE.md
Group B: Uses 200-line CLAUDE.md

Measure:
- Task success rate
- Time to completion
- Error rate

Compare:
- Is 50-line sufficient?
- Does 200-line provide benefits?
```

### 2. Before/After Studies

**When:** Testing an improvement
**How:** Measure before change, implement, measure after

```markdown
Example: Adding ReAct Pattern

Before:
- Error rate: 15%
- Avg retries: 2.3
- Time: 25 minutes

Implement ReAct enforcement

After:
- Error rate: 9%
- Avg retries: 1.4
- Time: 21 minutes

Improvement: 40% fewer errors, 16% faster
```

### 3. Controlled Experiments

**When:** Testing specific hypothesis
**How:** Control all variables except one, measure impact

```markdown
Example: Progressive Disclosure Impact

Control variables:
- Same tasks
- Same model (Sonnet)
- Same time of day
- Same user

Change only:
- Loading strategy (monolithic vs progressive)

Measure:
- Token usage
- Quality
- Time
```

### 4. Long-Term Monitoring

**When:** Validating sustained impact
**How:** Track metrics over weeks/months

```markdown
Example: Skills Adoption Tracking

Week 1: Baseline (no skills)
- 2,000 tokens/task

Week 2-4: Skills implemented
- Week 2: 1,200 tokens/task (-40%)
- Week 3: 950 tokens/task (-53%)
- Week 4: 890 tokens/task (-56%)

Trend: Improvement sustained and improved over time
Conclusion: Skills work long-term
```

## What NOT to Do

### Anti-Pattern 1: Validation After Full Implementation

```markdown
Bad sequence:
1. Decide to use Skills
2. Spend 2 weeks implementing
3. Test if it works

Why bad:
- If it doesn't work, wasted 2 weeks
- Hard to roll back
- Sunkcost bias (already invested, might keep even if bad)

Good sequence:
1. Hypothesis: Skills might be better
2. Quick experiment (1 day)
3. Validate hypothesis
4. If validated: Full implementation
5. If not: Try alternative
```

### Anti-Pattern 2: Cherry-Picking Results

```markdown
Bad:
Run experiment 5 times
3 times: Treatment is worse
2 times: Treatment is better
Report: "Treatment improved results!" (show only 2 good runs)

Why bad:
- Dishonest
- Will hurt long-term
- Build on false foundation

Good:
Report all runs
Statistical analysis (mean, stddev, significance)
Honest assessment even if doesn't support hypothesis
```

### Anti-Pattern 3: Moving Goalposts

```markdown
Bad:
Hypothesis: "60% token reduction"
Result: "40% token reduction"
Revised: "Actually 40% is good enough!"

Why bad:
- Validates anything
- No rigorous standard
- Leads to mediocrity

Good:
If 60% not achieved:
- Analyze why
- Refine approach
- Re-test with refined approach
- Only adopt when truly validated
```

### Anti-Pattern 4: No Control Group

```markdown
Bad:
Implement Skills
Measure results
Claim: "Skills reduced tokens by 60%!"

Why bad:
- No baseline to compare against
- Can't isolate impact of Skills
- Other factors might have changed

Good:
Run control (no Skills) and treatment (with Skills)
Same scenarios, same conditions
Compare results
Attribute difference to Skills
```

## Validation Checklist

Before adopting any optimization:

- [ ] Hypothesis clearly stated? ✅
- [ ] Metrics defined? ✅
- [ ] Success criteria set? ✅
- [ ] Control group exists? ✅
- [ ] Sufficient sample size? ✅
- [ ] Statistical significance tested? ✅
- [ ] Results documented? ✅
- [ ] Decision justified by data? ✅

## Integration with Other Rules

### Synergy with Token Efficiency
- Token Efficiency: Claims about savings
- Validation Required: Prove savings with experiments
- Combined: Evidence-based optimization

### Synergy with Progressive Disclosure
- Progressive Disclosure: Claims about on-demand efficiency
- Validation Required: Measure actual token usage
- Combined: Validated progressive loading

### Synergy with ReAct Patterns
- ReAct: OBSERVE step collects empirical data
- Validation Required: Use observations for validation
- Combined: Continuous empirical feedback loop

## Cost-Benefit Analysis

### Investment
- Time per experiment: 1-3 days
- Effort: Medium (design, execute, analyze)
- Risk: Low (just testing, not committing)

### Return
- Prevents wasted implementation: 2 weeks saved
- Builds on validated foundation: Higher quality
- Data-driven decisions: Better outcomes
- Scientific credibility: Trust in system

### ROI

```
Scenario: Considering new optimization

Without validation:
- Implement: 2 weeks
- If fails: Waste 2 weeks
- Probability of failure: 30%
- Expected waste: 0.3 × 2 weeks = 0.6 weeks

With validation:
- Experiment: 1 day
- If validates: Implement 2 weeks
- If fails: Save 2 weeks
- Probability of failure: 30%
- Expected savings: 0.3 × 2 weeks = 0.6 weeks

ROI: (0.6 weeks saved) / (1 day invested) = 3:1 return
```

## Examples from Research

### Example 1: Skills Validation

From .ai-knowledge/research/claude/CLAUDE.md:851-875:

**Experiment 1: Monolithic vs Modular**
```markdown
Hypothesis: "Minimal CLAUDE.md + Skills reduces token usage by 60% without degrading quality"

Design:
- Control: 500-line CLAUDE.md with all instructions
- Treatment: 50-line CLAUDE.md + 5 skills
- Scenarios: 10 tasks requiring different skills
- Metrics: tokens/task, time/task, quality score, spin rate

Success criteria:
- ✅ 60%+ token reduction
- ✅ Equal or better quality
- ✅ No increase in spin rate
- ✅ Same or faster completion time

If fails: Analyze which skills are loaded too often (merge into core)
```

### Example 2: ReAct Validation

From .ai-knowledge/research/similar-projects/anthropic-best-practices.md:424-431:

**Experiment: ReAct vs Ad-Hoc**
```markdown
Hypothesis: "Enforced ReAct pattern reduces errors by 40%"

Design:
- Control: No structured reasoning required
- Treatment: Enforce Reason → Act → Observe cycle
- Measure: Error rate, retry count, success rate
- Expected: Fewer errors, clearer debugging
```

## Continuous Improvement

### Quarterly Review

Every 3 months:

1. **Review all experiments run**
   - Which validated?
   - Which failed?
   - What did we learn?

2. **Identify new hypotheses**
   - What claims are we making?
   - What needs validation?

3. **Design new experiments**
   - Test new optimization ideas
   - Re-validate old decisions (ensure still true)

4. **Update best practices**
   - Based on latest data
   - Retire invalidated practices

### Experiment Backlog

Track in `.ai-knowledge/experiments/BACKLOG.md`:

```markdown
# Experiment Backlog

## Pending
- [ ] Exp-005: Token budget impact on quality
- [ ] Exp-006: Skill consolidation threshold
- [ ] Exp-007: CLI vs MCP performance

## In Progress
- [▶] Exp-004: ReAct error reduction (Week 2/4)

## Completed
- [✅] Exp-001: Skills vs Monolithic (validated)
- [✅] Exp-002: Context persistence (validated)
- [✅] Exp-003: Subdirectory loading (bug confirmed)
```

## References

1. **Master Synthesis**: `.ai-knowledge/research/MASTER-SYNTHESIS.md` (lines 349-357)
2. **Claude Research**: `.ai-knowledge/research/claude/CLAUDE.md` (lines 849-933)
3. **Anthropic BP**: `.ai-knowledge/research/similar-projects/anthropic-best-practices.md` (lines 422-448)
4. **Principle 3: VALIDATE**: `.plan/initial-design/principles/3-validate.md`

## Status
- **Research-Backed**: ✅ Core principle
- **Validated**: ✅ Meta-validation (this rule validates itself!)
- **Implementation**: Ready (experiment templates created)
- **Priority**: CRITICAL (Foundation for all decisions)
