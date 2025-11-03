# Experiments Log

This file contains a chronological record of all experiments conducted to validate optimizations in the optimized-ai system.

**Instructions**: Append each new experiment using the template from [3-validate.md](../.plan/initial-design/principles/3-validate.md). Never delete experiments - failures are valuable data.

---

## Table of Contents

<!-- Update this as experiments are added -->

- [Experiment Index](#experiment-index)
- [Experiments](#experiments)

---

## Experiment Index

| ID | Name | Date | Status | Decision | Summary |
|----|------|------|--------|----------|---------|
| - | - | - | - | - | *No experiments yet* |

---

## Experiments

*Experiments will be appended below this line using the template*

---

<!-- 
TEMPLATE - Copy this for each new experiment:

## Experiment: [ID] - [Short Name]

**Date**: YYYY-MM-DD  
**Status**: 🔬 IN PROGRESS | ✅ ADOPTED | ❌ REJECTED | 🔄 REFINING

### Hypothesis

[Specific, testable prediction. E.g., "Adding spin-detection prompts reduces repetitive edits by 50% and token usage by 40%"]

### Rationale

[Why we think this will work. What problem are we solving?]

### Experimental Design

**Scenario**: [Which test scenario - e.g., "firebase-auth-implementation"]

**Control Configuration**:
- Description: [baseline setup]
- Files: [path to config files]

**Treatment Configuration**:
- Description: [optimized setup]
- Files: [path to config files]
- Changes: [what's different from control]

**Metrics to Measure**:
- [ ] Time to completion
- [ ] Token usage
- [ ] Tool call count
- [ ] File edit count
- [ ] Repetitive edits
- [ ] Tests pass
- [ ] Linter clean
- [ ] Spin detected
- [ ] Manual intervention needed
- [ ] Code quality score

**Runs Planned**: [number] control, [number] treatment

### Results

#### Control Results (Baseline)

| Run | Duration | Tokens | Tool Calls | File Edits | Repetitive | Tests | Linter | Spin | Quality |
|-----|----------|--------|------------|------------|------------|-------|--------|------|---------|
| 1   |          |        |            |            |            |       |        |      |         |
| 2   |          |        |            |            |            |       |        |      |         |
| ... |          |        |            |            |            |       |        |      |         |

**Average**:
- Duration: [avg ± std]
- Tokens: [avg ± std]
- Tool Calls: [avg ± std]
- File Edits: [avg ± std]
- Repetitive Edits: [avg ± std]
- Spin Rate: [%]
- Tests Pass Rate: [%]
- Linter Clean Rate: [%]
- Quality Score: [avg ± std]

**Artifacts**: `experiments/results/exp-XXX/control/`

#### Treatment Results (Optimized)

| Run | Duration | Tokens | Tool Calls | File Edits | Repetitive | Tests | Linter | Spin | Quality |
|-----|----------|--------|------------|------------|------------|-------|--------|------|---------|
| 1   |          |        |            |            |            |       |        |      |         |
| 2   |          |        |            |            |            |       |        |      |         |
| ... |          |        |            |            |            |       |        |      |         |

**Average**:
- Duration: [avg ± std]
- Tokens: [avg ± std]
- Tool Calls: [avg ± std]
- File Edits: [avg ± std]
- Repetitive Edits: [avg ± std]
- Spin Rate: [%]
- Tests Pass Rate: [%]
- Linter Clean Rate: [%]
- Quality Score: [avg ± std]

**Artifacts**: `experiments/results/exp-XXX/treatment/`

### Analysis

**Comparison**:
- Duration: [% change, significance]
- Tokens: [% change, significance]
- Repetitive Edits: [% change, significance]
- Spin Rate: [% change]
- Quality: [% change]

**Statistical Significance**:
- Test used: [t-test, Mann-Whitney U, etc.]
- P-value: [value]
- Confidence Level: [%]
- Effect Size: [Cohen's d or similar]

**Qualitative Observations**:
- [Code readability improvements?]
- [Architecture quality?]
- [Unexpected benefits/drawbacks?]
- [Edge cases discovered?]

**Confounding Factors**:
- [Any variables that might have influenced results?]

### Conclusion

[Clear statement: Was hypothesis validated? Did treatment improve over control? By how much?]

**Decision**: ✅ ADOPT | ❌ REJECTED | 🔄 REFINE

**Rationale**: [Why this decision?]

**Next Steps** (if REFINE):
- [What to change for next iteration]
- [New hypothesis to test]

### Learnings

**What Worked**:
- [Key insights about what made this successful/unsuccessful]

**What Didn't**:
- [Unexpected failures or issues]

**Surprises**:
- [Anything unexpected?]

**Applied To**:
- [If adopted, where/how was it integrated?]
- [Which files were updated?]

---

END TEMPLATE -->

