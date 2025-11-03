# Pattern: Scientific Validation

**Category**: Pattern
**Source**: Scientific method, A/B testing practices, DSPy research methodology
**Key Pattern**: Validate every claim through controlled experiments before adopting

---

## Overview

**Anti-pattern**: "This optimization looks good, ship it!"

**Pattern**: **Scientific method** - Hypothesis → Experiment → Measure → Analyze → Conclude → Replicate.

The key insight: **Intuition is wrong surprisingly often**. What seems like an obvious improvement can hurt quality. What seems minor can have major impact. **Data beats opinions**.

This aligns perfectly with our **VALIDATE principle**: Every claim must be backed by experiments.

---

## The Scientific Method for Prompts

### Core Process

```markdown
1. OBSERVE
   Current state: What's working? What's not?
   Problem: Identify specific issue to address

2. QUESTION
   Why is this happening?
   What could improve it?

3. HYPOTHESIZE
   Testable prediction: "If we change X, then Y will happen"
   Quantify: "Y will improve by Z%"

4. EXPERIMENT
   Control group: Current state
   Treatment group: Proposed change
   Measure: Predefined metrics

5. ANALYZE
   Compare control vs treatment
   Statistical significance testing
   Effect size calculation

6. CONCLUDE
   Accept hypothesis (deploy change)
   Reject hypothesis (keep current)
   Inconclusive (need more data)

7. REPLICATE
   Run experiment again to confirm
   Test on different scenarios
   Validate robustness

8. DOCUMENT
   Record methodology
   Share results
   Update knowledge base
```

### Example: Token Reduction Experiment

**1. OBSERVE**
```markdown
Current CLAUDE.md: 1,456 tokens
Observation: Stack section uses prose (120 tokens)
Issue: Verbose format wastes tokens
```

**2. QUESTION**
```markdown
Can structured format reduce tokens without hurting clarity?
```

**3. HYPOTHESIZE**
```markdown
Hypothesis: "Converting stack section from prose to table will reduce tokens by 30% while maintaining or improving adherence"

Specific predictions:
- Token reduction: 30% (120 → 84 tokens)
- Adherence rate: No decrease (≥ 90%)
- Quality: No decrease (≥ 4.0/5.0)
```

**4. EXPERIMENT**
```markdown
Design:
- Control: Current prose format
- Treatment: Table format
- Sample size: 20 tasks (10 each)
- Tasks: Must reference tech stack
- Randomization: Coin flip for each task
- Blinding: Evaluator doesn't know which version

Control version:
"Our tech stack includes TypeScript with strict mode enabled for type safety. We use Firebase for authentication and Firestore for our database. For frontend we use Svelte and HTMX. We deliberately avoid React and Tailwind."
Tokens: 120

Treatment version:
| Component | Technology | Notes |
|-----------|------------|-------|
| Language | TypeScript | Strict mode |
| Auth | Firebase | - |
| DB | Firestore | - |
| Frontend | Svelte + HTMX | - |
| ❌ | React, Tailwind | Never use |
Tokens: 85

Metrics to measure:
- Token count (direct measurement)
- Adherence rate (does Claude follow stack constraints?)
- Task success rate (does task complete correctly?)
- Quality score (1-5 rating of output)
- Time to completion
```

**5. ANALYZE**
```markdown
Results:
Control group (prose, n=10):
- Tokens: 120
- Adherence: 9/10 (90%)
- Success: 10/10 (100%)
- Quality: avg 4.2
- Time: avg 15 min

Treatment group (table, n=10):
- Tokens: 85
- Adherence: 10/10 (100%)
- Success: 10/10 (100%)
- Quality: avg 4.4
- Time: avg 14 min

Statistical analysis:
- Token reduction: 29% (close to predicted 30%) ✓
- Adherence: 90% → 100% (improvement!) ✓
- Quality: 4.2 → 4.4 (p=0.03, significant) ✓
- Time: 15 → 14 min (not significant, p=0.21)

Effect sizes:
- Token reduction: Very large (29%)
- Adherence: Medium (Cohen's h = 0.46)
- Quality: Medium (Cohen's d = 0.52)
```

**6. CONCLUDE**
```markdown
Conclusion: ACCEPT hypothesis

Evidence:
✓ Token reduction achieved (29% vs predicted 30%)
✓ Adherence maintained (actually improved!)
✓ Quality maintained (actually improved!)
✓ No negative effects observed

Recommendation: Deploy table format to production CLAUDE.md

Confidence: High (p < 0.05, effect size medium-large)
```

**7. REPLICATE**
```markdown
Replication study (different tasks):
- Sample: 15 new tasks
- Results: Token reduction 31%, adherence 100%, quality 4.5
- Conclusion: Results replicate ✓

Generalization test (different sections):
- Apply table format to other sections
- Rules section: 35% reduction, adherence 95%
- Workflow section: 22% reduction, adherence 92%
- Conclusion: Pattern generalizes ✓
```

**8. DOCUMENT**
```markdown
experiments/EXPERIMENTS-LOG.md:

## Experiment: Stack Section Table Format
**Date**: 2025-11-10
**Hypothesis**: Table format reduces tokens 30% without quality loss
**Result**: CONFIRMED (29% reduction, quality improved)
**Status**: DEPLOYED
**Files changed**: .claude/CLAUDE.md (stack section)
**Learnings**:
- Structured formats improve both efficiency AND adherence
- Claude parses tables very well
- Pattern applicable to other sections
**Next steps**: Apply to rules and workflow sections
```

---

## Experiment Design Patterns

### Pattern 1: A/B Test

**Use case**: Compare two versions head-to-head.

```typescript
interface ABTestDesign {
  name: string;
  hypothesis: string;
  control: Prompt;
  treatment: Prompt;
  sampleSize: number;
  randomization: 'simple' | 'stratified' | 'blocked';
  metrics: Metric[];
  successCriteria: {
    metric: string;
    threshold: number;
    direction: 'increase' | 'decrease';
  }[];
}

async function runABTest(design: ABTestDesign): Promise<ABTestResult> {
  const results = { control: [], treatment: [] };

  // Run experiment
  for (let i = 0; i < design.sampleSize; i++) {
    const version = Math.random() < 0.5 ? 'control' : 'treatment';
    const prompt = version === 'control' ? design.control : design.treatment;
    const testCase = selectTestCase(design.randomization);
    const result = await runTask(prompt, testCase);

    results[version].push(result);
  }

  // Analyze
  const analysis = statisticalAnalysis(results.control, results.treatment);

  // Check success criteria
  const success = design.successCriteria.every(criterion => {
    const delta = analysis[criterion.metric].difference;
    const significant = analysis[criterion.metric].pValue < 0.05;
    const rightDirection = criterion.direction === 'increase' ? delta > 0 : delta < 0;
    const meetsThreshold = Math.abs(delta) >= criterion.threshold;

    return significant && rightDirection && meetsThreshold;
  });

  return {
    ...analysis,
    success,
    recommendation: success ? 'Deploy treatment' : 'Keep control'
  };
}
```

### Pattern 2: Multi-Variant Test

**Use case**: Compare 3+ versions simultaneously.

```typescript
interface MultiVariantTestDesign {
  name: string;
  hypothesis: string;
  variants: Prompt[];
  sampleSize: number;
  metrics: Metric[];
}

async function runMultiVariantTest(
  design: MultiVariantTestDesign
): Promise<MultiVariantResult> {
  const results = design.variants.map(() => []);

  // Distribute samples across variants
  const samplesPerVariant = Math.floor(design.sampleSize / design.variants.length);

  for (let i = 0; i < design.sampleSize; i++) {
    const variantIndex = i % design.variants.length;
    const prompt = design.variants[variantIndex];
    const testCase = selectTestCase();
    const result = await runTask(prompt, testCase);

    results[variantIndex].push(result);
  }

  // ANOVA for multiple comparisons
  const anova = anovaAnalysis(results);

  // Post-hoc pairwise comparisons
  const pairwise = pairwiseComparisons(results);

  // Find best variant
  const scores = results.map(r => mean(r.map(x => x.score)));
  const bestIndex = scores.indexOf(Math.max(...scores));

  return {
    anova,
    pairwise,
    bestVariant: design.variants[bestIndex],
    bestIndex,
    scores,
    recommendation: `Deploy variant ${bestIndex + 1}`
  };
}
```

### Pattern 3: Longitudinal Study

**Use case**: Track changes over time.

```typescript
interface LongitudinalStudyDesign {
  name: string;
  baseline: Prompt;
  optimizations: Array<{
    date: Date;
    change: string;
    prompt: Prompt;
  }>;
  measurementFrequency: 'daily' | 'weekly' | 'monthly';
  duration: number; // weeks
  metrics: Metric[];
}

async function runLongitudinalStudy(
  design: LongitudinalStudyDesign
): Promise<LongitudinalResult> {
  const timeline = [];

  // Measure baseline
  const baseline = await measureMetrics(design.baseline, design.metrics);
  timeline.push({ date: new Date(), version: 'baseline', metrics: baseline });

  // Track each optimization
  for (const opt of design.optimizations) {
    const metrics = await measureMetrics(opt.prompt, design.metrics);
    timeline.push({ date: opt.date, version: opt.change, metrics });
  }

  // Trend analysis
  const trends = analyzeTrends(timeline);

  // Regression analysis (is there consistent improvement?)
  const regression = linearRegression(timeline.map((t, i) => ({
    x: i,
    y: t.metrics.overallScore
  })));

  return {
    timeline,
    trends,
    regression,
    improvement: calculateTotalImprovement(baseline, timeline[timeline.length - 1].metrics),
    recommendation: regression.slope > 0 ? 'Continue optimizing' : 'Investigate plateau'
  };
}
```

### Pattern 4: Factorial Design

**Use case**: Test multiple factors simultaneously.

```typescript
interface FactorialDesign {
  factors: {
    name: string;
    levels: any[];
  }[];
  sampleSize: number;
  metrics: Metric[];
}

// Example: Test format + detail level + examples
const design: FactorialDesign = {
  factors: [
    { name: 'format', levels: ['prose', 'list', 'table'] },
    { name: 'detailLevel', levels: ['high', 'low'] },
    { name: 'exampleCount', levels: [0, 2] }
  ],
  sampleSize: 120,  // 3 × 2 × 2 × 10 repetitions
  metrics: ['tokens', 'quality', 'adherence']
};

async function runFactorialExperiment(
  design: FactorialDesign
): Promise<FactorialResult> {
  const conditions = generateAllCombinations(design.factors);
  const results = [];

  const samplesPerCondition = Math.floor(design.sampleSize / conditions.length);

  for (const condition of conditions) {
    const prompt = generatePrompt(condition);
    const conditionResults = [];

    for (let rep = 0; rep < samplesPerCondition; rep++) {
      const result = await runTask(prompt, selectTestCase());
      conditionResults.push(result);
    }

    results.push({
      condition,
      metrics: aggregateMetrics(conditionResults)
    });
  }

  // Main effects analysis
  const mainEffects = analyzeMainEffects(design.factors, results);

  // Interaction effects
  const interactions = analyzeInteractions(design.factors, results);

  // Find optimal combination
  const optimal = results.reduce((best, current) =>
    current.metrics.overallScore > best.metrics.overallScore ? current : best
  );

  return {
    results,
    mainEffects,      // Which factors matter most?
    interactions,     // Do factors interact?
    optimal,          // Best combination found
    recommendations: generateRecommendations(mainEffects, interactions)
  };
}
```

---

## Statistical Validation

### Required Sample Sizes

**Power analysis**: Determine how many samples needed for reliable results.

```typescript
function calculateSampleSize(
  effectSize: number,        // Expected difference
  significance: number = 0.05, // Alpha (false positive rate)
  power: number = 0.80        // 1 - Beta (false negative rate)
): number {
  // Cohen's d to sample size
  // Simplified formula (assumes t-test)
  const z_alpha = 1.96;  // For α = 0.05
  const z_beta = 0.84;   // For β = 0.20 (power = 0.80)

  const n = 2 * Math.pow((z_alpha + z_beta) / effectSize, 2);

  return Math.ceil(n);
}

// Examples
console.log('Small effect (d=0.2):', calculateSampleSize(0.2));  // ~393 per group
console.log('Medium effect (d=0.5):', calculateSampleSize(0.5)); // ~64 per group
console.log('Large effect (d=0.8):', calculateSampleSize(0.8));  // ~26 per group
```

**Rule of thumb**:
- Small expected improvement (< 10%): Need 50-100 samples per group
- Medium expected improvement (10-30%): Need 20-50 samples per group
- Large expected improvement (> 30%): Need 10-20 samples per group

### Significance Testing

**t-test for comparing means**:

```typescript
function tTest(
  group1: number[],
  group2: number[]
): { t: number; pValue: number; significant: boolean } {
  const mean1 = mean(group1);
  const mean2 = mean(group2);
  const std1 = standardDeviation(group1);
  const std2 = standardDeviation(group2);
  const n1 = group1.length;
  const n2 = group2.length;

  // Pooled standard error
  const pooledSE = Math.sqrt((std1 ** 2 / n1) + (std2 ** 2 / n2));

  // T-statistic
  const t = (mean1 - mean2) / pooledSE;

  // Degrees of freedom
  const df = n1 + n2 - 2;

  // P-value (two-tailed)
  const pValue = 2 * (1 - tDistribution(Math.abs(t), df));

  return {
    t,
    pValue,
    significant: pValue < 0.05
  };
}
```

**Effect size (Cohen's d)**:

```typescript
function cohensD(group1: number[], group2: number[]): number {
  const mean1 = mean(group1);
  const mean2 = mean(group2);
  const std1 = standardDeviation(group1);
  const std2 = standardDeviation(group2);
  const n1 = group1.length;
  const n2 = group2.length;

  // Pooled standard deviation
  const pooledStd = Math.sqrt(
    ((n1 - 1) * std1 ** 2 + (n2 - 1) * std2 ** 2) / (n1 + n2 - 2)
  );

  // Cohen's d
  const d = (mean1 - mean2) / pooledStd;

  return d;
}

// Interpretation:
// |d| < 0.2: Negligible
// |d| 0.2-0.5: Small
// |d| 0.5-0.8: Medium
// |d| > 0.8: Large
```

### Confidence Intervals

```typescript
function confidenceInterval(
  data: number[],
  confidence: number = 0.95
): { lower: number; upper: number } {
  const m = mean(data);
  const std = standardDeviation(data);
  const n = data.length;
  const se = std / Math.sqrt(n);

  // Critical value (z-score for 95% confidence = 1.96)
  const alpha = 1 - confidence;
  const z = 1.96;  // For 95% confidence

  return {
    lower: m - z * se,
    upper: m + z * se
  };
}

// Example usage
const results = [4.1, 4.3, 4.2, 4.4, 4.0, 4.5, 4.3, 4.2, 4.1, 4.4];
const ci = confidenceInterval(results);
console.log(`Mean quality: ${mean(results).toFixed(2)}`);
console.log(`95% CI: [${ci.lower.toFixed(2)}, ${ci.upper.toFixed(2)}]`);
// Interpretation: "We are 95% confident the true mean quality is between X and Y"
```

---

## Experiment Templates

### Template 1: Token Reduction Experiment

```markdown
## Experiment: [Section Name] Token Reduction

**Date**: YYYY-MM-DD
**Hypothesis**: [Specific optimization] will reduce tokens by [X]% without degrading [quality metric]

**Design**:
- Control: Current version
- Treatment: [Describe optimization]
- Sample size: [N] tasks per group
- Metrics: tokens, quality, adherence, success rate
- Success criteria: ≥ [X]% token reduction, no quality loss

**Procedure**:
1. Select [N] tasks that use [section name]
2. Randomize to control or treatment
3. Run tasks with assigned version
4. Measure all metrics
5. Statistical analysis (t-test, Cohen's d)

**Results**:
Control (n=[N]):
- Tokens: [mean ± SD]
- Quality: [mean ± SD]
- Adherence: [X]%
- Success: [X]%

Treatment (n=[N]):
- Tokens: [mean ± SD]
- Quality: [mean ± SD]
- Adherence: [X]%
- Success: [X]%

**Analysis**:
- Token reduction: [X]% (p=[value])
- Quality change: [X]% (p=[value])
- Adherence change: [X]% (p=[value])
- Effect sizes: [Cohen's d values]

**Conclusion**: [ACCEPT/REJECT] hypothesis
- Recommendation: [Deploy/Reject/Modify] treatment
- Confidence: [High/Medium/Low]
- Learnings: [Key insights]
```

### Template 2: Quality Improvement Experiment

```markdown
## Experiment: [Feature] Quality Improvement

**Date**: YYYY-MM-DD
**Hypothesis**: Adding [feature] will improve [quality metric] by [X]%

**Design**:
- Control: Without [feature]
- Treatment: With [feature]
- Sample size: [N] tasks per group
- Metrics: quality score, task success, time to completion
- Success criteria: ≥ [X]% quality improvement

**Procedure**:
[Detailed steps]

**Results**:
[Data tables]

**Analysis**:
[Statistical tests]

**Conclusion**: [ACCEPT/REJECT] hypothesis
[Recommendations]
```

### Template 3: Adherence Improvement Experiment

```markdown
## Experiment: [Instruction] Adherence Improvement

**Date**: YYYY-MM-DD
**Hypothesis**: Rewording [instruction] as [new version] will increase adherence from [X]% to [Y]%

**Design**:
- Control: Current wording
- Treatment: New wording
- Sample size: [N] tasks per group
- Metrics: adherence rate, task success
- Success criteria: ≥ [X] percentage point increase

**Procedure**:
[Detailed steps]

**Results**:
[Data]

**Analysis**:
[Statistical tests]

**Conclusion**: [ACCEPT/REJECT] hypothesis
[Recommendations]
```

---

## Validation Checklist

### Before Experiment
- [ ] Clear hypothesis stated
- [ ] Success criteria defined
- [ ] Sample size calculated (power analysis)
- [ ] Metrics defined and measurable
- [ ] Control and treatment versions prepared
- [ ] Randomization strategy chosen
- [ ] Test cases selected

### During Experiment
- [ ] Randomization applied correctly
- [ ] All tasks executed as designed
- [ ] Metrics collected consistently
- [ ] No changes to design mid-experiment
- [ ] Documentation maintained

### After Experiment
- [ ] Data quality checked
- [ ] Statistical analysis performed
- [ ] Effect sizes calculated
- [ ] Confidence intervals computed
- [ ] Results replicated (if possible)
- [ ] Learnings documented

### Before Deployment
- [ ] Hypothesis accepted (p < 0.05)
- [ ] Effect size meaningful (Cohen's d > 0.3)
- [ ] No negative side effects
- [ ] Results replicate
- [ ] Team review completed
- [ ] Rollback plan prepared

---

## Common Pitfalls

### Pitfall 1: P-Hacking

**Problem**: Running many tests until finding p < 0.05 by chance.

**Solution**: Pre-register hypotheses. Use Bonferroni correction for multiple comparisons.

```typescript
function bonferroniCorrection(pValues: number[], alpha: number = 0.05): boolean[] {
  const adjustedAlpha = alpha / pValues.length;
  return pValues.map(p => p < adjustedAlpha);
}

// Example: Testing 5 hypotheses
const pValues = [0.03, 0.04, 0.06, 0.01, 0.09];
const significant = bonferroniCorrection(pValues);
// Adjusted alpha = 0.05 / 5 = 0.01
// Only p=0.01 passes corrected threshold
```

### Pitfall 2: Small Sample Size

**Problem**: Not enough data to detect real effects.

**Solution**: Power analysis before experiment. Always aim for > 80% power.

### Pitfall 3: Confounding Variables

**Problem**: Other factors change between control and treatment.

**Solution**: Randomization. Control for known confounds. Use blocked designs.

### Pitfall 4: Publication Bias

**Problem**: Only publishing "successful" experiments, hiding failures.

**Solution**: Document ALL experiments in EXPERIMENTS-LOG.md, regardless of outcome.

### Pitfall 5: Overfitting

**Problem**: Optimizing for test cases, not generalizing.

**Solution**: Hold-out test set. Cross-validation. Test on new tasks regularly.

---

## Key Takeaways

1. **Always experiment** - Don't deploy without data
2. **Pre-register hypotheses** - Prevents p-hacking
3. **Use adequate sample sizes** - Power analysis first
4. **Statistical significance** - p < 0.05 threshold
5. **Effect sizes matter** - Significant ≠ meaningful
6. **Replicate findings** - One experiment is not enough
7. **Document everything** - Successes and failures
8. **Share learnings** - Build collective knowledge

**For our project**:
- ✅ experiments/EXPERIMENTS-LOG.md for all experiments
- ✅ Pre-registered hypotheses
- ✅ Statistical validation required
- ✅ Replication before deployment
- ✅ Effect size calculations
- ✅ Document failures too

**Scientific rigor** is not optional. It's the foundation of continuous improvement.

---

**Document Version**: 1.0
**Last Updated**: 2025-11-03
**Status**: Ready for implementation
**Related Docs**: README.md, pattern-continuous-optimization.md
