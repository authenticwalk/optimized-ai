# Commands Analysis - Claude Code Tresor

**Focus**: Multi-agent orchestration workflows

---

## Overview

Commands represent the top tier of Tresor's architecture - orchestrated workflows that coordinate skills and sub-agents to complete complex, end-to-end processes. Commands are explicitly triggered with `/command-name` syntax and use the Task tool to invoke sub-agents.

## Command Architecture

### Core Structure

```
command/
└─ README.md          # Complete workflow documentation
```

**Key Pattern**: Commands are workflows, not agents. They orchestrate existing agents rather than doing analysis themselves.

## 4 Core Commands Analyzed

### /scaffold
**Purpose**: Generate project structures, components, boilerplate
**Orchestrates**: File generation + test creation + documentation stubs
**Use Case**: Starting new components/projects quickly

**Pattern**: Template-based generation
```bash
/scaffold react-component UserProfile --hooks --tests --typescript

Workflow:
1. Read template for React component
2. Generate component file with hooks
3. Create test file with basic structure
4. Add TypeScript interfaces
5. Create documentation stub
```

**Applicability for Us**: Low priority - we're not building project scaffolding

### /review (MOST RELEVANT)
**Purpose**: Comprehensive multi-perspective code review
**Orchestrates**: @code-reviewer + @security-auditor + @performance-tuner + @architect
**Use Case**: PR reviews, pre-commit validation

**Detailed Workflow**:
```bash
/review --scope staged --checks all

Step 1: Identify scope
├─ Read git diff --staged
├─ Parse modified files
└─ Categorize changes (code vs config vs tests)

Step 2: Parallel agent invocation
├─ Task(subagent_type="code-reviewer", prompt="Analyze [files] for quality")
├─ Task(subagent_type="security-auditor", prompt="Scan [files] for vulnerabilities")
├─ Task(subagent_type="performance-tuner", prompt="Check [files] for bottlenecks")
└─ Task(subagent_type="architect", prompt="Validate [files] architecture")

Step 3: Configuration safety (SPECIAL HANDLING)
├─ Detect config files (.yml, .json, .env)
├─ Flag numeric changes
├─ Require justification for limits
└─ Check against known risky patterns

Step 4: Aggregate results
├─ Collect all agent outputs
├─ Deduplicate findings
├─ Prioritize by severity (CRITICAL → LOW)
└─ Add line numbers and context

Step 5: Generate report
└─ Unified markdown report with:
    - Critical issues (must fix)
    - Recommendations (should fix)
    - Suggestions (nice to have)
    - Positive feedback
```

**Key Insight**: Configuration safety is FIRST-CLASS concern
- Special scrutiny for config files
- "Prove it's safe" mentality for numeric changes
- Requires load testing evidence
- Checks for production outage patterns

**Applicability for Us**: VERY HIGH - this should be our self-evaluation loop pattern

### /test-gen
**Purpose**: Generate comprehensive test suites
**Orchestrates**: test-generator skill + @test-engineer agent
**Use Case**: Automated test creation

**Workflow**:
```bash
/test-gen --file utils.js --framework jest --coverage 90

Step 1: Analyze target file
├─ Read file content
├─ Identify functions/classes
└─ Detect current test coverage

Step 2: Skill quick scan
└─ test-generator skill identifies missing tests

Step 3: Agent comprehensive generation
└─ Task(subagent_type="test-engineer", prompt="Create suite for [file]")

Step 4: Generate tests
├─ Unit tests for all functions
├─ Edge cases and error handling
├─ Integration tests if applicable
└─ Achieve target coverage (90%)
```

**Applicability for Us**: Medium - useful for testing our experimental framework

### /docs-gen
**Purpose**: Generate documentation from code
**Orchestrates**: api-documenter skill + @docs-writer agent
**Use Case**: Automated documentation creation

**Applicability for Us**: Low - not primary concern

## Critical Command Patterns

### Pattern 1: Task Tool for Agent Invocation

**Tresor Implementation**:
```python
# Pseudocode from /review command
results = []

# Invoke agents in parallel
code_review = Task(
    subagent_type="code-reviewer",
    prompt=f"Analyze {files} for code quality and best practices"
)

security_review = Task(
    subagent_type="security-auditor",
    prompt=f"Scan {files} for security vulnerabilities"
)

# Wait for results
results.append(code_review.result)
results.append(security_review.result)

# Aggregate
report = aggregate_results(results)
```

**Key Benefit**: Parallel execution - multiple agents run simultaneously
**Mechanism**: Task tool creates separate contexts for each agent

**Our Application**: Our `/implement` command should use Task tool similarly

### Pattern 2: Workflow State Management

**Observation**: Commands maintain workflow state through steps
**Implementation**: Sequential steps with conditional branching

```
Step 1: Gather input
Step 2: Invoke agents (parallel if independent)
Step 3: Process results
Step 4: If issues: Branch to fix flow
Step 5: Generate output
```

**Our Application**: Our workflows should have clear state management

### Pattern 3: Configuration-Specific Logic

**Key Insight**: `/review` command has SPECIAL handling for config files

**Risky Configuration Patterns**:
```yaml
# Connection Pool Settings - HIGH RISK
pool_size: 50 → 25  # Can cause connection starvation
pool_size: 10 → 100 # Can overload database
timeout: 5000 → 1000 # Can cause false failures

# Memory and Resource Limits - CRITICAL
heap_size: "2g" → "4g"  # Can cause OOM
buffer_size: 8192 → 16384
cache_limit: 1000 → 5000
```

**Questions Command Asks**:
1. "Has this been tested with production-level load?"
2. "How quickly can this be reverted if issues occur?"
3. "What metrics will indicate if this change causes problems?"
4. "How does this interact with other system limits?"
5. "Have similar changes caused issues before?"

**Our Application**: Critical for validating `.cursorrules` changes
- Flag any change to token limits, timeout values
- Require A/B test results before adopting
- Check against `.ai-knowledge/failures.json` for similar past issues

### Pattern 4: Result Aggregation

**Pattern**: Combine multiple agent outputs into unified report

**Aggregation Steps**:
1. Collect all agent findings
2. Deduplicate (same issue found by multiple agents)
3. Prioritize by severity (CRITICAL → HIGH → MEDIUM → LOW)
4. Group by category (security, performance, style)
5. Add context (line numbers, file paths)
6. Generate actionable next steps

**Output Structure**:
```markdown
# Code Review Report

## 🚨 Critical Issues (Must fix before deployment)
1. [Issue with context]
   - Location: file.js:42
   - Found by: @security-auditor, @code-reviewer
   - Fix: [Specific recommendation]

## ⚠️ Recommendations (Should fix)
[Grouped by category]

## 💡 Suggestions (Nice to have)
[Lower priority improvements]

## ✅ Positive Feedback
[What's done well]
```

**Our Application**: Self-evaluation loop should produce similar report

### Pattern 5: Fail-Fast on Critical Issues

**Observation**: `/review` can block on critical issues
**Mechanism**: Severity threshold determines pass/fail

```bash
/review --severity critical

If CRITICAL issues found:
  - Report generated
  - Exit code 1 (failure)
  - Prevents commit/PR

If only HIGH/MEDIUM/LOW:
  - Report generated
  - Exit code 0 (success)
  - Commit/PR proceeds with warnings
```

**Our Application**: Self-evaluation should block PR creation if tests fail or linter errors

## Workflow Patterns for Our Project

### Workflow 1: Task Implementation

```bash
/implement "Add user authentication with Firebase"

Step 1: Planning
└─ Task(subagent_type="planner", prompt="Break down: [task]")
   Output: .plan/approach.md

Step 2: Check Knowledge Base
└─ Read .ai-knowledge/patterns.json for similar tasks
   If found: Use proven patterns

Step 3: Implementation
└─ Task(subagent_type="implementer", prompt="Implement following plan + patterns")
   Output: Code changes

Step 4: Testing
└─ Task(subagent_type="tester", prompt="Create tests for implementation")
   Output: Test files

Step 5: Self-Evaluation
└─ Task(subagent_type="reviewer", prompt="Validate against requirements")
   - Run tests (must pass)
   - Check linter (must pass)
   - Validate against patterns
   - Check preferences
   If fails: Loop back to Step 3

Step 6: Learning
└─ Task(subagent_type="learner", prompt="Extract patterns from successful implementation")
   Output: Updated .ai-knowledge/

Step 7: PR Creation
└─ Create branch, commit, push, open PR
```

**Key Features**:
- Uses Task tool for all agent invocations
- Loops on failure (Step 5 → Step 3)
- Captures learnings (Step 6)
- Only creates PR after passing evaluation

### Workflow 2: Optimization Experiment

```bash
/experiment "Test minimal .cursorrules (80 lines vs 500 lines)"

Step 1: Validate Phase 0 Ready
└─ Check experimental framework exists
   If not: Error with setup instructions

Step 2: Create Experiment Definition
└─ Generate experiment config:
   - Hypothesis
   - Control (current .cursorrules)
   - Treatment (minimal .cursorrules)
   - Metrics (tokens, time, quality)
   - Scenarios (firebase-auth, crud-operations)

Step 3: Run Baseline
└─ Task(subagent_type="experiment-runner", prompt="Run control group")
   Output: Baseline metrics

Step 4: Run Treatment
└─ Task(subagent_type="experiment-runner", prompt="Run treatment group")
   Output: Treatment metrics

Step 5: Statistical Analysis
└─ Compare results with t-test, confidence intervals
   Output: Is improvement significant?

Step 6: Decision
If improvement AND significant:
  └─ Adopt treatment
  └─ Update .ai-knowledge/patterns.json
Else:
  └─ Reject treatment
  └─ Document in .ai-knowledge/failures.json
```

**Key Features**:
- Requires Phase 0 framework
- Statistical validation (not guesses)
- Documents both successes and failures
- Evidence-based decisions

### Workflow 3: Spin Recovery

```bash
/unstick

Step 1: Detect Current State
└─ Read recent activity
   - File edit history
   - Error logs
   - Time since progress

Step 2: Check Failures Database
└─ Read .ai-knowledge/failures.json
   - Find similar stuck patterns
   - Retrieve what worked before

Step 3: Generate Alternatives
└─ Task(subagent_type="debugger", prompt="Analyze stuck pattern, suggest alternatives")
   Output: 2-3 different approaches

Step 4: Select Approach
└─ Rank alternatives by:
   - Past success rate
   - Estimated effort
   - Risk level

Step 5: Switch Context
└─ Update .plan/approach.md with new strategy
   Log spin incident for future learning

Step 6: Resume Implementation
└─ Task(subagent_type="implementer", prompt="Try alternative approach [X]")
```

**Key Features**:
- Learns from past failures
- Systematically tries alternatives
- Logs incidents for improvement

## Configuration Safety Implementation

### For `.cursorrules` Validation

Based on `/review` command's configuration safety pattern:

```python
# Pseudocode for .cursorrules validation

def validate_cursorrules_change(old_content, new_content):
    """Apply Tresor's config safety pattern to .cursorrules"""

    # Detect numeric changes
    numeric_changes = detect_numeric_changes(old_content, new_content)

    for change in numeric_changes:
        # Flag and require justification
        issues.append({
            "severity": "CRITICAL",
            "type": "config-change",
            "description": f"Numeric value changed: {change.name}",
            "old_value": change.old,
            "new_value": change.new,
            "questions": [
                "Why this specific value?",
                "Has this been A/B tested?",
                "What's the impact on token usage?",
                "What happens if limit is hit?"
            ],
            "required_evidence": "A/B test results showing no degradation"
        })

    # Check against known risky patterns
    risky_patterns = load_risky_patterns()
    for pattern in risky_patterns:
        if pattern.matches(new_content):
            issues.append({
                "severity": "HIGH",
                "type": "risky-pattern",
                "description": pattern.warning,
                "historical_incidents": find_past_issues(pattern)
            })

    return issues
```

**Application**: Our self-evaluation loop must include this validation

## Command Implementation Checklist

For creating our own commands based on Tresor patterns:

### ✅ Structure
- [ ] README.md with complete workflow documentation
- [ ] Clear purpose and use cases
- [ ] Example invocations with flags/options
- [ ] Integration examples with other commands

### ✅ Workflow Design
- [ ] Sequential steps with clear state
- [ ] Parallel execution where possible (Task tool)
- [ ] Conditional branching for failures
- [ ] Result aggregation from multiple agents
- [ ] Fail-fast on critical issues

### ✅ Agent Coordination
- [ ] Use Task tool for all agent invocations
- [ ] Pass clear prompts with context
- [ ] Handle agent failures gracefully
- [ ] Aggregate results intelligently

### ✅ Output
- [ ] Structured report format
- [ ] Severity levels (CRITICAL, HIGH, MEDIUM, LOW)
- [ ] Actionable next steps
- [ ] Context (line numbers, file paths)

## Key Differences from Agents

| Aspect | Agents | Commands |
|--------|--------|----------|
| **Purpose** | Deep analysis | Workflow orchestration |
| **Invocation** | `@agent-name` | `/command-name` |
| **Execution** | Single-focused | Multi-step process |
| **Coordination** | N/A (single entity) | Coordinates multiple agents |
| **Output** | Analysis report | Aggregated workflow result |

## Key Takeaways

1. **Commands = Orchestrators**: Coordinate agents, don't do analysis themselves
2. **Task Tool = Coordination**: Use Task tool to invoke agents in parallel
3. **Configuration Safety = First-Class**: Special handling for config changes
4. **Result Aggregation = Value**: Combine multiple perspectives into unified report
5. **Fail-Fast = Quality**: Block on critical issues, warn on minor ones

## Recommendations for Our Implementation

### ✅ High Priority Commands

1. **/implement** - Full task execution workflow (planner → implementer → tester → reviewer → learner)
2. **/review** - Self-evaluation before PR (based on Tresor's /review)
3. **/experiment** - Run A/B test in Phase 0 framework

### ⚠️ Medium Priority Commands

4. **/unstick** - Spin recovery workflow
5. **/sync-knowledge** - Merge global learnings into project

### ❌ Low Priority Commands

6. **/scaffold** - Project generation (not our focus)
7. **/docs-gen** - Documentation generation (not core goal)

---

**Next Steps**:
1. Design `/review` command for self-evaluation using Tresor pattern
2. Implement Task tool coordination for parallel agent execution
3. Add configuration safety validation for `.cursorrules`
4. Create result aggregation logic with severity prioritization

**See Also**:
- [skills/README.md](../skills/README.md) - Automatic background helpers
- [agents/README.md](../agents/README.md) - Manual expert sub-agents
- [patterns/README.md](../patterns/README.md) - Reusable meta-patterns
