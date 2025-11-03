# Meta Patterns Analysis - Claude Code Tresor

**Focus**: Reusable architectural patterns across the system

---

## Overview

This document extracts the meta-patterns that emerge from analyzing Tresor's skills, agents, and commands together. These are the fundamental design principles that make the system work effectively.

## The 10 Meta Patterns

### 1. Progressive Disclosure Architecture

**Pattern**: Complexity increases across tiers - automatic → manual → orchestrated

```
Tier 1: SKILLS (Automatic)
├─ Lightweight, always-on
├─ Limited tools (Read, Grep, Glob)
├─ Shared context
├─ < 1 second execution
└─ Detection without deep analysis

Tier 2: SUB-AGENTS (Manual)
├─ User-invoked (@agent)
├─ Full tools
├─ Separate context
├─ 30s - 5min execution
└─ Deep analysis with examples

Tier 3: COMMANDS (Orchestrated)
├─ Multi-step workflows (/command)
├─ Coordinates agents via Task tool
├─ Aggregates results
├─ 3-15min execution
└─ End-to-end automation
```

**Why It Works**:
- Users start simple (skills just work)
- Progress to manual invocation when needed (agents)
- Automate repetitive workflows (commands)
- Natural learning curve

**Application to Optimized AI**:
```
Skills:
- learner (auto-capture patterns)
- spin-detector (auto-flag stuck)
- cleaner (auto-suggest cleanup)

Agents:
- planner (manual task breakdown)
- implementer (manual code generation)
- tester (manual test creation)
- reviewer (manual quality check)

Commands:
- /implement (orchestrate full workflow)
- /review (multi-agent validation)
- /experiment (run A/B test)
```

---

### 2. Context Isolation Strategy

**Pattern**: Skills share context, agents have separate context

**Rationale**:
- **Shared Context (Skills)**: Efficient for quick checks, already in conversation
- **Separate Context (Agents)**: Focused analysis without pollution

**Trade-offs**:
| Shared Context | Separate Context |
|----------------|------------------|
| ✅ Fast activation | ⚠️ Context switch overhead |
| ✅ Low token usage | ⚠️ Higher token usage |
| ⚠️ Can pollute conversation | ✅ Clean focused analysis |
| ⚠️ Limited complexity | ✅ Can handle complex analysis |

**Application**: Our background skills share context, manual agents get separate context

---

### 3. Tool Restriction as Safety Mechanism

**Pattern**: Limit tools based on invocation method and risk

**Tool Access Hierarchy**:
```
Skills (Automatic → Safe Tools)
├─ Read: Yes ✅
├─ Write: Yes ✅ (controlled)
├─ Edit: Yes ✅ (controlled)
├─ Grep: Yes ✅
├─ Glob: Yes ✅
├─ Bash: No ❌ (can execute dangerous commands)
├─ Task: No ❌ (expensive, can spawn infinite agents)
└─ WebFetch: No ❌ (network calls, rate limits)

Agents (Manual → Full Tools)
├─ Read: Yes ✅
├─ Write: Yes ✅
├─ Edit: Yes ✅
├─ Grep: Yes ✅
├─ Glob: Yes ✅
├─ Bash: Yes ✅
├─ Task: Yes ✅
└─ WebFetch: Yes ✅
```

**Why It Matters**:
- Skills run automatically → must be safe
- Agents run on demand → user accepts risk
- Tool limits prevent runaway execution

**Application**: Define clear tool boundaries in our agent specs

---

### 4. Trigger-Based Activation

**Pattern**: Skills activate via keyword matching in description

**Mechanism**:
```yaml
---
description: Use when files modified, saved, or committed.
             Triggers on git diff, code edits, quality mentions.
---
```

**Trigger Keywords**:
- **Action Triggers**: "files modified", "saved", "committed"
- **Context Triggers**: "git diff", "code edits", "API endpoints"
- **Intent Triggers**: "quality", "security", "performance"

**How Claude Activates Skills**:
1. User action or message
2. Claude analyzes context
3. Matches context against skill descriptions
4. Activates relevant skills
5. Skills run in background, provide suggestions

**Application**: Our skills need carefully crafted descriptions with trigger keywords

---

### 5. Explicit vs Implicit Invocation

**Pattern**: Clear distinction between automatic (implicit) and manual (explicit)

**Decision Tree**:
```
Is it safe to run automatically?
├─ YES → Make it a Skill (implicit, always-on)
│   Examples: code review, secret scanning, spin detection
└─ NO → Make it an Agent (explicit, user-invoked)
    Examples: refactoring, architecture changes, learning extraction

Does it modify state significantly?
├─ YES → Must be explicit (user decides)
│   Examples: code generation, file editing, knowledge updates
└─ NO → Can be implicit (suggestions only)
    Examples: code quality suggestions, test suggestions
```

**Key Principle**: Automatic helpers suggest, manual agents implement

**Application**: Our `learner` skill suggests patterns, `@learner` agent does detailed extraction

---

### 6. Configuration Safety as First-Class Concern

**Pattern**: Special scrutiny for configuration changes

**Why Configuration is Dangerous**:
- Small numeric changes can cause production outages
- No compiler/linter catches config errors
- Impact only visible under load
- Rollback may be slow

**Risky Configuration Categories**:
```yaml
CONNECTION POOLS (🚨 CRITICAL)
- pool_size: Connection starvation or DB overload
- timeout: False failures or resource exhaustion

MEMORY LIMITS (🚨 CRITICAL)
- heap_size: OOM crashes
- buffer_size: Memory pressure
- cache_limit: Resource usage

TIMEOUTS (⚠️ HIGH)
- request_timeout: Cascading failures
- connect_timeout: False negatives
- read_timeout: UX degradation

RATE LIMITS (⚠️ HIGH)
- max_requests: Traffic handling capacity
- burst_size: Spike resilience
```

**Required Questions for Config Changes**:
1. "Why this specific value? What's the justification?"
2. "Has this been tested under production-like load?"
3. "How quickly can this be reverted?"
4. "What metrics indicate problems?"
5. "Have similar changes caused issues before?"

**Application**: Our `.cursorrules` changes must undergo this scrutiny

---

### 7. Evidence-Based Decision Making

**Pattern**: Require data, not assumptions, for optimization decisions

**Tresor Philosophy**: "Prove it's safe" for config changes
**Our Philosophy**: "Prove it's better" for optimizations

**Evidence Requirements**:
```
Before adopting optimization:
├─ Baseline metrics (control group)
├─ Treatment metrics (experimental group)
├─ Statistical significance (p < 0.05)
├─ Effect size (meaningful improvement)
├─ No quality degradation
└─ Reproducible results

Without evidence: Reject
With mixed evidence: Refine & retest
With clear evidence: Adopt & document
```

**Application**: Phase 0 experimental framework is mandatory - validates this pattern

---

### 8. Compositional Workflows

**Pattern**: Complex workflows emerge from composing simple agents

**Tresor Example** (`/review` command):
```
Simple Agents:
- @code-reviewer: Code quality analysis
- @security-auditor: Security scanning
- @performance-tuner: Performance analysis
- @architect: Architecture validation

Composed Workflow:
/review
├─ Task(subagent_type="code-reviewer") → Quality report
├─ Task(subagent_type="security-auditor") → Security report
├─ Task(subagent_type="performance-tuner") → Performance report
├─ Task(subagent_type="architect") → Architecture report
└─ Aggregate → Unified report with priorities
```

**Benefits**:
- Agents remain focused (single responsibility)
- Workflows are flexible (compose differently)
- Parallel execution (independent agents)
- Reusable components

**Application**: Our `/implement` command should compose planner + implementer + tester + reviewer

---

### 9. Structured Output for Parseable Results

**Pattern**: Consistent output format enables aggregation and automation

**Standard Output Structure**:
```markdown
## [Category] Results

### ✅ Positive Aspects
- [What's working well]

### ⚠️ Issues Found
#### 🚨 CRITICAL
- [Must fix immediately]

#### ⚠️ HIGH
- [Should fix soon]

#### 📋 MEDIUM
- [Consider fixing]

#### 💡 LOW
- [Nice to have]

### 🛠️ Recommended Actions
1. Action item 1
2. Action item 2

### 🎯 Next Steps
- Clear actionable steps
```

**Benefits**:
- Easy to parse (regex, markdown parsing)
- Severity prioritization built-in
- Aggregation across multiple agents
- Action-oriented

**Application**: All our agents should use this format

---

### 10. Documentation as Architecture

**Pattern**: For LLM-based systems, documentation IS configuration

**Observation**: Tresor agents have minimal JSON config, comprehensive README docs
**Insight**: LLMs learn behavior from examples, not just configuration

**Documentation Structure**:
```
agent/
└─ README.md
    ├─ Overview (purpose, capabilities)
    ├─ Examples (input → output patterns)
    ├─ Usage Patterns (when to use)
    ├─ Integration (with skills/commands)
    └─ Best Practices (how to get good results)
```

**Why Examples > Config**:
- LLMs learn from demonstrations
- Examples show expected behavior
- Edge cases documented through examples
- Natural language is the programming language

**Application**: Our agents need extensive examples, not just specifications

---

## Cross-Cutting Patterns

### A. Fail-Fast on Critical Issues

**Pattern**: Block workflows on critical severity issues

```
if critical_issues_found:
    generate_report()
    exit(1)  # Failure
    prevent_commit()

if only_medium_or_low:
    generate_report()
    exit(0)  # Success with warnings
    allow_commit()
```

**Application**: Self-evaluation should block PR creation if tests fail

---

### B. Learning from Failures

**Pattern**: Document what didn't work, not just what did

**Tresor Limitation**: No failure database (static utilities)
**Our Improvement**: `.ai-knowledge/failures.json`

**Failure Documentation**:
```json
{
  "failures": [
    {
      "scenario": "Reduced .cursorrules to 30 lines",
      "attempted": "2025-11-03",
      "hypothesis": "Minimal config improves performance",
      "result": "Quality degraded 40%, tokens same",
      "reason": "Essential context removed",
      "learning": "Need at least 80 lines for quality"
    }
  ]
}
```

**Application**: Every failed experiment goes into failures database

---

### C. Parallel Execution Where Possible

**Pattern**: Independent agents run simultaneously

**Tresor Implementation** (`/review`):
```python
# Sequential (SLOW)
result1 = invoke_code_reviewer()
result2 = invoke_security_auditor()  # Waits for result1
result3 = invoke_performance_tuner()  # Waits for result2

# Parallel (FAST)
results = Task.parallel([
    Task(subagent_type="code-reviewer"),
    Task(subagent_type="security-auditor"),
    Task(subagent_type="performance-tuner")
])
```

**Speedup**: 3x faster (3 agents in 5 min vs 15 min)

**Application**: Our workflows should identify independent steps and parallelize

---

### D. Sandboxing as Optional, Not Required

**Pattern**: Easy onboarding with optional security

**Tresor Decision**: Skills work WITHOUT sandboxing by default
**Rationale**:
- Reduces friction for new users
- Sandboxing available for security-sensitive environments
- Trust but verify

**Application**: Our framework should be easy to start, secure to scale

---

## Anti-Patterns Observed (Things to Avoid)

### ❌ Anti-Pattern 1: Static Utilities Don't Learn

**Problem**: Tresor utilities are fixed templates
**Impact**: Same mistakes repeated, no improvement over time
**Our Solution**: Dynamic learning via `.ai-knowledge/`

---

### ❌ Anti-Pattern 2: No Validation Framework

**Problem**: No way to measure if utilities actually help
**Impact**: Claims of improvement without evidence
**Our Solution**: Phase 0 experimental framework with A/B testing

---

### ❌ Anti-Pattern 3: Manual Pattern Updates

**Problem**: Maintainer must manually update prompts
**Impact**: Doesn't scale, knowledge silos
**Our Solution**: Automatic pattern extraction via `learner` skill

---

### ❌ Anti-Pattern 4: Exposed Internal Instructions

**Problem**: CLAUDE.md was in public repo initially
**Impact**: Internal instructions visible to users
**Our Solution**: Keep `.cursorrules` and internal configs private

---

## Synthesis: The Tresor Philosophy

### Core Principles

1. **Progressive Complexity**: Simple → Deep → Orchestrated
2. **Explicit > Implicit**: Manual invocation for risky operations
3. **Evidence > Assumptions**: Data-driven decisions for config
4. **Composition > Monoliths**: Small focused agents, composed workflows
5. **Documentation > Configuration**: Examples teach behavior

### Design Trade-offs

| They Prioritized | They Sacrificed | We Should Add |
|------------------|-----------------|---------------|
| ✅ Ease of use | ❌ Learning capability | ✅ .ai-knowledge/ |
| ✅ Static reliability | ❌ Dynamic improvement | ✅ Pattern learning |
| ✅ Manual expertise | ❌ Automatic optimization | ✅ Experimental validation |
| ✅ Clear separation | ❌ System integration | ✅ Holistic learning |

---

## Implementation Checklist for Optimized AI

Based on Tresor patterns, our system should have:

### ✅ Architecture
- [ ] 3-tier: Skills (automatic) → Agents (manual) → Commands (orchestrated)
- [ ] Progressive disclosure (simple → complex)
- [ ] Context isolation (shared for skills, separate for agents)
- [ ] Tool restriction (limited for skills, full for agents)

### ✅ Activation
- [ ] Trigger-based for skills (keyword matching)
- [ ] Explicit for agents (@agent-name)
- [ ] Explicit for commands (/command-name)
- [ ] Clear when/why each activates

### ✅ Configuration Safety
- [ ] Special scrutiny for .cursorrules changes
- [ ] Required questions for numeric changes
- [ ] Evidence requirements (A/B tests)
- [ ] Failure database check

### ✅ Workflows
- [ ] Compositional (small agents composed)
- [ ] Parallel execution (Task tool)
- [ ] Result aggregation (unified reports)
- [ ] Fail-fast on critical issues

### ✅ Learning (Our Addition)
- [ ] Automatic pattern capture (learner skill)
- [ ] Success documentation (patterns.json)
- [ ] Failure documentation (failures.json)
- [ ] Continuous improvement

### ✅ Validation (Our Addition)
- [ ] Experimental framework (Phase 0)
- [ ] A/B testing for optimizations
- [ ] Statistical significance
- [ ] Metrics-driven decisions

---

## Key Takeaways

### What Makes Tresor Successful

1. **Clear Tiers**: Skills/Agents/Commands are distinct and purposeful
2. **User Control**: Automatic detection, manual deep-dive
3. **Production Focus**: Real-world outage prevention (config safety)
4. **Comprehensive Docs**: Every component fully documented
5. **Zero Breaking Changes**: v1 → v2 was additive

### What We Must Add

1. **Learning**: Static → Dynamic (our .ai-knowledge/)
2. **Validation**: Claims → Evidence (our Phase 0)
3. **Optimization**: Manual → Automatic (our experimental framework)
4. **Integration**: Separate utilities → Holistic system
5. **Self-Improvement**: Fixed → Evolving

### The Combined Vision

```
Tresor's Strengths          Our Additions              = Optimized AI
├─ 3-tier architecture  +   Learning system            = Adaptive tiers
├─ Configuration safety +   A/B testing                = Validated optimizations
├─ Agent composition    +   Workflow learning          = Smarter workflows
├─ Documentation-first  +   Knowledge extraction       = Self-documenting
└─ Production focus     +   Continuous improvement     = Gets better over time
```

---

**Conclusion**: Tresor provides a proven architecture (Skills → Agents → Commands) and production-ready patterns (config safety, evidence-based). We should adopt their architecture wholesale, then layer our learning and validation systems on top. This gives us battle-tested foundations with self-improving capabilities they lack.

**Next Actions**:
1. Implement 3-tier architecture using Tresor patterns
2. Add `.ai-knowledge/` learning system (our innovation)
3. Build Phase 0 experimental validation (our innovation)
4. Combine proven patterns with our improvements

**See Also**:
- [skills/README.md](../skills/README.md) - Skill implementation patterns
- [agents/README.md](../agents/README.md) - Agent design patterns
- [commands/README.md](../commands/README.md) - Workflow orchestration patterns
- [../README.md](../README.md) - Complete analysis overview
