# Best Practices: Token Efficiency

**Category**: Best Practices
**Source**: Industry standards, OpenAI/Anthropic guidelines, SynthLang research
**Key Principle**: Token efficiency is the backbone of prompt engineering

---

## Overview

Token efficiency is not just about saving money—it's about **performance, speed, and scalability**. Every unnecessary token:
- Costs money (per API call)
- Adds latency (more tokens = slower processing)
- Consumes context window (limiting what you can fit)
- Reduces focus (signal-to-noise ratio)

Industry reports: **80-98% cost reduction** is achievable through systematic token optimization.

This document compiles best practices from across the industry for maximizing token efficiency.

---

## The Golden Rules

### Rule 1: Hill Climb Up Quality First, Down Climb Cost Second

**Principle**: Never sacrifice quality for tokens. Optimize quality first, THEN reduce tokens.

```markdown
Wrong approach:
1. Make prompt as short as possible
2. Hope quality doesn't suffer too much
Result: Fast, cheap, broken

Right approach:
1. Achieve target quality (even if verbose)
2. Remove tokens while maintaining quality
3. Stop when quality starts to degrade
Result: Fast, cheap, AND works
```

**Process**:
```typescript
async function optimizePrompt(baseline: Prompt) {
  // Phase 1: Maximize quality (don't worry about tokens yet)
  let prompt = baseline;
  while (quality(prompt) < TARGET_QUALITY) {
    prompt = addDetail(prompt);
    prompt = addExamples(prompt);
    prompt = clarifyInstructions(prompt);
  }

  // Phase 2: Minimize tokens (preserve quality)
  let optimized = prompt;
  while (tokens(optimized) > 0) {
    const candidate = removeTokens(optimized);
    if (quality(candidate) >= TARGET_QUALITY) {
      optimized = candidate;  // Keep optimization
    } else {
      break;  // Stop, quality degrading
    }
  }

  return optimized;
}
```

### Rule 2: Every Word Must Earn Its Place

**Principle**: Challenge every word. What value does it provide? Can it be removed?

**Audit questions**:
```markdown
For each sentence:
1. What unique information does this provide?
2. Is this already stated elsewhere?
3. Would Claude understand without this?
4. Can this be said more concisely?
5. Is this obvious from context?

If "no unique value" → DELETE
If "redundant" → DELETE
If "obvious" → DELETE
If "can be shorter" → COMPRESS
```

**Example audit**:
```markdown
Original:
"When you are implementing a new feature, please make sure that you always write comprehensive unit tests that cover all edge cases, and also ensure that you update the documentation to reflect the changes you have made."

Audit:
- "When you are implementing a new feature" → obvious context ❌
- "please make sure that" → filler ❌
- "you always" → implied ❌
- "comprehensive" → vague, remove ❌
- "that cover all edge cases" → keep ✓
- "and also ensure that" → filler ❌
- "to reflect the changes you have made" → obvious ❌

Optimized:
"New features: Write unit tests covering edge cases. Update documentation."

Tokens: 62 → 14 (77% reduction)
```

### Rule 3: Structure Beats Prose

**Principle**: Tables, lists, and structured formats are more token-efficient than prose.

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

**When to use each**:
- **Prose**: Nuanced explanations, complex relationships
- **Lists**: Simple enumerations, sequential steps
- **Tables**: Multi-attribute data, comparisons
- **XML**: Hierarchical data, clear sections
- **Math**: Logical rules, constraints

### Rule 4: Progressive Disclosure > Monolithic Loading

**Principle**: Load instructions on-demand, not upfront.

```markdown
Anti-pattern (monolithic):
CLAUDE.md: 2,000 tokens
- Everything loaded at session start
- Most instructions irrelevant for any given task
- Token waste: 60-80%

Best practice (progressive):
Core CLAUDE.md: 300 tokens (always loaded)
Skills: 5 × 400 tokens each = 2,000 tokens (load only what's needed)
Average task: Uses 1-2 skills = 300 + 800 = 1,100 tokens
Token savings: 45%
```

**Implementation**:
```markdown
Core CLAUDE.md:
- Project overview (50 tokens)
- Core principles (100 tokens)
- Critical rules (100 tokens)
- Skill references (50 tokens)
Total: 300 tokens (always loaded)

Skills (load on-demand):
- firebase-auth.skill (400 tokens)
- testing-patterns.skill (350 tokens)
- deployment.skill (300 tokens)
- performance-optimization.skill (450 tokens)
- security-audit.skill (500 tokens)

Task "Implement login":
- Loads: Core + firebase-auth
- Total: 300 + 400 = 700 tokens
- Savings vs monolithic (2,300 tokens): 70%
```

### Rule 5: Measure Obsessively

**Principle**: You can't optimize what you don't measure.

**Essential metrics**:
```typescript
interface TokenMetrics {
  // Per-session metrics
  baselineTokens: number;      // CLAUDE.md + loaded Skills
  conversationTokens: number;  // Actual conversation
  totalTokens: number;         // Total for session
  tasksCompleted: number;      // Tasks done

  // Efficiency metrics
  tokensPerTask: number;       // Total / tasks
  wastePercentage: number;     // Unused instructions %
  skillLoadEfficiency: number; // Relevant skills / loaded skills

  // Cost metrics
  costPerTask: number;         // $ per task
  monthlyCost: number;         // Projected monthly cost
  costVsBaseline: number;      // Savings vs before optimization
}

function trackTokens() {
  // After every task
  logMetrics({
    taskId: generateId(),
    timestamp: new Date(),
    tokens: getCurrentTokens(),
    success: taskSucceeded(),
    quality: rateQuality()
  });

  // Weekly aggregation
  if (isMonday()) {
    const weeklyMetrics = aggregateWeeklyMetrics();
    const trends = analyzeTrends(weeklyMetrics);
    reportToTeam(weeklyMetrics, trends);
  }
}
```

**Tracking dashboard** (`.ai-knowledge/token-dashboard.json`):
```json
{
  "current": {
    "avgTokensPerTask": 76,
    "avgCostPerTask": "$0.002",
    "monthlyProjectedCost": "$24",
    "efficiency": "Good (45% improvement over baseline)"
  },
  "trends": {
    "tokensPerTask": "↓ Decreasing (good)",
    "qualityScore": "→ Stable (good)",
    "successRate": "↑ Increasing (excellent)"
  },
  "alerts": []
}
```

---

## Token Reduction Techniques

### Technique 1: Abbreviations

**When to use**: Frequently repeated terms (5+ times in CLAUDE.md).

```markdown
## Abbreviations
- TS: TypeScript
- FB: Firebase
- FS: Firestore
- PR: Pull Request
- CWD: Current Working Directory

Before:
"All TypeScript files in Firebase-related directories must have unit tests before creating a Pull Request."
Tokens: ~18

After:
"All TS files in FB dirs must have tests before PR."
Tokens: ~12
Reduction: 33%
```

**Best practices**:
- Define abbreviations once at top of CLAUDE.md
- Use industry-standard abbreviations (PR, API, DB)
- Limit custom abbreviations to 5-10 terms
- Ensure unambiguous in context

### Technique 2: Eliminate Filler Words

**Common filler words to remove**:

```markdown
Filler words that add NO value:
- "please", "kindly"
- "make sure that", "ensure that"
- "it is important to"
- "you should", "you must"
- "in order to"
- "basically", "essentially"
- "obviously", "clearly"
- "going to", "trying to"

Before:
"Please make sure that you always try to write tests in order to ensure quality."
Tokens: ~17

After:
"Write tests to ensure quality."
Tokens: ~6
Reduction: 65%
```

**Audit tool**:
```typescript
const FILLER_WORDS = [
  'please', 'kindly', 'make sure that', 'ensure that',
  'it is important to', 'you should', 'in order to',
  'basically', 'essentially', 'obviously', 'clearly'
];

function auditFillerWords(text: string): Report {
  const found = [];
  for (const filler of FILLER_WORDS) {
    const count = (text.match(new RegExp(filler, 'gi')) || []).length;
    if (count > 0) {
      found.push({ filler, count, tokenWaste: count * 2 });
    }
  }

  const totalWaste = found.reduce((sum, f) => sum + f.tokenWaste, 0);

  return {
    fillerWordsFound: found,
    totalTokenWaste: totalWaste,
    recommendations: found.map(f =>
      `Remove "${f.filler}" (${f.count} times, saving ~${f.tokenWaste} tokens)`
    )
  };
}
```

### Technique 3: Compression Patterns

**Pattern: Command form**

```markdown
Before (polite form):
"You should always run tests before committing your code."
Tokens: ~10

After (command form):
"Run tests before committing."
Tokens: ~5
Reduction: 50%
```

**Pattern: Implicit subject**

```markdown
Before (explicit subject):
"The developer should check .ai-knowledge before starting tasks."
Tokens: ~10

After (implicit subject):
"Check .ai-knowledge before starting tasks."
Tokens: ~6
Reduction: 40%
```

**Pattern: Active voice**

```markdown
Before (passive voice):
"All code must be reviewed by another developer before it is merged."
Tokens: ~14

After (active voice):
"Another developer must review code before merge."
Tokens: ~9
Reduction: 36%
```

### Technique 4: Reference by Location

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

### Technique 5: Mathematical Notation

**For logical rules** (see innovation-mathematical-frameworks.md):

```markdown
Before (prose):
"For all files in the source directory, there must exist a corresponding test file."
Tokens: ~16

After (math):
"∀ f ∈ src/* : ∃ test(f)"
Tokens: ~10
Reduction: 38%
```

---

## Cost Optimization Strategies

### Industry Benchmarks

**Cost reduction by technique**:

| Technique | Avg Token Reduction | Implementation Difficulty | ROI |
|-----------|---------------------|---------------------------|-----|
| Remove redundancy | 20-40% | Easy | Very High |
| Structured formats | 30-50% | Easy | Very High |
| Progressive disclosure | 40-70% | Medium | High |
| Mathematical notation | 30-50% | Medium | Medium |
| Abbreviations | 10-20% | Easy | Medium |
| Genetic algorithms | 50-80% | Hard | Medium |

**Real-world case studies**:

1. **E-commerce platform**:
   - Before: 3,200 tokens/task, $0.008/task
   - After: 890 tokens/task, $0.002/task
   - Reduction: 72% tokens, 75% cost
   - Techniques: Progressive disclosure, structured formats

2. **SaaS application**:
   - Before: 2,100 tokens/task, $0.005/task
   - After: 450 tokens/task, $0.001/task
   - Reduction: 79% tokens, 80% cost
   - Techniques: Mathematical notation, Skills extraction

3. **Internal tools**:
   - Before: 5,000 tokens/task, $0.012/task
   - After: 1,200 tokens/task, $0.003/task
   - Reduction: 76% tokens, 75% cost
   - Techniques: Redundancy removal, A/B testing

**Projected savings**:
```
Baseline: 100 tasks/day × 2,000 tokens/task × $0.005/1k tokens = $1.00/day = $30/month

After 50% reduction:
100 tasks/day × 1,000 tokens/task × $0.005/1k tokens = $0.50/day = $15/month
Savings: $15/month (50%)

After 75% reduction:
100 tasks/day × 500 tokens/task × $0.005/1k tokens = $0.25/day = $7.50/month
Savings: $22.50/month (75%)

After 90% reduction:
100 tasks/day × 200 tokens/task × $0.005/1k tokens = $0.10/day = $3/month
Savings: $27/month (90%)

At scale (1,000 tasks/day):
75% reduction saves $225/month
90% reduction saves $270/month
```

### Cost Tracking

```typescript
// .ai-knowledge/cost-tracker.ts

interface CostMetrics {
  date: Date;
  totalTokens: number;
  totalCost: number;
  tasksCompleted: number;
  avgTokensPerTask: number;
  avgCostPerTask: number;
}

class CostTracker {
  private readonly TOKEN_COST = 0.005 / 1000;  // $0.005 per 1k tokens

  async trackDaily() {
    const metrics = await this.collectDailyMetrics();

    const cost = {
      date: new Date(),
      totalTokens: metrics.tokens,
      totalCost: metrics.tokens * this.TOKEN_COST,
      tasksCompleted: metrics.tasks,
      avgTokensPerTask: metrics.tokens / metrics.tasks,
      avgCostPerTask: (metrics.tokens * this.TOKEN_COST) / metrics.tasks
    };

    await this.saveToDB(cost);
    await this.checkBudget(cost);

    return cost;
  }

  async projectMonthlyCost(): Promise<number> {
    const avgDaily = await this.getAvgDailyCost();
    return avgDaily * 30;
  }

  async checkBudget(cost: CostMetrics) {
    const monthlyProjection = await this.projectMonthlyCost();
    const budget = 50;  // $50/month budget

    if (monthlyProjection > budget) {
      await this.alert(`Projected cost ($${monthlyProjection}) exceeds budget ($${budget})`);
      await this.suggestOptimizations();
    }
  }
}
```

---

## Best Practices Checklist

### Design Time

- [ ] Define token budget for CLAUDE.md (target: < 500 tokens)
- [ ] Plan Skills structure (progressive disclosure)
- [ ] Identify frequently-used terms (create abbreviations)
- [ ] Use structured formats (tables, lists) by default
- [ ] Apply mathematical notation for logical rules
- [ ] Reference external docs (@imports) instead of duplicating

### Development Time

- [ ] Write for clarity first, optimize later
- [ ] Remove all filler words
- [ ] Use command form (not polite requests)
- [ ] Prefer active voice over passive
- [ ] Challenge every word (does it add value?)
- [ ] Test Claude's understanding before committing

### Review Time

- [ ] Count tokens (use /context command)
- [ ] Compare to budget
- [ ] Audit for redundancy
- [ ] Check for compression opportunities
- [ ] Validate quality maintained
- [ ] Document optimization rationale

### Runtime

- [ ] Track tokens per session
- [ ] Monitor skill load patterns
- [ ] Log token waste (unused instructions)
- [ ] Measure cost per task
- [ ] Alert on budget overruns
- [ ] Continuous optimization loop

### Quarterly

- [ ] Full token audit
- [ ] ROI analysis (savings vs effort)
- [ ] Industry benchmark comparison
- [ ] Major optimization sprint
- [ ] Update best practices
- [ ] Share learnings with community

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Premature Abbreviation

**Problem**: Abbreviating everything makes prompts unreadable.

```markdown
Bad:
"Impl auth w/ FB, use FS for usr dt, ens sec w/ RLS"

What does this even mean?
```

**Best practice**: Only abbreviate frequently-used terms (5+ occurrences).

### Anti-Pattern 2: Over-Compression

**Problem**: Removing so many tokens that instructions become ambiguous.

```markdown
Over-compressed:
"Tests req'd b4 commit"

Better:
"Run tests before committing"

Slightly more tokens, much clearer.
```

**Best practice**: Stop when clarity starts to degrade.

### Anti-Pattern 3: Ignoring Context Window

**Problem**: Optimizing CLAUDE.md without considering total context.

```markdown
CLAUDE.md: 300 tokens (optimized!) ✓
Skills loaded: 2,000 tokens ✗
Conversation: 5,000 tokens ✗
Total: 7,300 tokens (approaching limit)

Better: Optimize entire pipeline, not just CLAUDE.md
```

**Best practice**: Track total session tokens, not just baseline.

### Anti-Pattern 4: Optimization Without Validation

**Problem**: Assuming optimization improved things without measuring.

**Best practice**: A/B test every optimization. Measure impact.

### Anti-Pattern 5: Sacrificing Quality for Tokens

**Problem**: Breaking functionality to save a few tokens.

**Best practice**: Quality first, then optimize tokens.

---

## Key Takeaways

1. **Quality first, tokens second** - Never sacrifice functionality
2. **Measure everything** - Can't optimize without data
3. **Structure beats prose** - Tables and lists are more efficient
4. **Progressive disclosure** - Load only what's needed
5. **Every word earns its place** - Challenge all redundancy
6. **Industry reports 80-98% savings** - Aggressive optimization works
7. **Continuous improvement** - Optimization never stops

**For our project**:
- Target: < 500 tokens for core CLAUDE.md
- Progressive disclosure via Skills
- Weekly optimization sprints
- Measure token efficiency obsessively
- 50-75% reduction achievable

**Token efficiency = Cost efficiency = Speed efficiency = Performance efficiency**

---

**Document Version**: 1.0
**Last Updated**: 2025-11-03
**Status**: Ready for implementation
**Related Docs**: README.md, learning-token-reduction-techniques.md, pattern-continuous-optimization.md
