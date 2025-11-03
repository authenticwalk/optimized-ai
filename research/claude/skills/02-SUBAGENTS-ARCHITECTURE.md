# Claude Code Subagents - Architecture & Patterns

**Status**: ✅ Validated by official documentation and production implementations

---

## Table of Contents
1. [What Are Subagents?](#what-are-subagents)
2. [Architecture Patterns](#architecture-patterns)
3. [File Structure & Configuration](#file-structure--configuration)
4. [Tool Management](#tool-management)
5. [Context Isolation](#context-isolation)
6. [Invocation Methods](#invocation-methods)
7. [Best Practices](#best-practices)
8. [Production Examples](#production-examples)
9. [Integration with Skills](#integration-with-skills)

---

## What Are Subagents?

### Definition
**Subagents are specialized AI assistants that Claude Code can delegate tasks to. Each operates with its own context window, custom system prompt, and configurable tool access.**

### Key Benefits

**Separate Context Windows**
```
Main Agent: 200k context
├─ Subagent A: Fresh 200k context
├─ Subagent B: Fresh 200k context
└─ Subagent C: Fresh 200k context

Total effective context: 800k!
```

**Task Specialization**
- Each agent has specific expertise
- Custom system prompts per domain
- Isolated focus prevents confusion

**Context Preservation**
- Main agent doesn't get polluted
- Can handle longer overall sessions
- Clear separation of concerns

### When to Use Subagents

✅ **Use subagents for:**
- Complex multi-step tasks
- Tasks requiring different expertise
- When context is getting bloated
- Clear handoff points exist
- Parallel execution opportunities
- Need for specialized focus

❌ **Don't use subagents for:**
- Simple, single-step tasks
- When overhead exceeds benefit
- Tasks requiring continuous context
- Rapid back-and-forth iteration

---

## Architecture Patterns

### 1. Hub-and-Spoke Pattern

**Description**: Central orchestrator routes tasks to specialized agents

```
                    ┌─────────────┐
                    │   Main      │
                    │ Orchestrator│
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    ┌─────────┐      ┌─────────┐      ┌─────────┐
    │ Planner │      │ Builder │      │Reviewer │
    └─────────┘      └─────────┘      └─────────┘
```

**Benefits**
- Prevents agents from self-selecting incorrectly
- Eliminates peer-to-peer communication chaos
- Central routing logic
- Clear responsibility boundaries

**Implementation**
```yaml
---
name: task-orchestrator
description: Main orchestrator. Routes tasks to specialized agents based on type. Use as primary entry point for complex workflows.
---

# Task Orchestrator

## Responsibilities
1. Analyze incoming request
2. Break down into subtasks
3. Route to appropriate subagents
4. Integrate results
5. Report back to user

## Routing Logic
- Requirements gathering → pm-spec agent
- Architecture decisions → architect-review agent
- Implementation → implementer-tester agent
- Code quality → code-reviewer agent
- Debugging → debugger agent

## Handoff Protocol
For each subtask:
1. Prepare context summary
2. Invoke appropriate agent
3. Validate result
4. Update status
5. Proceed to next step
```

### 2. Multi-Stage Pipeline Pattern

**Description**: Sequential workflow with clear stages

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ PM-Spec  │ → │Architect │ → │Implement │
│  Agent   │    │  Agent   │    │  Agent   │
└──────────┘    └──────────┘    └──────────┘
     │               │                │
     ▼               ▼                ▼
  spec.md         ADR.md         code + tests
```

**Stages**

1. **PM-Spec**
   - Reads enhancement requests
   - Writes specifications
   - Asks clarifying questions
   - Outputs: `spec.md`, questions list

2. **Architect-Review**
   - Validates design against constraints
   - Produces Architecture Decision Record (ADR)
   - Defines implementation guardrails
   - Outputs: `ADR.md`, technical approach

3. **Implementer-Tester**
   - Implements code per spec and ADR
   - Writes comprehensive tests
   - Updates documentation
   - Outputs: code, tests, docs

**Benefits**
- Clear handoff points
- Human review gates between stages
- Audit trail of decisions
- Prevents scope creep

**Example Configuration**

```yaml
# pm-spec.md
---
name: pm-spec
description: Product Manager agent. Reads enhancement requests, writes specifications, identifies ambiguities. Use FIRST for new features. Outputs to docs/specs/.
tools: Read, Write, Grep, Glob
model: sonnet
---

# PM-Spec Agent

## Role
Act as a Product Manager analyzing feature requests.

## Process
1. Read enhancement from enhancements/_queue.json
2. Analyze requirements for completeness
3. Identify ambiguities and assumptions
4. Ask clarifying questions
5. Write specification to docs/specs/{slug}.md
6. Update queue status to READY_FOR_ARCH

## Definition of Done
- [ ] Spec includes user stories
- [ ] Acceptance criteria defined
- [ ] Edge cases identified
- [ ] Questions documented
- [ ] Dependencies listed

## Stop Conditions
- If requirements are ambiguous: ASK USER
- If conflicts with existing features: FLAG
- If scope too large: SUGGEST BREAKDOWN
```

```yaml
# architect-review.md
---
name: architect-review
description: Architecture reviewer. Validates designs, produces ADRs with technical decisions and guardrails. Use after PM-Spec completes. Outputs to docs/decisions/.
tools: Read, Write, Grep, Glob
model: sonnet
---

# Architect-Review Agent

## Role
Act as Lead Architect reviewing feature designs.

## Process
1. Read spec from docs/specs/{slug}.md
2. Analyze against system constraints
3. Design technical approach
4. Identify risks and tradeoffs
5. Write ADR to docs/decisions/ADR-{slug}.md
6. Update queue status to READY_FOR_BUILD

## ADR Template
- Context: Why this decision matters
- Decision: What we're doing
- Alternatives: What we considered
- Consequences: Tradeoffs and risks
- Guardrails: Implementation constraints

## Definition of Done
- [ ] ADR complete with all sections
- [ ] Technical approach clear
- [ ] Risks documented
- [ ] Guardrails defined
- [ ] Files to change listed

## Stop Conditions
- If architectural conflict: ESCALATE
- If public API change: ASK USER
- If performance concern: FLAG
```

### 3. Product Trinity Pattern

**Description**: Three specialized agents compress development cycle

```
    ┌─────────────┐
    │   Product   │
    │   Manager   │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │ UX Designer │
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │Implementation│
    │  Specialist │
    └─────────────┘
```

**Responsibilities**

**Product Manager**
- Requirements analysis
- User story creation
- Acceptance criteria
- Priority setting

**UX Designer**
- Interface planning
- User flow design
- Wireframing
- Interaction patterns

**Implementation Specialist**
- Code generation
- Testing
- Integration
- Deployment

**Benefits**
- Mirrors real team structure
- Clear ownership
- Comprehensive coverage
- Weeks → Hours compression

### 4. Parallel Execution Pattern

**Description**: Independent agents work simultaneously

```
                ┌──────────────┐
                │Main Orchestr.│
                └───────┬──────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │Backend  │    │Frontend │    │Testing  │
   │ Agent   │    │ Agent   │    │ Agent   │
   └─────────┘    └─────────┘    └─────────┘
        │               │               │
        └───────────────┴───────────────┘
                        │
                        ▼
                  Integration
```

**Constraints**
- Only for disjoint features
- Different modules/files
- No shared dependencies
- Clear boundaries

**Safety Measures**
```markdown
## Conflict Detection
Before parallel execution:
1. List all files each agent will touch
2. Check for overlaps
3. If overlap detected: execute sequentially
4. If disjoint: proceed in parallel
```

---

## File Structure & Configuration

### Storage Locations

**Project-level** (highest priority)
```
.claude/agents/
├── pm-spec.md
├── architect-review.md
├── implementer-tester.md
├── code-reviewer.md
└── debugger.md
```

**User-level**
```
~/.claude/agents/
├── personal-reviewer.md
└── helper.md
```

**Plugin-provided**
```
[plugin-dir]/agents/
└── specialized-agent.md
```

### YAML Configuration Format

```yaml
---
name: agent-identifier
description: When and how to use this agent
tools: tool1, tool2, tool3  # Optional
model: sonnet              # Optional
---

[System prompt defining agent's expertise,
approach, and behavioral guidelines]
```

### Required Fields

**name**
- Lowercase with hyphens
- Unique identifier
- Example: `code-reviewer`, `test-generator`

**description**
- Natural language purpose
- Should include "when to use"
- Can include "PROACTIVELY" for automatic invocation
- Example: "Code quality reviewer. Use PROACTIVELY after implementation. Checks quality, security, maintainability."

### Optional Fields

**tools**
- Comma-separated list
- Restricts available operations
- Omit to inherit all tools from main agent
- Example: `Read, Grep, Glob` (read-only)

**model**
- Specify model: `sonnet`, `opus`, `haiku`
- Use `'inherit'` to match main conversation
- Affects cost and capability

---

## Tool Management

### Tool Access Levels

**Level 1: Read-Only** (safest)
```yaml
tools: Read, Grep, Glob
```
Use for: reviewers, analyzers, reporters

**Level 2: Code Modification**
```yaml
tools: Read, Write, Edit, Grep, Glob
```
Use for: implementers, refactorers

**Level 3: Full Access**
```yaml
tools: Read, Write, Edit, Grep, Glob, Bash
```
Use for: builders, testers, deployers

**Level 4: Inherit All** (including MCP)
```yaml
# Omit tools field entirely
```
Use for: orchestrators, full-stack agents

### Best Practices

**Principle of Least Privilege**
```yaml
# ✅ Good: PM only needs to read and write specs
---
name: pm-spec
tools: Read, Write, Grep, Glob
---

# ❌ Bad: PM doesn't need bash access
---
name: pm-spec
# tools field omitted - grants everything!
---
```

**Permission Hygiene**
- Read-heavy agents: `Read, Grep, Glob`
- Implementers: add `Write, Edit`
- System operations: add `Bash`
- Default (omit field): grants all including MCP

---

## Context Isolation

### How It Works

```
Main Context Window (200k tokens):
├─ User conversation history
├─ Current task
├─ Core skills loaded
└─ Project CLAUDE.md

Subagent Context Window (separate 200k):
├─ System prompt (agent-specific)
├─ Task delegation from main
├─ Agent-specific skills
└─ Fresh perspective
```

### Benefits

**1. Pollution Prevention**
- Main conversation stays clean
- Each agent has focused context
- No cross-contamination

**2. Extended Sessions**
```
Without subagents:
- Single 200k context fills up
- Must /clear or /compact
- Loses valuable history

With subagents:
- Main context: high-level tracking
- Subagents: detailed work
- Effective context: 4x-10x larger
```

**3. Specialized Focus**
- Each agent sees only relevant information
- No distraction from unrelated tasks
- Better quality per domain

### Example: Context Flow

```markdown
## Main Agent Context
```
User: "Add Firebase authentication"
Main: [loads firebase-related skills]
Main: "Delegating to implementer-tester agent..."
```

## Implementer-Tester Context (Fresh)
```
System: You are an implementation specialist...
Task: Implement Firebase authentication
Relevant files: [provided by main agent]
[Loads: firebase-auth skill, testing skill]
[No pollution from previous unrelated tasks]
```

## Back to Main Context
```
Implementer: [returns completed code + tests]
Main: "Implementation complete. Delegating to reviewer..."
```

## Code-Reviewer Context (Fresh)
```
System: You are a code quality reviewer...
Task: Review Firebase auth implementation
Code: [provided by main agent]
[Loads: code-review skill, security skill]
[No bias from implementation process]
```
```

---

## Invocation Methods

### 1. Automatic Delegation

**Trigger words in description**
```yaml
description: "Use PROACTIVELY after implementation completes..."
description: "MUST BE USED for all architecture decisions..."
description: "Automatically apply when debugging errors..."
```

**Example Flow**
```
User: "Implement user registration"
Claude: [detects implementation task]
Claude: [automatically delegates to implementer agent]
Implementer: [completes work]
Claude: [detects completion]
Claude: [automatically delegates to code-reviewer]
Reviewer: [validates quality]
Claude: [returns final result]
```

### 2. Explicit Invocation

**By user**
```
"Use the code-reviewer agent to check my changes"
"Have the debugger subagent investigate this error"
"Ask the architect agent to design this feature"
```

**By main agent**
```markdown
## In system prompt
When implementation complete:
1. Delegate to code-reviewer agent
2. If issues found: return to implementer
3. If passing: report to user
```

### 3. Hook-Based Invocation

**Pattern: Hooks suggest, humans approve**

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "pm-spec",
        "hooks": [
          {
            "type": "command",
            "command": "echo '\n✅ Spec complete. Next: claude --agents architect-review ...'"
          }
        ]
      }
    ]
  }
}
```

**Benefits**
- Prevents runaway agent chains
- Human in the loop
- Quick review before proceeding
- Copy-paste to approve

---

## Best Practices

### 1. Single-Responsibility Design

✅ **Do**: Clear, focused purpose
```yaml
---
name: test-generator
description: Generates comprehensive unit tests with >90% coverage. Use after implementation completes.
---
```

❌ **Don't**: Multiple responsibilities
```yaml
---
name: do-everything
description: Writes code, tests it, reviews it, deploys it, and makes coffee.
---
```

### 2. Definition of Done

**Every agent should have clear completion criteria**

```markdown
## Definition of Done
- [ ] Code implements all requirements from spec
- [ ] All tests passing (>90% coverage)
- [ ] No linter errors
- [ ] Documentation updated
- [ ] Performance acceptable
- [ ] Security reviewed
```

### 3. Stop Conditions

**Explicit gates for human intervention**

```markdown
## Stop and Ask User If:
- Requirements are ambiguous
- Multiple valid approaches exist
- Public API changes required
- Security implications detected
- Breaking changes needed
```

### 4. Audit Trail

**Use slugs across artifacts**

```
Enhancement: use-case-presets

Queue: enhancements/_queue.json
  ├─ id: "use-case-presets"
  └─ status: "READY_FOR_ARCH"

Notes: docs/claude/working-notes/use-case-presets.md
  └─ PM findings, architect notes

Decision: docs/claude/decisions/ADR-use-case-presets.md
  └─ Technical approach

Code: src/features/use-case-presets/
  └─ Implementation

Commit: "feat(use-case-presets): Add preset management"
```

### 5. Humans-in-the-Loop (HITL)

**Pattern**: Hooks print suggested commands

```bash
# Hook output after PM completes
✅ PM-Spec complete: use-case-presets
📋 Spec: docs/specs/use-case-presets.md
❓ Questions: 2 items need clarification

Next step (copy-paste to proceed):
  claude --agents architect-review "Review spec: use-case-presets"
```

**Benefits**
- Forces quick review
- Prevents runaway chains
- Enables corrections before proceeding
- Maintains human oversight

### 6. Error Handling

**Agent should gracefully handle failures**

```markdown
## Error Handling
If task cannot be completed:
1. Document what was attempted
2. Explain why it failed
3. Suggest alternative approaches
4. Update status to BLOCKED
5. DO NOT proceed to next step
```

### 7. Logging & Observability

**Track agent execution**

```bash
# hooks.log
2025-11-03 14:23:45 [pm-spec] Started: use-case-presets
2025-11-03 14:24:12 [pm-spec] Complete: spec written
2025-11-03 14:24:15 [hook] SubagentStop → suggest next step
2025-11-03 14:30:01 [architect-review] Started: use-case-presets
2025-11-03 14:31:22 [architect-review] Complete: ADR written
```

---

## Production Examples

### Example 1: Code Reviewer

```yaml
---
name: code-reviewer
description: Expert code review specialist. Reviews for quality, security, performance, and maintainability. Use PROACTIVELY after implementation completes.
tools: Read, Grep, Glob
model: sonnet
---

# Code Reviewer Agent

## Role
Senior engineer conducting thorough code review.

## Review Checklist

### Code Quality
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Clear, descriptive names
- [ ] Appropriate abstractions
- [ ] SOLID principles followed

### Security
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] Auth checks in place

### Performance
- [ ] No N+1 queries
- [ ] Appropriate caching
- [ ] Efficient algorithms
- [ ] No memory leaks
- [ ] Database indexes considered

### Testing
- [ ] >90% coverage
- [ ] Edge cases tested
- [ ] Error paths tested
- [ ] Integration tests present
- [ ] Tests are maintainable

### Documentation
- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] Changelog updated

## Output Format
```
## Code Review: [Feature Name]

### ✅ Strengths
- [List what's well done]

### ⚠️ Issues Found
- [ ] Critical: [Must fix before merge]
- [ ] Important: [Should fix]
- [ ] Nice-to-have: [Consider fixing]

### 📝 Detailed Feedback
[File-by-file review]

### 🎯 Recommendation
[APPROVE / REQUEST CHANGES / NEEDS REWORK]
```

## Escalation
If critical security or architectural issues:
- Mark as BLOCK
- Document issue clearly
- Suggest remediation
- DO NOT auto-approve
```

### Example 2: Debugger

```yaml
---
name: debugger
description: Systematic debugging specialist. Investigates errors, test failures, and unexpected behavior. Use PROACTIVELY when encountering bugs or failures.
tools: Read, Grep, Bash, Glob
model: sonnet
---

# Debugger Agent

## Role
Expert debugger using systematic root cause analysis.

## Debugging Process

### 1. Reproduce
- Gather error message/stack trace
- Identify steps to reproduce
- Verify reproduction reliability

### 2. Isolate
- Identify failing component
- Create minimal reproduction
- Eliminate variables

### 3. Analyze
- Read relevant code
- Check recent changes (git log)
- Review related tests
- Examine logs

### 4. Hypothesize
- Form theory about root cause
- Identify what assumptions might be wrong
- Consider edge cases

### 5. Test Hypothesis
- Add debug logging if needed
- Run targeted tests
- Verify theory

### 6. Fix
- Implement minimal fix
- Verify fix resolves issue
- Ensure no regressions

### 7. Prevent Recurrence
- Add regression test
- Update documentation if needed
- Consider if similar bugs exist elsewhere

## Anti-Patterns to Avoid
❌ Guess-and-check debugging
❌ Changing multiple things at once
❌ Not understanding the fix
❌ Skipping regression test

## Output Format
```
## Debug Report: [Issue]

### Problem
[Clear description]

### Root Cause
[What actually caused it]

### Fix
[What was changed and why]

### Verification
- [ ] Issue resolved
- [ ] Tests passing
- [ ] No regressions
- [ ] Regression test added

### Prevention
[How to avoid in future]
```
```

### Example 3: Test Generator

```yaml
---
name: test-generator
description: Test creation specialist. Generates comprehensive unit and integration tests with >90% coverage. Use after implementation or when adding tests to existing code.
tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
---

# Test Generator Agent

## Role
QA Engineer creating comprehensive test suites.

## Test Strategy

### Coverage Target
- Minimum: 90% line coverage
- Prefer: 100% for critical paths
- Include: edge cases, error paths, integration

### Test Structure
```typescript
describe('Component/Function Name', () => {
  describe('happy paths', () => {
    it('should handle normal case', () => {})
  })

  describe('edge cases', () => {
    it('should handle empty input', () => {})
    it('should handle null input', () => {})
    it('should handle maximum values', () => {})
  })

  describe('error cases', () => {
    it('should throw on invalid input', () => {})
    it('should handle network errors', () => {})
  })

  describe('integration', () => {
    it('should work with real dependencies', () => {})
  })
})
```

### What to Test
- ✅ All public APIs
- ✅ Edge cases (empty, null, undefined, max values)
- ✅ Error conditions
- ✅ Boundary conditions
- ✅ Integration points

### What NOT to Test
- ❌ Third-party library internals
- ❌ Framework behavior
- ❌ What TypeScript catches

## Test Patterns

### Arrange-Act-Assert
```typescript
it('should calculate total correctly', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }]

  // Act
  const total = calculateTotal(items)

  // Assert
  expect(total).toBe(30)
})
```

### Mocking
```typescript
it('should call API with correct params', async () => {
  // Arrange
  const mockApi = jest.fn()
  const service = new Service(mockApi)

  // Act
  await service.fetchUser('123')

  // Assert
  expect(mockApi).toHaveBeenCalledWith('/users/123')
})
```

## Definition of Done
- [ ] >90% coverage achieved
- [ ] All edge cases covered
- [ ] Error paths tested
- [ ] Integration tests present
- [ ] Tests are maintainable
- [ ] Tests document behavior
- [ ] All tests passing

## Verification
Run coverage report:
```bash
npm run test:coverage
# or
jest --coverage
```
```

---

## Integration with Skills

### Skills + Subagents = Powerful Combination

**Pattern**: Subagent loads relevant skills

```yaml
---
name: firebase-implementer
description: Firebase implementation specialist. Use for Firebase-related features.
---

# Firebase Implementer

When activated, this agent automatically loads:
- firebase-auth skill (if auth-related)
- firebase-firestore skill (if database-related)
- firebase-functions skill (if functions-related)
- testing-patterns skill (always)

The agent focuses on Firebase best practices
while skills provide domain-specific guidance.
```

### Skill Loading Strategy

**Automatic**
```
User: "Add Firebase authentication"
Main: Delegates to firebase-implementer agent
Agent: [fresh context]
Agent: [loads firebase-auth skill]
Agent: [loads testing skill]
Agent: [implements with skill guidance]
```

**Explicit**
```markdown
## Required Skills
This agent requires:
- firebase-auth
- testing-patterns
- error-handling

Load these skills when agent activates.
```

---

## Key Takeaways for Optimized AI

### 1. Start with Hub-and-Spoke
- Main orchestrator
- 3-5 specialized agents
- Clear routing logic

### 2. Use Pipeline for Complex Features
- pm-spec → architect-review → implementer-tester
- Human gates between stages
- Audit trail with slugs

### 3. Implement HITL Pattern
- Hooks suggest next steps
- Humans copy-paste to approve
- Prevents runaway chains

### 4. Tool Restrictions Matter
- Reviewers: Read-only
- Implementers: Read + Write + Edit
- Orchestrators: All tools

### 5. Track Everything
- Agent start/stop in logs
- Status in queue
- Decisions in ADRs
- Notes in working-notes/

### 6. Measure Benefits
- Context window utilization
- Quality improvements
- Time savings
- Human intervention points

---

## Resources

### Official
- [Claude Code Subagents Docs](https://docs.claude.com/en/docs/claude-code/sub-agents)

### Community
- [wshobson/agents](https://github.com/wshobson/agents) - Multi-agent orchestration
- [vanzan01/claude-code-sub-agent-collective](https://github.com/vanzan01/claude-code-sub-agent-collective) - Hub-and-spoke research
- [Best Practices Guide](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)

---

**Next**: Read [03-MCP-SERVERS.md](03-MCP-SERVERS.md) for Model Context Protocol integration
