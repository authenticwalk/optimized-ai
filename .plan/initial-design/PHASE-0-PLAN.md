# Phase 0: Experimental Framework - Detailed Plan

**Duration**: 2 weeks  
**Status**: Ready to start  
**Priority**: CRITICAL - Must complete before any optimization work

## Why Phase 0 Must Come First

**You cannot optimize what you cannot measure.**

Every optimization we propose must be validated with data:
- Does it actually improve performance?
- By how much?
- Are there tradeoffs?
- Is it reproducible?

Without the experimental framework, we're building on guesses and theory.

## Goals

1. ✅ Run Claude Code from CLI with custom prompts
2. ✅ Capture all outputs (code, metrics, logs)
3. ✅ Validate generated code automatically
4. ✅ Run A/B tests comparing approaches
5. ✅ Establish baseline measurements
6. ✅ Store and query experiment results

## Deliverables

### 1. Experiment Runner CLI

**Location**: `experiments/runner/`

**Commands**:
```bash
# Create new experiment
experiment create \
  --id exp-001-minimal-cursorrules \
  --hypothesis "Minimal .cursorrules improves performance" \
  --scenario firebase-auth \
  --control baseline \
  --treatment minimal-rules

# Run experiment
experiment run \
  --experiment exp-001-minimal-cursorrules \
  --treatment control \
  --runs 5

# Validate results
experiment validate \
  --experiment exp-001-minimal-cursorrules \
  --run control-run-1

# Analyze results
experiment analyze \
  --experiment exp-001-minimal-cursorrules \
  --compare control,treatment \
  --metrics tokens,time,quality

# Generate report
experiment report \
  --experiment exp-001-minimal-cursorrules \
  --format markdown
```

**Implementation Tasks**:
- [ ] CLI scaffold with commander.js
- [ ] Experiment definition schema (JSON)
- [ ] Claude Code CLI integration
- [ ] Output capture and parsing
- [ ] Metric extraction (tokens, time, tool calls)
- [ ] Run management (parallel, sequential)
- [ ] Result storage (JSON files)

### 2. Scenario Library

**Location**: `experiments/scenarios/`

**Structure**:
```
scenarios/
├── basic/
│   ├── firebase-auth.scenario.json
│   ├── crud-operations.scenario.json
│   ├── simple-api.scenario.json
│   └── form-validation.scenario.json
├── complex/
│   ├── multi-file-feature.scenario.json
│   ├── refactoring-task.scenario.json
│   └── api-integration.scenario.json
└── edge-cases/
    ├── race-conditions.scenario.json
    ├── error-handling.scenario.json
    └── concurrent-updates.scenario.json
```

**Scenario Format**:
```json
{
  "id": "firebase-auth",
  "name": "Firebase Authentication Implementation",
  "description": "Implement user signup/login with Firebase Auth",
  "category": "basic",
  "difficulty": "medium",
  "prompt": "Implement user authentication using Firebase. Include signup with email/password and login functionality. Add proper error handling.",
  "requirements": [
    "User can sign up with email/password",
    "User can log in with email/password",
    "Errors are handled gracefully",
    "Firebase initialized correctly"
  ],
  "validation": {
    "files": ["auth.ts", "firebase.config.ts"],
    "testsRequired": true,
    "lintingRequired": true
  },
  "expectedDuration": 180,
  "expectedTokens": 30000,
  "knownIssues": [
    "Sometimes creates duplicate error handling",
    "May over-engineer auth flow"
  ]
}
```

**Initial Scenarios to Create**:
- [ ] firebase-auth (basic)
- [ ] crud-operations (basic)
- [ ] simple-api (basic)
- [ ] form-validation (basic)
- [ ] multi-file-feature (complex)
- [ ] refactoring-task (complex)
- [ ] error-handling (edge-case)
- [ ] race-conditions (edge-case)

### 3. Validation Engine

**Location**: `experiments/validation/`

**Capabilities**:
```typescript
interface Validator {
  // Code validation
  checkCompilation(files: string[]): ValidationResult;
  runTests(files: string[]): TestResult;
  runLinter(files: string[]): LintResult;
  
  // Requirement validation
  checkRequirements(scenario: Scenario, output: Output): RequirementResult;
  
  // Quality metrics
  measureComplexity(files: string[]): ComplexityResult;
  detectDuplication(files: string[]): DuplicationResult;
  
  // Behavioral validation
  detectSpin(activity: Activity[]): SpinDetection;
  detectRepetition(edits: Edit[]): RepetitionResult;
}
```

**Implementation Tasks**:
- [ ] TypeScript compilation checker
- [ ] Test runner integration (Vitest)
- [ ] Linter integration (ESLint)
- [ ] Requirements checker (keyword/pattern matching)
- [ ] Code quality analyzer (complexity, duplication)
- [ ] Spin detector (repetition patterns)
- [ ] Aggregated validation results

### 4. Baseline Measurements

**Goal**: Establish performance baselines with NO optimizations

**Approach**:
```bash
# Create baseline experiment set
experiment batch-create \
  --set baseline-2025-11 \
  --scenarios firebase-auth,crud-operations,simple-api,form-validation \
  --control pure-claude \
  --runs 10

# Run all baseline experiments
experiment batch-run --set baseline-2025-11

# Generate baseline report
experiment batch-report --set baseline-2025-11
```

**Metrics to Capture**:
- Time to completion (seconds)
- Token usage (input + output)
- Tool call count
- File edit count
- Repetitive edit count
- Test pass rate
- Linter pass rate
- Requirements met rate
- Spin incidents
- Manual intervention needed

**Target**: 10 runs each for 8 key scenarios = 80 baseline runs

**Tasks**:
- [ ] Define 8 key scenarios
- [ ] Run 10 times each with pure Claude (no optimizations)
- [ ] Capture all metrics
- [ ] Calculate averages and std deviation
- [ ] Document baseline performance
- [ ] Identify common failure modes

### 5. A/B Testing Framework

**Location**: `experiments/ab-testing/`

**Capabilities**:
```typescript
interface ABTest {
  // Test execution
  runABTest(experiment: Experiment): ABTestResult;
  
  // Statistical analysis
  calculateMean(values: number[]): number;
  calculateStdDev(values: number[]): number;
  calculateConfidenceInterval(values: number[], confidence: number): Interval;
  performTTest(control: number[], treatment: number[]): TTestResult;
  
  // Comparison
  compareMetrics(control: Metrics[], treatment: Metrics[]): Comparison;
  calculateImprovement(control: number, treatment: number): Improvement;
  
  // Reporting
  generateComparisonReport(result: ABTestResult): Report;
}
```

**Statistical Analysis**:
- Mean and standard deviation
- Confidence intervals (95%)
- T-test for significance
- Effect size calculation
- Visual comparison (charts/graphs optional for now)

**Tasks**:
- [ ] AB test orchestrator
- [ ] Statistical utilities
- [ ] Comparison engine
- [ ] Significance testing
- [ ] Report generator
- [ ] Result visualization (optional)

### 6. Results Database

**Location**: `experiments/results/`

**Structure**:
```
results/
├── experiments/
│   ├── exp-001-minimal-cursorrules/
│   │   ├── definition.json
│   │   ├── control/
│   │   │   ├── run-1.json
│   │   │   ├── run-2.json
│   │   │   └── ...
│   │   ├── treatment/
│   │   │   ├── run-1.json
│   │   │   ├── run-2.json
│   │   │   └── ...
│   │   ├── analysis.json
│   │   └── report.md
│   └── ...
├── baselines/
│   └── baseline-2025-11/
│       └── ...
└── index.json
```

**Query Capabilities**:
```bash
# Query experiments
experiment query --hypothesis "spin detection" --status completed

# Get metrics for experiment
experiment metrics --id exp-001 --metric tokens --treatment all

# Compare across experiments
experiment compare --ids exp-001,exp-002,exp-003 --metric improvement

# Trend analysis
experiment trends --metric tokens --over-time
```

**Tasks**:
- [ ] File-based storage structure
- [ ] Index for fast queries
- [ ] Query interface
- [ ] Aggregation utilities
- [ ] Trend analysis
- [ ] Export to CSV/JSON

## Technical Stack

### Core Technologies:
- **TypeScript** - Type-safe implementation
- **Node.js** - Runtime
- **Commander.js** - CLI framework
- **Vitest** - Testing (also used for validation)
- **ESLint** - Linting (also used for validation)
- **Zod** - Schema validation

### File Formats:
- **JSON** - Experiment definitions, results storage
- **Markdown** - Reports and documentation

### Build:
- **tsup** - Simple TypeScript bundler
- **pnpm** - Package manager

## Implementation Phases

### Week 1: Core Infrastructure

**Days 1-2**: Experiment Runner CLI
- CLI scaffold
- Experiment definition
- Basic commands (create, run)
- Claude Code integration

**Days 3-4**: Scenario Library
- Scenario schema
- 8 initial scenarios
- Scenario loader
- Validation of scenario format

**Days 5-7**: Validation Engine
- Compilation checker
- Test runner
- Linter integration
- Basic requirement validation

### Week 2: Analysis & Baseline

**Days 8-9**: A/B Testing Framework
- Statistical utilities
- AB test orchestrator
- Comparison engine
- Report generator

**Days 10-12**: Baseline Measurements
- Run 80 baseline experiments
- Capture all metrics
- Analyze results
- Document baselines

**Days 13-14**: Results Database & Polish
- Results storage structure
- Query interface
- Final testing
- Documentation

## Success Criteria

### Week 1 Success:
- ✅ Can run Claude Code from CLI with custom prompt
- ✅ Can capture output and parse metrics
- ✅ Have 8 defined scenarios
- ✅ Can validate generated code (compile, test, lint)

### Week 2 Success:
- ✅ Can run A/B tests with statistical analysis
- ✅ Have baseline measurements for 8 scenarios
- ✅ Can compare control vs treatment
- ✅ Can generate experiment reports
- ✅ Results stored and queryable

### Overall Success:
- ✅ Complete experimental framework operational
- ✅ Baseline data establishes comparison point
- ✅ Can run rigorous experiments for Phase 1+
- ✅ All tools documented and tested

## Example: First Real Experiment

**After Phase 0 complete, we'll run our first real experiment:**

```bash
# Hypothesis: Minimal .cursorrules (50 lines) performs 
# as well as monolithic (500 lines) with 40% fewer tokens

experiment create \
  --id exp-001-minimal-cursorrules \
  --hypothesis "Minimal .cursorrules reduces tokens 40% without quality loss" \
  --scenario firebase-auth

# Control: 500-line .cursorrules
experiment run --experiment exp-001 --treatment control --runs 10

# Treatment: 50-line .cursorrules  
experiment run --experiment exp-001 --treatment minimal --runs 10

# Analyze
experiment analyze --experiment exp-001

# Result will tell us:
# - Did tokens reduce by 40%? ✓ or ✗
# - Did quality stay same? ✓ or ✗
# - Was difference statistically significant? ✓ or ✗
# - Should we adopt minimal approach? ADOPT / REJECT / REFINE
```

## Risk Mitigation

### Risk: Claude Code CLI doesn't work as expected
**Mitigation**: Test CLI integration in first 2 days, pivot to API if needed

### Risk: Metrics hard to capture automatically
**Mitigation**: Start with manual capture, automate incrementally

### Risk: Baseline takes too long
**Mitigation**: Reduce scenarios or runs, but maintain statistical validity

### Risk: A/B testing too complex
**Mitigation**: Start with simple comparisons, add sophistication later

## Dependencies

### External:
- Claude Code CLI (or API access)
- Node.js 20+
- TypeScript
- Test framework (Vitest)
- Linter (ESLint)

### Internal:
- None - Phase 0 is the foundation

## Deliverable Checklist

Before declaring Phase 0 complete:

- [ ] Experiment runner CLI works
- [ ] Can create experiments
- [ ] Can run experiments
- [ ] Can validate results
- [ ] Can analyze results
- [ ] Can generate reports
- [ ] Have 8 defined scenarios
- [ ] Can validate generated code
- [ ] Can run A/B tests
- [ ] Have statistical analysis
- [ ] Have baseline measurements (8 scenarios × 10 runs)
- [ ] Results stored in structured format
- [ ] Can query results
- [ ] Documentation complete
- [ ] Examples provided
- [ ] Ready to start Phase 1

## Next Phase Preview

**Phase 1: Minimal Core**

Once Phase 0 is complete, we'll use it to:
1. Test different .cursorrules sizes (10, 20, 30, 50 lines)
2. Find optimal minimal configuration
3. Validate each rule's contribution
4. Build the minimal core based on data

**Everything after Phase 0 will be data-driven.**

---

**Ready to start building the experiment runner?** 🚀

