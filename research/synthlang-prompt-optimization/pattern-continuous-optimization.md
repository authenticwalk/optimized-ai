# Pattern: Continuous Optimization

**Category**: Pattern
**Source**: Industry best practices, SynthLang, DSPy, DevOps principles
**Key Pattern**: Treat prompt optimization as an ongoing process, not a one-time task

---

## Overview

**Anti-pattern**: Write CLAUDE.md once, never touch it again.

**Pattern**: **Continuous optimization loop** - Measure, analyze, hypothesize, test, learn, repeat.

The key insight: **Prompts decay over time**. As your codebase evolves, team practices change, and use cases shift, even the best prompts become outdated. Continuous optimization keeps prompts effective and efficient.

---

## The Continuous Optimization Loop

### Core Cycle

```markdown
   ┌─────────────────────────────────────────┐
   │                                         │
   │         CONTINUOUS OPTIMIZATION         │
   │                                         │
   └─────────────────────────────────────────┘
               │
               ▼
        ┌──────────────┐
        │   MEASURE    │  Collect metrics on current performance
        │   Baseline   │  Track: tokens, success rate, quality
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │   ANALYZE    │  Identify bottlenecks and opportunities
        │   Patterns   │  What's working? What's not?
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │ HYPOTHESIZE  │  Form testable hypotheses
        │ Improvements │  "If we change X, Y will improve"
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │     TEST     │  Run controlled experiments
        │  Variations  │  A/B test proposed changes
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │    LEARN     │  Document findings
        │  & Document  │  Update .ai-knowledge/
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │    APPLY     │  Implement validated changes
        │   Winners    │  Update CLAUDE.md, Skills
        └──────┬───────┘
               │
               └───────────► Back to MEASURE
```

### Weekly Optimization Sprint

**Monday: Measure & Analyze**
```markdown
1. Run standard test suite (20 tasks)
2. Measure current metrics:
   - Average tokens per session
   - Success rate
   - Quality scores
   - Adherence to instructions
   - Spin rate (got stuck?)
3. Analyze patterns:
   - Which instructions helped most?
   - Which were ignored?
   - What tasks failed?
   - Where did we waste tokens?
4. Document in .ai-knowledge/weekly-metrics/2025-11-03.json
```

**Tuesday: Hypothesize**
```markdown
Based on analysis, form 3 hypotheses:

Example hypotheses:
- H1: "Stack description can be table instead of prose → 30% token reduction"
- H2: "Adding example to firebase-auth skill → 20% higher success rate"
- H3: "Math notation for rules → 15% better adherence"

For each hypothesis:
- State expected outcome
- Define success criteria
- Identify test approach
```

**Wednesday-Thursday: Test**
```markdown
Run experiments:
- Create variations (control vs treatment)
- Test each on 5-10 relevant tasks
- Measure outcomes
- Compare to baseline
- Document results in experiments/EXPERIMENTS-LOG.md
```

**Friday: Learn & Apply**
```markdown
1. Analyze experiment results
2. Determine winners (p < 0.05)
3. Update CLAUDE.md or Skills with winners
4. Document learnings in experiments/LEARNINGS.md
5. Commit changes with detailed notes
6. Plan next week's optimizations
```

**Next Monday**: Measure new baseline, repeat cycle

---

## Measurement Framework

### Essential Metrics

**1. Token Efficiency**
```typescript
interface TokenMetrics {
  claudeMd: number;           // CLAUDE.md tokens
  skillsLoaded: number;       // Skills loaded this session
  conversationTokens: number; // Conversation tokens
  totalSession: number;       // Total tokens
  tasksCompleted: number;     // Tasks completed
  tokensPerTask: number;      // Efficiency metric
}

function measureTokenEfficiency(session: Session): TokenMetrics {
  return {
    claudeMd: countTokens(readFile('.claude/CLAUDE.md')),
    skillsLoaded: session.skills.reduce((sum, skill) =>
      sum + countTokens(skill.content), 0
    ),
    conversationTokens: session.messages.reduce((sum, msg) =>
      sum + msg.tokens, 0
    ),
    totalSession: session.totalTokens,
    tasksCompleted: session.tasks.filter(t => t.success).length,
    tokensPerTask: session.totalTokens / session.tasks.length
  };
}
```

**2. Success Rate**
```typescript
interface SuccessMetrics {
  totalTasks: number;
  successful: number;
  failed: number;
  successRate: number;      // successful / totalTasks
  qualityScore: number;     // avg(quality) for successful tasks
  timeToComplete: number;   // avg time per task
}

function measureSuccess(tasks: Task[]): SuccessMetrics {
  const successful = tasks.filter(t => t.success);
  return {
    totalTasks: tasks.length,
    successful: successful.length,
    failed: tasks.length - successful.length,
    successRate: successful.length / tasks.length,
    qualityScore: avg(successful.map(t => t.quality)),
    timeToComplete: avg(tasks.map(t => t.duration))
  };
}
```

**3. Adherence Rate**
```typescript
interface AdherenceMetrics {
  instructionsTotal: number;
  instructionsFollowed: number;
  instructionsIgnored: number;
  adherenceRate: number;
  mostFollowed: Instruction[];
  mostIgnored: Instruction[];
}

function measureAdherence(session: Session): AdherenceMetrics {
  const instructions = extractInstructions('.claude/CLAUDE.md');
  const followed = [];
  const ignored = [];

  for (const instruction of instructions) {
    if (wasFollowed(instruction, session)) {
      followed.push(instruction);
    } else {
      ignored.push(instruction);
    }
  }

  return {
    instructionsTotal: instructions.length,
    instructionsFollowed: followed.length,
    instructionsIgnored: ignored.length,
    adherenceRate: followed.length / instructions.length,
    mostFollowed: followed.slice(0, 5),
    mostIgnored: ignored.slice(0, 5)
  };
}
```

**4. Quality Metrics**
```typescript
interface QualityMetrics {
  codeQuality: number;        // 0-1: linting, formatting, best practices
  testCoverage: number;       // 0-1: % coverage
  securityScore: number;      // 0-1: vulnerabilities found
  performanceScore: number;   // 0-1: performance benchmarks
  maintainabilityIndex: number; // 0-100: maintainability
}

function measureQuality(code: string): QualityMetrics {
  return {
    codeQuality: runLinter(code).score,
    testCoverage: measureCoverage(code),
    securityScore: securityAudit(code).score,
    performanceScore: benchmarkPerformance(code),
    maintainabilityIndex: calculateMaintainabilityIndex(code)
  };
}
```

### Tracking Over Time

**Metrics database**: `.ai-knowledge/metrics.json`

```json
{
  "baseline": {
    "date": "2025-11-03",
    "tokens": {
      "claudeMd": 1456,
      "avgSkillsLoaded": 800,
      "avgSession": 2256,
      "perTask": 113
    },
    "success": {
      "rate": 0.85,
      "qualityScore": 4.2,
      "avgTime": "18min"
    },
    "adherence": {
      "rate": 0.78
    }
  },
  "history": [
    {
      "date": "2025-11-10",
      "optimization": "Removed redundancy",
      "tokens": {
        "claudeMd": 1020,
        "avgSkillsLoaded": 800,
        "avgSession": 1820,
        "perTask": 91,
        "improvement": "19%"
      },
      "success": {
        "rate": 0.87,
        "qualityScore": 4.3,
        "avgTime": "17min",
        "improvement": "+2%"
      },
      "adherence": {
        "rate": 0.81,
        "improvement": "+3%"
      }
    },
    {
      "date": "2025-11-17",
      "optimization": "Added math notation",
      "tokens": {
        "claudeMd": 720,
        "avgSkillsLoaded": 800,
        "avgSession": 1520,
        "perTask": 76,
        "improvement": "33%"
      },
      "success": {
        "rate": 0.88,
        "qualityScore": 4.4,
        "avgTime": "16min",
        "improvement": "+3%"
      },
      "adherence": {
        "rate": 0.86,
        "improvement": "+8%"
      }
    }
  ],
  "trends": {
    "tokensPerTask": {
      "start": 113,
      "current": 76,
      "reduction": "33%",
      "trend": "improving"
    },
    "successRate": {
      "start": 0.85,
      "current": 0.88,
      "improvement": "+3%",
      "trend": "improving"
    },
    "adherenceRate": {
      "start": 0.78,
      "current": 0.86,
      "improvement": "+8%",
      "trend": "improving"
    }
  }
}
```

---

## Optimization Strategies

### Strategy 1: Hill Climbing

**Approach**: Make small incremental improvements continuously.

```typescript
async function hillClimbing(
  baseline: Prompt,
  testCases: TestCase[],
  iterations: number = 20
): Promise<Prompt> {
  let current = baseline;
  let currentScore = await evaluate(current, testCases);

  for (let i = 0; i < iterations; i++) {
    // Generate neighbor (small variation)
    const neighbor = generateSmallVariation(current);
    const neighborScore = await evaluate(neighbor, testCases);

    // If better, move to neighbor
    if (neighborScore > currentScore) {
      current = neighbor;
      currentScore = neighborScore;
      console.log(`Iteration ${i}: Improvement to ${currentScore}`);
    } else {
      console.log(`Iteration ${i}: No improvement`);
    }
  }

  return current;
}

function generateSmallVariation(prompt: Prompt): Prompt {
  // Examples of small variations:
  // - Rewrite one sentence
  // - Change format of one section
  // - Add/remove one example
  // - Adjust one parameter
  const variations = [
    () => rewriteSentence(prompt, randomSentence()),
    () => changeFormat(prompt, randomSection()),
    () => addExample(prompt, randomSkill()),
    () => removeRedundancy(prompt, findRedundancy())
  ];

  const variation = randomChoice(variations);
  return variation();
}
```

**When to use**:
- Early optimization (finding quick wins)
- Continuous incremental improvement
- Low risk tolerance

### Strategy 2: Simulated Annealing

**Approach**: Accept occasional "worse" changes to escape local optima.

```typescript
async function simulatedAnnealing(
  baseline: Prompt,
  testCases: TestCase[],
  initialTemp: number = 1.0,
  coolingRate: number = 0.95,
  iterations: number = 50
): Promise<Prompt> {
  let current = baseline;
  let currentScore = await evaluate(current, testCases);
  let best = current;
  let bestScore = currentScore;
  let temperature = initialTemp;

  for (let i = 0; i < iterations; i++) {
    const neighbor = generateVariation(current);
    const neighborScore = await evaluate(neighbor, testCases);

    // Always accept if better
    if (neighborScore > currentScore) {
      current = neighbor;
      currentScore = neighborScore;

      if (neighborScore > bestScore) {
        best = neighbor;
        bestScore = neighborScore;
      }
    } else {
      // Sometimes accept if worse (probability decreases with temperature)
      const delta = neighborScore - currentScore;
      const acceptanceProbability = Math.exp(delta / temperature);

      if (Math.random() < acceptanceProbability) {
        current = neighbor;
        currentScore = neighborScore;
        console.log(`Accepted worse solution (temp=${temperature.toFixed(2)})`);
      }
    }

    // Cool down
    temperature *= coolingRate;
  }

  return best;
}
```

**When to use**:
- When stuck in local optimum
- Exploring radical changes
- Later-stage optimization

### Strategy 3: A/B Testing

**Approach**: Compare variations head-to-head systematically.

```typescript
interface ABTest {
  name: string;
  hypothesis: string;
  controlVersion: Prompt;
  treatmentVersion: Prompt;
  testCases: TestCase[];
  sampleSize: number;
}

async function runABTest(test: ABTest): Promise<ABTestResult> {
  const controlResults = [];
  const treatmentResults = [];

  // Run test cases on both versions
  for (let i = 0; i < test.sampleSize; i++) {
    const testCase = randomChoice(test.testCases);

    // Run on control
    const controlResult = await runTask(test.controlVersion, testCase);
    controlResults.push(controlResult);

    // Run on treatment
    const treatmentResult = await runTask(test.treatmentVersion, testCase);
    treatmentResults.push(treatmentResult);
  }

  // Statistical analysis
  const controlMean = mean(controlResults.map(r => r.score));
  const treatmentMean = mean(treatmentResults.map(r => r.score));
  const pValue = tTest(controlResults, treatmentResults);
  const improvement = (treatmentMean - controlMean) / controlMean;

  return {
    name: test.name,
    hypothesis: test.hypothesis,
    controlMean,
    treatmentMean,
    improvement,
    pValue,
    significant: pValue < 0.05,
    winner: pValue < 0.05 && treatmentMean > controlMean ? 'treatment' : 'control',
    recommendation: pValue < 0.05 && treatmentMean > controlMean
      ? 'Deploy treatment version'
      : 'Keep control version'
  };
}
```

**Example A/B test**:
```markdown
## A/B Test: Stack Section Format

Hypothesis: Table format reduces tokens by 30% vs prose

Control (prose):
"Our stack includes TypeScript with strict mode for type safety, Firebase for authentication, Firestore for database, and Svelte + HTMX for frontend."
Tokens: 32

Treatment (table):
| Component | Technology |
|-----------|------------|
| Language | TypeScript |
| Auth | Firebase |
| DB | Firestore |
| Frontend | Svelte + HTMX |
Tokens: 22

Test: Run 20 tasks, 10 with each version

Results:
- Control: avg quality 4.2, success rate 85%
- Treatment: avg quality 4.3, success rate 87%
- p-value: 0.03 (significant!)
- Token reduction: 31%

Conclusion: Deploy treatment (table format)
```

### Strategy 4: Multi-Armed Bandit

**Approach**: Balance exploration (try new things) vs exploitation (use what works).

```typescript
interface Arm {
  name: string;
  prompt: Prompt;
  successCount: number;
  trialCount: number;
  estimatedValue: number;
}

function selectArm(arms: Arm[], explorationRate: number = 0.1): Arm {
  // Epsilon-greedy strategy
  if (Math.random() < explorationRate) {
    // Explore: random choice
    return randomChoice(arms);
  } else {
    // Exploit: best performing
    return arms.reduce((best, arm) =>
      arm.estimatedValue > best.estimatedValue ? arm : best
    );
  }
}

async function multiarmedBandit(
  variations: Prompt[],
  testCases: TestCase[],
  budget: number = 100
): Promise<Prompt> {
  const arms: Arm[] = variations.map(prompt => ({
    name: prompt.name,
    prompt,
    successCount: 0,
    trialCount: 0,
    estimatedValue: 0.5  // Start with neutral estimate
  }));

  // Run trials up to budget
  for (let trial = 0; trial < budget; trial++) {
    // Select arm
    const arm = selectArm(arms);

    // Run test
    const testCase = randomChoice(testCases);
    const result = await runTask(arm.prompt, testCase);

    // Update statistics
    arm.trialCount++;
    if (result.success) arm.successCount++;
    arm.estimatedValue = arm.successCount / arm.trialCount;

    console.log(`Trial ${trial}: ${arm.name} - Value: ${arm.estimatedValue.toFixed(2)}`);
  }

  // Return best arm
  return arms.reduce((best, arm) =>
    arm.estimatedValue > best.estimatedValue ? arm : best
  ).prompt;
}
```

**When to use**:
- Testing multiple variations simultaneously
- Limited test budget
- Want to converge to winner quickly

---

## Feedback Loops

### User Feedback Loop

**Pattern**: Capture feedback from developers using the system.

```markdown
## After Task Completion

Prompt developer:
1. Did Claude follow the instructions? (Y/N)
2. Rate quality (1-5): [    ]
3. What worked well? [              ]
4. What didn't work? [              ]
5. Suggestions: [                   ]

Automatically log:
{
  "taskId": "t-2025-11-03-001",
  "userFeedback": {
    "followedInstructions": true,
    "quality": 5,
    "workedWell": "Firebase auth skill was perfect",
    "didntWork": "Should have mentioned session persistence earlier",
    "suggestions": "Add session management to firebase-auth skill"
  },
  "timestamp": "2025-11-03T14:30:00Z"
}

Weekly review:
- Aggregate feedback
- Identify common themes
- Prioritize improvements
- Update CLAUDE.md or Skills
```

### Automated Feedback Loop

**Pattern**: System monitors and self-adjusts.

```typescript
// experiments/runner/auto-optimize.ts

class AutoOptimizer {
  private metrics: MetricsCollector;
  private threshold = {
    successRate: 0.90,       // If below 90%, investigate
    tokensPerTask: 100,      // If above 100, optimize
    adherenceRate: 0.85,     // If below 85%, clarify instructions
    qualityScore: 4.0        // If below 4.0, improve guidance
  };

  async monitor() {
    // Collect metrics weekly
    const metrics = await this.metrics.collect();

    // Check thresholds
    if (metrics.successRate < this.threshold.successRate) {
      await this.investigateFailures();
    }

    if (metrics.tokensPerTask > this.threshold.tokensPerTask) {
      await this.optimizeTokenUsage();
    }

    if (metrics.adherenceRate < this.threshold.adherenceRate) {
      await this.clarifyInstructions();
    }

    if (metrics.qualityScore < this.threshold.qualityScore) {
      await this.improveGuidance();
    }
  }

  private async investigateFailures() {
    // Analyze failed tasks
    const failures = await this.metrics.getFailedTasks();
    const patterns = this.analyzePatterns(failures);

    // Generate hypotheses
    const hypotheses = this.generateHypotheses(patterns);

    // Create experiments
    for (const hypothesis of hypotheses) {
      await this.createExperiment(hypothesis);
    }

    // Alert team
    await this.notifyTeam('Failure rate high - experiments created');
  }

  private async optimizeTokenUsage() {
    // Find token-heavy sections
    const sections = await this.metrics.getTokenUsageBySections();
    const heavy = sections.filter(s => s.tokens > 100);

    // Suggest optimizations
    for (const section of heavy) {
      const optimized = await this.suggestOptimizations(section);
      await this.createPR({
        title: `Optimize ${section.name} (save ${optimized.tokensSaved} tokens)`,
        changes: optimized.changes,
        rationale: optimized.rationale
      });
    }
  }
}

// Run weekly
setInterval(() => new AutoOptimizer().monitor(), 7 * 24 * 60 * 60 * 1000);
```

### Learning Loop

**Pattern**: System captures and applies learnings automatically.

```markdown
## Learning Loop Process

1. CAPTURE: After every task
   - What worked?
   - What didn't?
   - New patterns discovered?
   - Edge cases encountered?

2. ANALYZE: Weekly
   - Aggregate all learnings
   - Identify frequent patterns
   - Find common pitfalls
   - Recognize successful approaches

3. CODIFY: Update knowledge base
   - Add to .ai-knowledge/patterns.json
   - Update relevant Skills
   - Refine CLAUDE.md instructions
   - Document in experiments/LEARNINGS.md

4. APPLY: Use in future tasks
   - System references patterns automatically
   - Claude applies learned approaches
   - Success rate improves
   - Fewer repeated mistakes

5. MEASURE: Track improvement
   - Compare tasks before vs after learning
   - Measure pattern reuse rate
   - Track quality improvement
   - Validate learning effectiveness
```

---

## Optimization Checklist

### Daily
- [ ] Log task outcomes (success/failure, quality)
- [ ] Note any instruction adherence issues
- [ ] Capture new patterns or insights
- [ ] Quick wins: Fix obvious issues immediately

### Weekly
- [ ] Run standard test suite (20 tasks)
- [ ] Collect and analyze metrics
- [ ] Form 3 optimization hypotheses
- [ ] Run A/B tests on hypotheses
- [ ] Deploy validated improvements
- [ ] Update experiments/LEARNINGS.md

### Monthly
- [ ] Comprehensive metric review
- [ ] Trend analysis (improving or declining?)
- [ ] Major optimization sprint
- [ ] Genetic algorithm run on CLAUDE.md
- [ ] Skill quality audit
- [ ] Team retrospective on AI assistance

### Quarterly
- [ ] Full system optimization
- [ ] Compare to industry benchmarks
- [ ] ROI analysis (cost savings, productivity)
- [ ] Major version update to CLAUDE.md
- [ ] Documentation update
- [ ] Community knowledge sharing

---

## Anti-Patterns to Avoid

### 1. Set and Forget

**Anti-pattern**: Write CLAUDE.md once, never update it.

**Problem**: Prompts become stale as codebase evolves.

**Solution**: Continuous optimization loop (weekly updates).

### 2. Optimization Without Measurement

**Anti-pattern**: "This seems better" without data.

**Problem**: Don't know if changes actually improve things.

**Solution**: Measure before and after every change. Require statistical significance.

### 3. Premature Optimization

**Anti-pattern**: Optimize before understanding baseline.

**Problem**: Don't know what to optimize or how to measure success.

**Solution**: Establish baseline metrics first. Measure for 2 weeks before optimizing.

### 4. Over-Optimization

**Anti-pattern**: Optimize every single word obsessively.

**Problem**: Diminishing returns, time wasted on micro-optimizations.

**Solution**: Focus on high-impact changes (80/20 rule). Stop when improvements < 5%.

### 5. Siloed Optimization

**Anti-pattern**: Optimize CLAUDE.md without considering Skills, or vice versa.

**Problem**: Local optimization might hurt global performance.

**Solution**: Optimize holistically. Consider entire system (CLAUDE.md + Skills + Agents).

### 6. Ignoring Quality

**Anti-pattern**: Optimize for tokens only, ignore quality degradation.

**Problem**: Ship faster but broken code.

**Solution**: Multi-objective optimization. Quality is always the top priority.

---

## Success Metrics

### North Star Metrics

**1. Value Delivered Per Token**
```
Value = (tasks_completed × avg_quality) / tokens_used

Target: Increase by 50% over 6 months
```

**2. Developer Productivity**
```
Productivity = tasks_completed / developer_time

Target: 2x improvement over baseline
```

**3. Cost Efficiency**
```
Cost_efficiency = value_delivered / API_cost

Target: 80% cost reduction
```

### Leading Indicators

- Token reduction week-over-week
- Adherence rate improvement
- Skill reuse rate increasing
- Experiment success rate
- Team satisfaction scores

### Lagging Indicators

- Total cost savings
- Productivity improvement
- Quality improvement
- Onboarding time reduction
- Team adoption rate

---

## Key Takeaways

1. **Continuous, not one-time** - Optimization never stops
2. **Measure everything** - Can't improve what you don't measure
3. **Small iterations** - Weekly improvements compound
4. **Data-driven** - Rely on metrics, not intuition
5. **Feedback loops** - Learn from every task

**For our project**:
- ✅ Weekly optimization sprints
- ✅ Automated metric collection
- ✅ A/B testing framework
- ✅ Learning loops built-in
- ✅ Track improvements over time

**Expected outcome**: Continuous improvement of 5-10% per month in efficiency and quality.

---

**Document Version**: 1.0
**Last Updated**: 2025-11-03
**Status**: Ready for implementation
**Related Docs**: README.md, pattern-scientific-validation.md, best-practices-token-efficiency.md
