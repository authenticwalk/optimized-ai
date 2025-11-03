# Agents Analysis - Claude Code Tresor

**Focus**: Manual expert sub-agents for deep analysis

---

## Overview

Sub-agents in Tresor represent expert specialists that provide comprehensive analysis when explicitly invoked. Unlike skills (automatic), agents require manual invocation with `@agent-name` syntax and have full tool access for deep work.

## Agent Architecture

### Core Structure

```
agent/
├─ README.md          # Comprehensive guide with examples
└─ (No AGENT.md in repo - using Claude Code's native sub-agent format)
```

**Key Insight**: Tresor relies on Claude Code's native sub-agent functionality rather than creating custom configuration files. This is different from skills which have explicit SKILL.md files.

## 8 Core Agents Analyzed

### Code Quality Domain

#### @code-reviewer
**Expertise**: Code quality, security, performance, best practices
**When to Use**: PR reviews, refactoring validation, architecture decisions
**Tool Access**: Read, Edit, Bash, Grep, Glob, Task
**Key Feature**: Multi-perspective analysis (quality + security + performance)

**Pattern**: Complementary to code-reviewer skill
- Skill: Quick scan while coding ("⚠️ Missing error handling")
- Agent: Deep analysis with examples, alternatives, architectural impact

**Example Output Structure**:
```markdown
## Code Review Results

### ✅ Positive Aspects
- [What's done well]

### ⚠️ Issues Found
#### 🔴 Critical Issues
- [Must fix before deployment]

#### 🟡 Improvements Suggested
- [Should fix for better quality]

#### 🔵 Performance Optimizations
- [Optional improvements]

### 📊 Code Quality Score: X/10

### 🛠️ Recommended Refactor
[Complete working example]

### 🎯 Next Steps
1. Action item 1
2. Action item 2
```

**Applicability for Us**: Our `reviewer` agent should follow this structured output format

#### @test-engineer
**Expertise**: Test strategies, test creation, QA practices
**When to Use**: Comprehensive test suite creation, coverage analysis
**Tool Access**: Read, Write, Edit, Bash, Grep, Glob, Task
**Key Feature**: Framework-aware (Jest, Vitest, pytest, JUnit)

**Pattern**: Coordinates with test-generator skill
- Skill: "Missing tests for function X" (detection)
- Agent: Creates comprehensive suite with edge cases (implementation)

**Applicability for Us**: Our `tester` agent should have framework detection capabilities

### System Design Domain

#### @architect
**Expertise**: System design, architecture decisions, technology evaluation
**When to Use**: Architecture reviews, design decisions, scalability planning
**Tool Access**: Full (can use WebFetch for research)
**Key Feature**: Long-term maintainability focus

**Pattern**: Strategic vs tactical
- Other agents: Tactical improvements to existing code
- Architect: Strategic architectural decisions

**Applicability for Us**: Our system might not need dedicated architect agent - can be part of planner

### Problem Solving Domain

#### @debugger
**Expertise**: Root cause analysis, troubleshooting, stack trace analysis
**When to Use**: Production issues, complex bugs, performance problems
**Tool Access**: Full (Bash for reproduction, Task for investigation)
**Key Feature**: Systematic debugging methodology

**Pattern**: Problem-solving workflow
1. Analyze error/stack trace
2. Reproduce issue
3. Identify root cause
4. Propose fix with validation

**Applicability for Us**: Useful for debugging our own framework issues

### Security Domain

#### @security-auditor
**Expertise**: Security assessment, OWASP compliance, vulnerability analysis
**When to Use**: Security audits, penetration test prep, compliance validation
**Tool Access**: Full (can run security scanning tools via Bash)
**Key Feature**: Threat modeling + remediation

**Pattern**: Defense in depth
- Checks multiple layers: input validation, authentication, authorization, data protection
- Provides severity ratings and remediation priorities

**Applicability for Us**: Less critical for our framework (we're not building user-facing apps)

### Performance Domain

#### @performance-tuner
**Expertise**: Performance optimization, profiling, bottleneck analysis
**When to Use**: Slow queries, memory leaks, optimization opportunities
**Tool Access**: Full (can run profiling tools)
**Key Feature**: Data-driven recommendations

**Pattern**: Measure → Analyze → Optimize → Validate
- Requires profiling data
- Quantifies improvements
- Validates optimizations

**Applicability for Us**: Critical for Phase 0 - validating our optimizations actually improve performance

### Code Structure Domain

#### @refactor-expert
**Expertise**: Code restructuring, design patterns, technical debt
**When to Use**: Legacy code modernization, technical debt reduction
**Tool Access**: Full (Edit for refactoring)
**Key Feature**: Pattern-based refactoring

**Applicability for Us**: Less relevant (our code is new, not legacy)

#### @docs-writer
**Expertise**: Technical documentation, user guides, API docs
**When to Use**: Documentation creation, tutorials, troubleshooting guides
**Tool Access**: Full (can generate diagrams, examples)
**Key Feature**: Audience-aware documentation

**Pattern**: Multi-format output
- API docs (OpenAPI)
- User guides (Markdown)
- Architecture diagrams (Mermaid)
- Interactive docs (Docusaurus)

**Applicability for Us**: Could help document our experimental results

## Key Agent Patterns

### Pattern 1: Separate Context for Focus

**Tresor Implementation**: Agents have separate context from main conversation
**Benefit**: Enables focused analysis without polluting main chat
**Cost**: Context switch overhead

**Our Application**:
- Main conversation: User task
- Agent context: Deep analysis of specific issue
- Return to main: Aggregated results

### Pattern 2: Full Tool Access for Comprehensive Work

**Tresor Implementation**: Agents can use all tools (Read, Write, Edit, Bash, Grep, Glob, Task, WebFetch)
**Rationale**: Deep analysis requires comprehensive capabilities
**Contrast**: Skills limited to Read, Grep, Glob

**Our Application**:
- Skills: Quick checks with limited tools
- Agents: Comprehensive work with full toolset
- Commands: Orchestrate agents using Task tool

### Pattern 3: Explicit Invocation Prevents Overhelp

**Tresor Pattern**: User must explicitly call `@agent-name`
**Benefit**: AI doesn't interrupt with unwanted help
**Design Philosophy**: "Help when asked, silent when not"

**Our Application**:
- Background skills for automatic detection
- Explicit agents for deep analysis
- User controls when deep work happens

### Pattern 4: Structured Output for Actionability

**Tresor Pattern**: All agents use consistent output structure
- Section headers (## Issues, ## Recommendations)
- Severity indicators (🚨 CRITICAL, ⚠️ HIGH, 💡 LOW)
- Code examples with before/after
- Actionable next steps

**Our Application**: Standardize agent output format for easy parsing and action

### Pattern 5: Domain Specialization

**Tresor Approach**: Each agent is expert in specific domain
**Benefit**: Deep expertise in narrow area vs shallow across all
**Trade-off**: Need multiple agents for comprehensive analysis

**Our Application**:
```
planner: Task breakdown + approach design
implementer: Code generation following patterns
tester: Test creation + validation
reviewer: Quality assessment + recommendations
learner: Knowledge extraction + storage
```

## Agent Coordination Pattern

### Multi-Agent Workflow (from /review command)

```
/review --scope staged --checks all

Step 1: Read git diff
Step 2: Invoke agents in parallel:
  - Task(subagent_type="code-reviewer") → Code quality
  - Task(subagent_type="security-auditor") → Security
  - Task(subagent_type="performance-tuner") → Performance
  - Task(subagent_type="architect") → Architecture

Step 3: Aggregate results
Step 4: Prioritize issues (CRITICAL → LOW)
Step 5: Generate unified report
```

**Key Insight**: Commands use Task tool to invoke sub-agents, then aggregate results

**Our Application**:
```
/implement feature-description

Step 1: Task(subagent_type="planner") → Create plan
Step 2: Task(subagent_type="implementer") → Write code
Step 3: Task(subagent_type="tester") → Create tests
Step 4: Task(subagent_type="reviewer") → Validate quality
Step 5: If issues: Loop back to implementer
Step 6: Task(subagent_type="learner") → Capture patterns
Step 7: Create PR
```

## Critical Insights for Our Project

### Insight 1: Agent ≠ Always Complex

**Observation**: Some agents are simple (git-commit-helper), others complex (code-reviewer)
**Lesson**: Agent complexity should match task complexity
**Application**: Our agents should have variable depth - planner might be simple, reviewer complex

### Insight 2: Agents Can Invoke Agents

**Pattern**: `@architect` might invoke `@code-reviewer` for code quality within architecture review
**Mechanism**: Task tool enables agent → agent invocation
**Benefit**: Composability, reusability

**Application**: Our `reviewer` might invoke `tester` to validate test coverage as part of review

### Insight 3: Explicit > Automatic for Deep Work

**Philosophy**: Deep analysis should be explicit decision, not automatic
**Rationale**: Prevents AI from wasting tokens on unwanted deep dives
**Balance**: Skills for automatic detection, agents for explicit analysis

**Application**: Never auto-invoke expensive agents - user decides when deep work is needed

### Insight 4: Documentation as Implementation

**Observation**: Agent behavior defined primarily through comprehensive README
**Mechanism**: Examples shape agent behavior more than configuration
**Insight**: For LLM-based agents, documentation IS configuration

**Application**: Our agent docs should include extensive examples of desired behavior

## Recommended Agents for Optimized AI

### Core Agents (Aligned with SPEC.md)

#### 1. planner
**Purpose**: Break down tasks into steps, create approach
**When to Use**: Starting new task
**Tools**: Read (for context), Grep (for similar patterns), Glob (for related files)
**Output**: `.plan/approach.md` with step-by-step plan

#### 2. implementer
**Purpose**: Write code following learned patterns
**When to Use**: Executing planned steps
**Tools**: Full (Read, Write, Edit, Bash, Grep, Glob)
**Input**: `.plan/approach.md`, `.ai-knowledge/patterns.json`
**Output**: Code changes

#### 3. tester
**Purpose**: Create and run tests, validate functionality
**When to Use**: After implementation, before PR
**Tools**: Full (needs Bash for test execution)
**Output**: Test files, coverage reports

#### 4. reviewer
**Purpose**: Self-review code against requirements, patterns, preferences
**When to Use**: Before PR creation
**Tools**: Read, Grep (review-only, no edits)
**Output**: Issues list, pass/fail decision

#### 5. learner (Agent, not Skill)
**Purpose**: Deep analysis of what to capture in knowledge base
**When to Use**: Explicitly after complex task or when pattern emerges
**Tools**: Full (needs to read code, write to .ai-knowledge/)
**Output**: Updated `.ai-knowledge/` files

**Note**: Also have `learner` skill for automatic lightweight capture

### Support Agents

#### 6. debugger
**Purpose**: Analyze failures, spin incidents, blockers
**When to Use**: When stuck, encountering errors
**Tools**: Full
**Output**: Root cause analysis, alternative approaches

## Agent Implementation Checklist

For implementing our agents based on Tresor patterns:

### ✅ Structure
- [ ] Comprehensive README.md with examples
- [ ] Clear expertise definition
- [ ] Tool access specification
- [ ] Integration with related skills/commands

### ✅ Behavior
- [ ] Explicit invocation only (user calls `@agent-name`)
- [ ] Structured output (sections, severity, examples)
- [ ] Actionable recommendations
- [ ] Next steps clearly defined

### ✅ Integration
- [ ] Can be invoked by commands via Task tool
- [ ] Can invoke other agents if needed
- [ ] Produces parseable output for aggregation
- [ ] Works with skills (skills detect, agents analyze)

### ✅ Documentation
- [ ] README.md with comprehensive examples
- [ ] Multiple example scenarios showing input → output
- [ ] Integration examples with skills/commands
- [ ] Clear when-to-use guidance

## Key Differences from Skills

| Aspect | Skills | Agents |
|--------|--------|--------|
| **Invocation** | Automatic (model-invoked) | Manual (`@agent-name`) |
| **Context** | Shared | Separate |
| **Tools** | Limited (Read, Grep, Glob) | Full access |
| **Speed** | Fast (< 1s) | Slower (30s - 5min) |
| **Depth** | Surface-level checks | Deep analysis |
| **Output** | Quick suggestions | Comprehensive reports |
| **Cost** | Low token usage | Higher token usage |

## Coordination Patterns

### Pattern A: Skill → Agent
```
1. Skill detects issue (automatic)
2. User decides to investigate
3. User invokes agent (`@agent-name`)
4. Agent provides deep analysis
```

### Pattern B: Command → Multiple Agents
```
1. User runs command (`/review`)
2. Command invokes agents via Task tool
3. Agents execute in parallel (separate contexts)
4. Command aggregates results
```

### Pattern C: Agent → Agent
```
1. User invokes primary agent
2. Primary agent uses Task tool to invoke secondary agents
3. Results aggregated by primary
4. User sees unified output
```

## Key Takeaways

1. **Agents = Deep Experts**: Manual invocation, comprehensive analysis
2. **Separate Context = Focus**: Dedicated context enables thorough work
3. **Full Tools = Power**: Comprehensive capabilities for complex tasks
4. **Structured Output = Action**: Consistent format enables parsing and aggregation
5. **Composability**: Agents can invoke other agents via Task tool

## Recommendations for Our Implementation

### ✅ Adopt
1. Structured output format with severity levels
2. Separate context for focused analysis
3. Full tool access for agents (vs limited for skills)
4. Task tool for agent coordination
5. Comprehensive README-based documentation

### ⚠️ Adapt
1. Add learning capability (Tresor agents are static)
2. Integration with `.ai-knowledge/` (read patterns, write learnings)
3. Spin detection integration (agents should detect if they're stuck)
4. Experimental validation (Tresor has no metrics)

### ❌ Avoid
1. Automatic invocation of agents (should be explicit)
2. Agents without clear expertise (must have defined domain)
3. Lack of examples (documentation must show behavior)

---

**Next Steps**:
1. Design agent README structure using Tresor format
2. Define tool access boundaries for each agent
3. Create example workflows showing agent coordination
4. Integrate agents with `.ai-knowledge/` system

**See Also**:
- [skills/README.md](../skills/README.md) - Automatic background helpers
- [commands/README.md](../commands/README.md) - Orchestration workflows
- [patterns/README.md](../patterns/README.md) - Reusable meta-patterns
