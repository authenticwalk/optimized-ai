# Experimental Validation Framework

## Core Principle

**This project must be defensible with empirical evidence, not theory.**

Every optimization, pattern, and technique must be validated through rigorous experimentation using the scientific method. We don't just write prompts and hope they work - we test them, measure results, iterate, and prove improvement.

## Scientific Method Applied to AI Optimization

### 1. Hypothesis Formation
For each optimization:
- State the problem clearly
- Propose a solution (optimization/prompt/pattern)
- Make a measurable prediction
- Define success criteria

### 2. Experimental Design
- Define control (baseline behavior)
- Define treatment (optimized behavior)
- Identify variables to measure
- Define multiple test scenarios
- Plan A/B tests

### 3. Execution
- Run experiments with actual Claude Code CLI
- Record all inputs and outputs
- Capture metrics
- Document observations
- Multiple runs to validate consistency

### 4. Analysis
- Compare control vs treatment
- Statistical significance (if applicable)
- Qualitative assessment
- Identify confounding factors

### 5. Iteration
- If improved: Document and adopt
- If not improved: Analyze why, revise hypothesis
- If inconclusive: Refine experiment
- Repeat until validated

### 6. Documentation
- Every experiment recorded
- Results preserved
- Learnings captured
- Failures documented (as important as successes)

## Experimental Infrastructure

### Command-Line Test Harness

**Goal**: Run Claude Code from CLI with prompts, capture results, validate outcomes

**Components:**

1. **Experiment Runner** (`experiments/runner/`)
   - Execute Claude Code CLI with specific prompts
   - Capture stdout/stderr
   - Record timing
   - Save generated code
   - Track tool calls made
   - Measure token usage

2. **Scenario Library** (`experiments/scenarios/`)
   - Common tasks (auth, CRUD, API integration, etc.)
   - Edge cases (error handling, race conditions, etc.)
   - Complex scenarios (multi-file refactoring, etc.)
   - Real-world problems from past experience

3. **Validation Engine** (`experiments/validation/`)
   - Check if code works (tests pass)
   - Check if code compiles
   - Check if requirements met
   - Measure code quality (linting, complexity)
   - Detect spin/confusion patterns
   - Measure efficiency (time, tokens, tool calls)

4. **A/B Testing Framework** (`experiments/ab-testing/`)
   - Run same scenario with control vs treatment
   - Randomize order to avoid bias
   - Multiple runs for statistical validity
   - Compare outcomes objectively

5. **Results Database** (`experiments/results/`)
   - JSON records of all experiments
   - Searchable and queryable
   - Trend analysis over time
   - Success/failure rates

### Experiment Record Format

```json
{
  "experimentId": "exp-001-spin-detection",
  "hypothesis": "Adding spin-detection prompt reduces repetitive edits by 50%",
  "date": "2025-11-03",
  "scenario": "auth-implementation",
  "treatments": {
    "control": {
      "config": "baseline-cursorrules",
      "prompt": "Implement user authentication with Firebase",
      "runs": [
        {
          "runId": "control-run-1",
          "duration": 180,
          "tokensUsed": 45000,
          "toolCalls": 23,
          "fileEdits": 8,
          "repetitiveEdits": 5,
          "testsPass": true,
          "linterClean": false,
          "spunOut": true,
          "codeQuality": 7.5,
          "artifacts": ["control-run-1/generated-code/"]
        }
      ],
      "avgMetrics": {
        "duration": 180,
        "tokensUsed": 45000,
        "repetitiveEdits": 5,
        "spunOut": 1.0
      }
    },
    "treatment": {
      "config": "with-spin-detection",
      "prompt": "Implement user authentication with Firebase",
      "runs": [
        {
          "runId": "treatment-run-1",
          "duration": 120,
          "tokensUsed": 30000,
          "toolCalls": 18,
          "fileEdits": 5,
          "repetitiveEdits": 1,
          "testsPass": true,
          "linterClean": true,
          "spunOut": false,
          "codeQuality": 8.5,
          "artifacts": ["treatment-run-1/generated-code/"]
        }
      ],
      "avgMetrics": {
        "duration": 120,
        "tokensUsed": 30000,
        "repetitiveEdits": 1,
        "spunOut": 0.0
      }
    }
  },
  "analysis": {
    "improvement": {
      "duration": "-33%",
      "tokensUsed": "-33%",
      "repetitiveEdits": "-80%",
      "spunOut": "eliminated"
    },
    "conclusion": "Hypothesis validated. Spin detection significantly reduces wasted effort.",
    "confidence": "high",
    "notes": "Treatment produced cleaner code and better linting results as bonus."
  },
  "decision": "ADOPT - Add to core system"
}
```

## Key Metrics to Measure

### Efficiency Metrics:
- **Time to completion** - How long did task take?
- **Token usage** - Cost/efficiency
- **Tool call count** - Overhead
- **File edit count** - Churn
- **Repetitive edits** - Spinning indicator

### Quality Metrics:
- **Tests pass** - Does it work?
- **Linter clean** - Code quality
- **Requirements met** - Did it solve the problem?
- **Code complexity** - Maintainability
- **Boilerplate ratio** - Unnecessary code

### Behavioral Metrics:
- **Spin detected** - Got stuck?
- **Errors repeated** - Learning failure?
- **Pattern compliance** - Followed rules?
- **Manual intervention needed** - Had to correct?

### Learning Metrics:
- **Pattern reuse** - Applied learnings?
- **Failure avoidance** - Didn't repeat mistakes?
- **Improvement over time** - Getting better?

## Test Scenarios Library

### Category: Basic CRUD Operations
1. **Simple Create** - Add user profile model
2. **Simple Read** - List all users with pagination
3. **Simple Update** - Edit user profile
4. **Simple Delete** - Soft delete user

### Category: Authentication
1. **Firebase Auth** - Email/password sign up and login
2. **Social Auth** - Google OAuth integration
3. **Protected Routes** - Middleware for auth checking
4. **Session Management** - Token refresh logic

### Category: Database Operations
1. **Firestore Queries** - Complex where clauses
2. **Supabase RLS** - Row-level security policies
3. **Transactions** - Multi-document atomic updates
4. **Real-time Listeners** - Subscribe to changes

### Category: Edge Cases
1. **Race Conditions** - Handle concurrent updates
2. **Error Recovery** - Graceful failure handling
3. **Validation** - Input sanitization and validation
4. **Rate Limiting** - API throttling

### Category: Refactoring
1. **Extract Function** - DRY up duplicate code
2. **Rename Symbol** - Change function name throughout
3. **Move File** - Reorganize file structure
4. **Type Safety** - Add TypeScript types to JS

### Category: Complex Tasks
1. **Multi-file Feature** - Add feature across 5+ files
2. **API Integration** - Integrate 3rd party API
3. **State Management** - Add Svelte stores
4. **Testing Suite** - Write comprehensive tests

### Category: Real-World Problems
1. **Debug Production Issue** - Fix reported bug
2. **Performance Optimization** - Reduce load time
3. **Security Patch** - Fix vulnerability
4. **Dependency Update** - Upgrade major version

## Experiment Workflow

### Phase 1: Design Experiment
```bash
# Create experiment definition
experiments/runner create-experiment \
  --id exp-001-spin-detection \
  --hypothesis "Spin detection reduces repetitive edits" \
  --scenario auth-implementation \
  --control baseline \
  --treatment with-spin-detection
```

### Phase 2: Run Experiments
```bash
# Run control
experiments/runner run \
  --experiment exp-001-spin-detection \
  --treatment control \
  --runs 5

# Run treatment
experiments/runner run \
  --experiment exp-001-spin-detection \
  --treatment treatment \
  --runs 5
```

### Phase 3: Validate Results
```bash
# Validate each run
experiments/validator validate \
  --experiment exp-001-spin-detection \
  --check-tests \
  --check-linting \
  --check-requirements
```

### Phase 4: Analyze
```bash
# Compare control vs treatment
experiments/analyzer compare \
  --experiment exp-001-spin-detection \
  --metrics duration,tokens,repetitive-edits,spin \
  --confidence-level 0.95
```

### Phase 5: Document
```bash
# Generate report
experiments/reporter generate \
  --experiment exp-001-spin-detection \
  --format markdown
```

### Phase 6: Decide
- Review analysis
- Make decision: ADOPT, REJECT, REFINE
- Update system if adopted
- Document learnings

## A/B Testing Strategy

### Test Variations
For each optimization, test multiple variations:
- **A**: Baseline (no optimization)
- **B**: Proposed optimization
- **C**: Alternative approach
- **D**: Combination of techniques

### Randomization
- Randomize order of runs
- Avoid time-of-day bias
- Use different Claude versions (if testing across versions)
- Vary scenario slightly (but equivalently)

### Statistical Validity
- Minimum 5 runs per treatment
- Calculate standard deviation
- Test for statistical significance
- Consider outliers

### Qualitative Assessment
Numbers don't tell everything:
- Code readability
- Maintainability
- Architectural quality
- "Feels right" factor (from experienced dev perspective)

## Continuous Validation

### Regression Testing
When system evolves:
- Re-run key experiments
- Ensure improvements still hold
- Catch regressions early
- Validate on new Claude versions

### New Scenario Addition
When encountering new problems:
- Document the scenario
- Add to test library
- Run against current system
- Establish new baseline

### Improvement Tracking
Over time:
- Track metric trends
- Identify patterns in improvements
- Find areas still needing work
- Celebrate wins

## Integration with Development

### Before Implementing Feature:
1. Design experiment to validate need
2. Run baseline measurements
3. Identify target improvement
4. Implement feature
5. Re-run experiment
6. Validate improvement
7. Only merge if validated

### During Development:
- Run relevant scenarios continuously
- Catch regressions immediately
- Iterate based on results

### Before Release:
- Run full test suite
- Validate all key scenarios
- Ensure no regressions
- Document performance characteristics

## Experiment Categories

### 1. Prompt Engineering Experiments
- Test different prompt structures
- Evaluate instruction clarity
- Optimize for specific behaviors
- Measure compliance rates

### 2. Configuration Experiments
- Test `.cursorrules` variations
- Evaluate `claude.md` structures
- Optimize knowledge base formats
- Compare storage strategies

### 3. Architecture Experiments
- Test MCP vs alternatives
- Evaluate agent coordination strategies
- Compare learning approaches
- Validate cleanup strategies

### 4. Integration Experiments
- Test IDE plugin effectiveness
- Evaluate Firebase/Supabase helpers
- Validate PR workflow
- Measure end-to-end performance

## Failure Analysis

**Failures are valuable data.**

When experiments fail:
1. **Document thoroughly** - What went wrong?
2. **Analyze root cause** - Why did it fail?
3. **Extract learnings** - What did we learn?
4. **Adjust hypothesis** - How to revise?
5. **Try again** - Iterate

Failed experiments prevent wasted effort on ineffective optimizations.

## Success Criteria

### For Individual Experiments:
- ✅ Hypothesis validated or invalidated
- ✅ Measurable improvement (if validating improvement)
- ✅ No significant regressions
- ✅ Reproducible results
- ✅ Documented thoroughly

### For Overall System:
- ✅ 80%+ of key scenarios improved
- ✅ No scenario significantly regressed
- ✅ Consistent improvement over baseline
- ✅ All claims backed by data
- ✅ Comprehensive experiment coverage

## Documentation Requirements

### For Each Experiment:
- Hypothesis and rationale
- Experimental design
- Scenario details
- All configurations
- Complete results (success AND failure)
- Analysis and conclusions
- Decision and rationale

### Experiment Log:
Maintain `experiments/LOG.md` with:
- Chronological list of experiments
- Quick summary of each
- Links to detailed records
- Decision outcomes
- Learnings extracted

### Validation Report:
For each release, generate:
- Summary of validated optimizations
- Performance improvements demonstrated
- Scenarios tested
- Confidence levels
- Known limitations

## Tools to Build

### Priority 1: Experiment Runner
- CLI tool to run Claude Code with prompts
- Capture all output and metrics
- Save artifacts
- Repeatable and scriptable

### Priority 2: Validation Engine
- Check if generated code works
- Run tests automatically
- Measure quality metrics
- Detect common issues

### Priority 3: A/B Testing Framework
- Run multiple treatments
- Randomize execution
- Compare results
- Statistical analysis

### Priority 4: Results Database
- Store all experiment data
- Query and analyze
- Generate reports
- Track trends

### Priority 5: Scenario Library
- Curated test scenarios
- Easy to add new ones
- Parameterizable
- Covers common + edge cases

## Integration with Phase 1

**Phase 1 must include experimental validation infrastructure.**

Before building the actual optimized-ai system:
1. Build experiment runner
2. Establish baseline measurements
3. Define key scenarios
4. Create validation pipeline

Then, as we implement each feature:
- Design validation experiment
- Run control (without feature)
- Implement feature
- Run treatment (with feature)
- Validate improvement
- Only proceed if validated

## Example: Validating Spin Detection

### Hypothesis:
"Adding spin-detection prompts and monitoring will reduce instances of AI getting stuck by 70% and reduce wasted token usage by 40%."

### Experimental Design:

**Control**: Baseline `.cursorrules` without spin detection

**Treatment**: `.cursorrules` with:
- Spin detection prompts
- Self-monitoring instructions
- Alternative approach guidance

**Scenario**: Implement Firebase authentication (known to sometimes cause spinning)

**Metrics**:
- Number of repetitive edits to same file
- Token usage
- Time to completion
- Did AI get stuck (yes/no)
- Manual intervention needed (yes/no)

**Runs**: 10 control, 10 treatment

### Expected Outcome:
- Control: ~5 repetitive edits, ~50k tokens, ~30% spin rate
- Treatment: ~1 repetitive edit, ~30k tokens, ~5% spin rate

### Decision Criteria:
- If treatment ≥50% improvement: ADOPT
- If treatment 20-50% improvement: REFINE
- If treatment <20% improvement: REJECT

### Execute and Analyze:
Run the experiments, measure results, make data-driven decision.

---

## Bottom Line

**Every optimization in this system must be empirically validated.**

No theoretical improvements. No "should work" assumptions. Only proven, measured, reproducible improvements based on rigorous experimental validation.

This is the only way to ensure the system actually delivers value and doesn't become another pile of AI hype.

