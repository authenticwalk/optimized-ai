# Rule: Token Efficiency

## The Rule
Keep CLAUDE.md < 500 tokens. Every word must earn its place.

## Research Backing

### Source 1: Claude Research (.ai-knowledge/research/claude/CLAUDE.md:12)
> "CLAUDE.md should be < 5,000 tokens (official recommendation)"
> "Target 2,000 tokens for optimal balance"
> **Our target: < 500 tokens (10x more aggressive)**

### Source 2: Token Efficiency Best Practices (.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:26-40)
> "Hill Climb Up Quality First, Down Climb Cost Second"
> "Never sacrifice quality for tokens. Optimize quality first, THEN reduce tokens."
> **Industry reports: 80-98% cost reduction achievable**

### Source 3: Skills System Efficiency (.ai-knowledge/research/claude/CLAUDE.md:478-490)
```
Token efficiency comparison:

| Approach | Baseline Tokens | Per-Task Tokens | Total (10 tasks) |
|----------|----------------|-----------------|------------------|
| Monolithic CLAUDE.md (5,000 tokens) | 5,000 | 0 | 50,000 |
| Core CLAUDE.md (500 tokens) + Skills | 500 | +1,500/task | 15,500 |
| **Savings** | | | **69%** |
```

## Why This Rule Exists

### Problem Solved
1. **Context Window Waste**: Large CLAUDE.md consumes context that could be used for task-specific information
2. **Always-On Tax**: Every token in CLAUDE.md is loaded at session start, even if irrelevant
3. **Cost Impact**: More tokens = higher API costs
4. **Performance Degradation**: Larger context = slower processing

### Real-World Evidence
From research (.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:424-451):

**E-commerce platform:**
- Before: 3,200 tokens/task, $0.008/task
- After: 890 tokens/task, $0.002/task
- **Reduction: 72% tokens, 75% cost**

**SaaS application:**
- Before: 2,100 tokens/task, $0.005/task
- After: 450 tokens/task, $0.001/task
- **Reduction: 79% tokens, 80% cost**

## How to Apply This Rule

### Technique 1: Remove Filler Words
(.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:282-304)

```markdown
REMOVE these filler words:
- "please", "kindly"
- "make sure that", "ensure that"
- "it is important to"
- "you should", "you must"
- "in order to"
- "basically", "essentially"

Example:
Before: "Please make sure that you always try to write tests in order to ensure quality."
Tokens: ~17

After: "Write tests to ensure quality."
Tokens: ~6
Reduction: 65%
```

### Technique 2: Use Command Form
(.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:337-348)

```markdown
Before (polite form): "You should always run tests before committing your code."
Tokens: ~10

After (command form): "Run tests before committing."
Tokens: ~5
Reduction: 50%
```

### Technique 3: Structure Over Prose
(.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:109-139)

```markdown
Prose: "Our tech stack consists of TypeScript with strict mode for type safety..."
Tokens: ~85

List:
Stack:
- Language: TypeScript (strict)
- Auth: Firebase
- DB: Firestore
- Frontend: Svelte

Tokens: ~20
Reduction: 76%
```

### Technique 4: Challenge Every Word
(.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md:68-105)

**Audit questions for each sentence:**
1. What unique information does this provide?
2. Is this already stated elsewhere?
3. Would Claude understand without this?
4. Can this be said more concisely?
5. Is this obvious from context?

If "no unique value" → DELETE
If "redundant" → DELETE
If "obvious" → DELETE
If "can be shorter" → COMPRESS

## What NOT to Do

### Anti-Pattern 1: Over-Compression
```markdown
Bad: "Tests req'd b4 commit"
Good: "Run tests before committing"

Slightly more tokens, much clearer.
```
**Rule:** Stop when clarity starts to degrade

### Anti-Pattern 2: Premature Abbreviation
```markdown
Bad: "Impl auth w/ FB, use FS for usr dt, ens sec w/ RLS"
```
**Rule:** Only abbreviate frequently-used terms (5+ occurrences)

### Anti-Pattern 3: Sacrificing Quality
```markdown
Don't break functionality to save a few tokens.
Quality first, then optimize.
```

## Validation Method

### Measurement
1. Count tokens in CLAUDE.md: Use `/context` command or tokenizer
2. Track baseline: Document initial token count
3. Optimize: Apply techniques above
4. Re-measure: Count tokens again
5. Validate quality: Test with representative tasks

### Success Criteria
- [ ] CLAUDE.md < 500 tokens ✅
- [ ] No degradation in Claude's understanding ✅
- [ ] All critical information preserved ✅
- [ ] Skills loaded on-demand for details ✅

### A/B Testing
```typescript
// Test optimized vs verbose
const scenarios = [
  'Implement Firebase auth',
  'Fix TypeScript error',
  'Refactor component',
  'Add unit tests',
  'Create PR'
];

// Run with monolithic CLAUDE.md
const baseline = runScenarios(scenarios, { claudemd: 'verbose' });

// Run with minimal CLAUDE.md + Skills
const optimized = runScenarios(scenarios, { claudemd: 'minimal' });

// Compare
assert(optimized.tokenUsage < baseline.tokenUsage * 0.3);  // 70% reduction
assert(optimized.quality >= baseline.quality);  // No quality loss
```

## Integration with Other Rules

### Synergy with Progressive Disclosure
- Minimal CLAUDE.md (this rule)
- + Skills loaded on-demand (progressive-disclosure.md)
- = Maximum token efficiency

### Synergy with Reference Not Duplicate
- Minimal CLAUDE.md (this rule)
- + @imports for details (reference-not-duplicate.md)
- = Load full context only when needed

### Synergy with Structure Over Prose
- Minimal CLAUDE.md (this rule)
- + Tables/lists instead of prose (structure-over-prose.md)
- = Maximum information density

## Cost-Benefit Analysis

### Investment
- Time: 2-4 hours to optimize CLAUDE.md
- Effort: Medium (requires careful editing)
- Risk: Low (easily reversible)

### Return
- Token savings: 60-80% (validated by research)
- Cost savings: Proportional to token savings
- Performance: Faster processing
- Clarity: Often improves with conciseness

### ROI
```
Baseline: 100 tasks/day × 2,000 tokens/task × $0.005/1k = $1/day = $30/month

After 75% reduction:
100 tasks/day × 500 tokens/task × $0.005/1k = $0.25/day = $7.50/month
Savings: $22.50/month (75%)

Annual savings: $270/year
Time invested: 4 hours
ROI: $270 / 4 hours = $67.50/hour ⭐⭐⭐⭐⭐
```

## Examples from Research

### Example 1: Chorus App (1,600 tokens)
Source: .ai-knowledge/research/claude/CLAUDE.md:323-360

**Analysis:**
- Original: 1,200 words, ~1,600 tokens
- Could be reduced to: ~400 tokens with structure + skills
- **75% reduction achievable**

### Example 2: Pheromind Token Budget
Source: .ai-knowledge/research/pheromind/learning-token-budget-management.md

**System Instructions:** 2,000 tokens (1.5% of 128k budget)
- Our target: 500 tokens (0.4% of 128k budget)
- **4x more efficient than Pheromind's already-optimized approach**

## Continuous Improvement

### Quarterly Review
Every 3 months:
1. Measure current token usage
2. Audit for new redundancies
3. Check for compression opportunities
4. Compare against industry benchmarks
5. Update if 10%+ savings available

### Token Dashboard
Track in `.ai-knowledge/metrics/token-usage.json`:
```json
{
  "claudemd": {
    "tokens": 487,
    "target": 500,
    "status": "✅ Within budget",
    "lastOptimized": "2025-11-04",
    "historicalSavings": "73%"
  }
}
```

## References

1. **Claude Research**: `.ai-knowledge/research/claude/CLAUDE.md` (lines 12, 478-490)
2. **Token Efficiency BP**: `.ai-knowledge/research/synthlang-prompt-optimization/best-practices-token-efficiency.md`
3. **Token Budget Management**: `.ai-knowledge/research/pheromind/learning-token-budget-management.md`
4. **Master Synthesis**: `.ai-knowledge/research/MASTER-SYNTHESIS.md` (Principle 1: MINIMIZE)

## Status
- **Research-Backed**: ✅ Multiple sources
- **Validated**: ✅ Industry evidence (80-98% savings)
- **Implementation**: Ready
- **Priority**: CRITICAL
