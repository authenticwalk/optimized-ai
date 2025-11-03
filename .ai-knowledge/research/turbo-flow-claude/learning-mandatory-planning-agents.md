# Learning: Mandatory Planning Agents (Doc-Planner & Microtask-Breakdown)

## Discovery Context
Found in CLAUDE.md, FEEDCLAUDE.md, and throughout turbo-flow-claude documentation as an **enforced requirement**.
Every task, swarm, and hive-mind MUST start by loading doc-planner and microtask-breakdown agents before any implementation work.
This appears in multiple places with emphatic language: "MANDATORY", "ALWAYS", "EVERY task MUST start with".

## The Core Insight
**"ALWAYS START WITH DOC-PLANNER AND MICROTASK-BREAKDOWN"** - Mandatory planning phase before any coding, enforced through repetition and explicit instructions.

### What They Do
```bash
# MANDATORY pattern enforced in every session:
[Single Message - Planning First]:
  # 1. Load mandatory agents
  Read("agents/doc-planner.md")
  Read("agents/microtask-breakdown.md")

  # 2. Execute planning via Task tool
  Task("Doc Planning", "Follow doc-planner methodology to create comprehensive plan", "planner")
  Task("Microtask Breakdown", "Follow microtask-breakdown methodology to break into 10-minute atomic tasks", "analyst")

  # 3. ONLY THEN proceed with implementation
  Task("Implementation", "Build features based on plan", "coder")
  Task("Testing", "Write tests for implementation", "tester")
```

**Doc-Planner Agent** (411 lines):
- Creates comprehensive documentation plans following SPARC workflow
- Implements London School TDD methodology (mock-first, then integration)
- Breaks complex systems into phases with specific, measurable tasks
- Enforces "Brutal Honesty First" - no mocks, no theater, reality check
- MANDATORY: Every phase MUST include atomic task breakdown (task_000 through task_099+)

**Microtask-Breakdown Agent** (241 lines):
- Decomposes phases into atomic 10-minute microtasks
- Follows strict CLAUDE.md principles (no mocks, TDD mandatory, one feature at a time)
- Every task structured as: 2min write test, 5min implement, 3min verify/refactor
- Validates against 100/100 production readiness criteria
- Reality check first: "You have NO prior context or assumptions"

## Deep Analysis

### Why This Approach Was Taken
1. **Prevents rushing to code**: Forces upfront thinking and planning
2. **Atomic decomposition**: Complex tasks broken into manageable 10-minute units
3. **Standardized workflow**: SPARC methodology provides consistent structure
4. **Reality enforcement**: Multiple checks against "theater code" and assumptions
5. **Documentation-first**: Creates artifacts for future reference and onboarding
6. **TDD from start**: Planning includes test strategy before implementation

### What Problem It Solves
- **Scope creep**: Clear boundaries and atomic tasks prevent feature bloat
- **Spinning/confusion**: Detailed plan reduces AI uncertainty
- **Incomplete implementations**: Every task has clear success criteria
- **Technical debt**: TDD and planning reduce shortcuts and hacks
- **Knowledge loss**: Documentation preserves decisions and reasoning
- **Context switching**: Clear phases prevent jumping between tasks

### How It Compares to Alternatives
**Traditional approach** (code-first):
- Pro: Faster initial start
- Pro: Feels productive immediately
- Con: High risk of rework
- Con: No documentation trail
- Con: Easy to miss edge cases

**Their approach** (mandatory planning-first):
- Pro: Clear roadmap before coding
- Pro: Atomic tasks reduce overwhelm
- Pro: Built-in documentation
- Pro: TDD from ground up
- Con: Slower initial start
- Con: Overhead for simple tasks
- Con: Requires discipline to follow

**Hybrid approach** (planning sometimes):
- Pro: Flexible based on task size
- Con: Inconsistent process
- Con: Hard to enforce standards
- Con: No clear "when to plan" criteria

### Edge Cases and Considerations
1. **Simple tasks overhead**: 10-line fix doesn't need full SPARC workflow
2. **Agent availability**: Requires 600+ agent library to be present (external dependency)
3. **Learning curve**: New users must understand SPARC, TDD, atomic tasks
4. **Enforcement challenge**: "Mandatory" only works if actually enforced
5. **Agent synchronization**: Planning agents must stay updated with project evolution
6. **Over-planning risk**: Can spend too much time planning vs doing

## Implementation Details

### Examples from Their Codebase

**Doc-Planner Structure:**
```markdown
# Phase [N]: [Phase Name]

## Overview
- **Purpose**: Clear statement of phase goals
- **Dependencies**: Required completed phases/components
- **Deliverables**: Concrete outputs
- **Success Criteria**: Measurable completion indicators

## SPARC Breakdown
### Specification
- Requirements, Constraints, Invariants

### Pseudocode
- High-level algorithm descriptions

### Architecture
- Components, Interfaces, Data Flow

### Refinement
- Implementation Details, Optimizations, Error Handling

### Completion
- Test Coverage, Integration Points, Validation

## Atomic Task Breakdown (000-099)
### Environment Setup (000-019)
- **task_000**: [Specific 10-minute task]
- **task_001**: [Specific 10-minute task]

### Component Implementation (020-039)
- **task_020**: [Specific 10-minute task]
- **task_021**: [Specific 10-minute task]
```

**Microtask-Breakdown Structure:**
```markdown
# Task [Number]: [Specific Action]
**Estimated Time: [6-10] minutes**

## Context
[YOU ARE STARTING FRESH. Explain what exists NOW and what this adds.]

## Current System State
- [What files/types/methods exist]
- [What has been verified to work]
- [What integration points are confirmed]

## Test First (RED Phase)
```language
[The FAILING TEST to write first]
```

## Minimal Implementation (GREEN Phase)
```language
[The SIMPLEST code that makes the test pass]
```

## Refactored Solution (REFACTOR Phase)
```language
[The cleaned up version]
```

## Verification Commands
```bash
cargo test test_name
cargo build
```

## Success Criteria
- [ ] Test written and initially fails
- [ ] Implementation makes test pass
- [ ] Code compiles without warnings
- [ ] No mocks or stubs - real implementation only
- [ ] Integration point verified
```

**Enforcement Pattern in CLAUDE.md:**
```markdown
## 🔴 MANDATORY: Doc-Planner & Microtask-Breakdown

**EVERY coding session, swarm, and hive-mind MUST start with:**

```bash
# ALWAYS start with mandatory agents
cat $WORKSPACE_FOLDER/agents/doc-planner.md
cat $WORKSPACE_FOLDER/agents/microtask-breakdown.md
```

1. **Doc-Planner Agent**: Creates comprehensive documentation plans
2. **Microtask-Breakdown Agent**: Decomposes phases into atomic 10-minute tasks
```

**Master Prompting Pattern:**
```
"Identify all of the subagents that could be useful in any way for this task
and then figure out how to utilize the claude-flow hivemind to maximize your
ability to accomplish the task. Start with doc-planner and microtask-breakdown."
```

## Applicability to Our Project

### Alignment with Our Goals
**MIXED ALIGNMENT** with potential for adaptation:

**MINIMIZE** (Principle 1):
- ❌ **CONFLICTS**: 411+241=652 lines loaded EVERY time is massive overhead
- ❌ Doc-planner alone is 411 lines (4x our target .cursorrules)
- ❌ Forces loading of full SPARC methodology even for simple tasks
- ✅ Atomic tasks concept aligns with efficiency

**SEPARATE** (Principle 2):
- ✅ **ALIGNS**: Planning separated from implementation
- ✅ Clear phase boundaries (doc → microtask → implement)
- ✅ Agents are separate, composable units
- ❌ But forces loading both planning agents together always

**VALIDATE** (Principle 3):
- ⚠️ **UNKNOWN**: No evidence of A/B testing mandatory vs optional planning
- ⚠️ No metrics showing when planning overhead is worth it
- ⚠️ "100/100 production readiness" claims but no validation methodology
- ✅ Built-in success criteria for each task

**LEARN** (Principle 4):
- ✅ **ALIGNS**: Documentation creates learning artifacts
- ✅ Each task documents decisions and context
- ❌ But doesn't adapt based on learnings (always same overhead)

### Specific Ways This Applies
1. **Planning phase**: We could benefit from structured planning for complex features
2. **Atomic task decomposition**: 10-minute tasks align with focus and measurement
3. **Reality checks**: "No prior context" reminders combat hallucination
4. **TDD enforcement**: Built into planning prevents "code-first" shortcuts
5. **Documentation artifacts**: Preserves knowledge for future reference

### Integration Points
```javascript
// Adapted approach for our project:
// NOT mandatory for all tasks, but available as opt-in skill

// Simple task (no planning needed):
[Single Message]:
  Read("skills/firebase-auth.skill")  // 100 lines, focused
  Task("Add login button", "Implement Firebase auth button", "coder")

// Complex feature (planning beneficial):
[Single Message]:
  Read("skills/planning.skill")  // Condensed 150 lines vs 652
  Task("Plan architecture", "Break down multi-week feature into phases", "planner")
  Task("Create microtasks", "Define 10-minute atomic tasks", "planner")

  // Then implementation
  Task("Implement phase 1", "Build first phase", "coder")
  Task("Test phase 1", "Verify functionality", "tester")
```

### What Would We Need to Change?
1. **Make planning optional, not mandatory**: Load only when task complexity warrants it
2. **Condense planning agents**: 652 lines → ~150 lines core concepts
3. **Adaptive loading**: Small task = no planning, medium = basic, complex = full SPARC
4. **Validate experimentally**: Measure when planning overhead pays off
5. **Simplify SPARC**: Full 5-phase workflow may be overkill for many tasks
6. **Remove agent library dependency**: Don't require external 600+ agent repo

## Experiments & Validation

### Hypothesis to Test
"Mandatory upfront planning reduces total tokens and time for complex tasks (>2 hours estimated) but increases overhead for simple tasks (<30 min)"

### Experimental Design
**Independent Variable**: Planning approach
- Control: No planning (code immediately)
- Treatment 1: Optional planning (developer decides)
- Treatment 2: Mandatory planning (their approach)
- Treatment 3: Adaptive planning (auto-decide based on complexity)

**Task Categories**:
- Simple (estimate <30 min): "Add logout button", "Fix typo in error message"
- Medium (estimate 30min-2hr): "Add password reset flow", "Implement pagination"
- Complex (estimate >2hr): "Build admin dashboard", "Add real-time chat"

**Metrics**:
- Total tokens used (planning + implementation)
- Total time (start to working code)
- Rework required (changes after initial implementation)
- Quality score (tests pass, requirements met, no bugs)
- Developer satisfaction (subjective rating)

**Sample Size**: 10 tasks per category per treatment = 120 total trials

### Expected Results
**Simple Tasks**:
- Control: Fastest, lowest tokens
- Mandatory planning: 3-5x overhead, no quality improvement
- Adaptive: Similar to control (skips planning)

**Medium Tasks**:
- Control: Some rework needed
- Mandatory planning: Faster overall (less rework)
- Adaptive: Similar to mandatory

**Complex Tasks**:
- Control: Significant rework, higher total cost
- Mandatory planning: Lower total cost despite upfront overhead
- Adaptive: Similar to mandatory

**Decision Criteria**:
- If mandatory planning improves complex tasks by >30%: **ADOPT FOR COMPLEX**
- If overhead on simple tasks is >100%: **MAKE OPTIONAL**
- If adaptive performs within 10% of optimal in all cases: **ADOPT ADAPTIVE**

## Related Learnings
- [learning-operation-batching.md](./learning-operation-batching.md) - Agents loaded in batch
- [pattern-atomic-task-breakdown.md](./pattern-atomic-task-breakdown.md) - 10-minute task methodology
- [pattern-sparc-methodology.md](./pattern-sparc-methodology.md) - SPARC workflow details
- [mistake-over-complexity.md](./mistake-over-complexity.md) - When mandatory becomes overhead

## Our Project Application

### Recommendations
1. **ADAPT, DON'T ADOPT**: Planning is valuable but shouldn't be mandatory for all tasks
2. **Create condensed planning skill**: 150 lines with core concepts, not 652 lines
3. **Implement adaptive trigger**: Use task complexity to auto-decide planning level
4. **Validate experimentally**: Run A/B test to find optimal planning thresholds
5. **Make SPARC optional**: Full 5-phase workflow for complex projects only

### Implementation Strategy
**Phase 1**: Create planning skill (not mandatory)
```
skills/planning.skill (~150 lines)
├── Core Concepts
│   - Atomic task decomposition (10-minute rule)
│   - TDD-first planning
│   - Reality check (no assumptions)
│   - Success criteria definition
├── SPARC Lite
│   - Specification (requirements)
│   - Architecture (high-level design)
│   - Refinement (implementation approach)
└── When to Use
    - Complex features (>2 hour estimate)
    - Multi-developer coordination
    - High-risk changes
    - New architectural patterns
```

**Phase 2**: Add to .cursorrules (5 lines)
```
# Planning
- For complex tasks (>2hr), consider loading skills/planning.skill
- Break work into 10-minute atomic tasks when possible
- Define success criteria before implementation
- Reality check: What exists NOW vs what's assumed?
```

**Phase 3**: Validate improvement
```javascript
Experiment: "planning-threshold"
- Control: No planning prompt
- Treatment: Prompt to consider planning for complex tasks
- Measure: When developers load planning skill and if it helps
- Refine: Adjust complexity threshold based on data
```

**Phase 4**: Consider automation
```javascript
// Future: Auto-suggest planning for complex tasks
if (taskEstimate > 120minutes || fileChanges > 5 || newComponents > 2) {
  suggestSkill("planning.skill", "This appears complex. Consider planning first?");
}
```

### Success Criteria
- ✅ Planning skill created and validated (<200 lines)
- ✅ Experimental data shows when planning overhead pays off
- ✅ Developers use planning for ~20% of tasks (the complex ones)
- ✅ Planning improves outcomes when used (less rework, faster overall)
- ✅ No mandatory overhead on simple tasks
- ✅ Documentation artifacts captured in `.ai-knowledge/`

## Key Takeaways
1. **Planning has value**: Structured upfront thinking reduces rework on complex tasks
2. **Not all tasks need planning**: Simple tasks suffer from mandatory planning overhead
3. **Atomic tasks are gold**: 10-minute decomposition works well across complexity levels
4. **Reality checks matter**: "No prior context" reminders combat AI hallucination
5. **TDD integration**: Planning that includes test strategy is powerful
6. **652 lines is too much**: Core concepts can be condensed significantly
7. **Mandatory isn't optimal**: Adaptive loading based on complexity is smarter
8. **Validate the threshold**: Experiment to find when planning pays off

## Quality Score
**Concept Quality**: 85/100
- Excellent: Structured planning, atomic tasks, TDD integration, reality checks
- Good: Documentation artifacts, clear phases
- Concerns: Mandatory overhead, 652-line burden, no validation of thresholds

**Implementation Quality**: 60/100
- Excellent: Detailed agent specs, clear methodology
- Good: Repetition ensures it's seen
- Poor: No experimental validation, no adaptive logic, too heavyweight

**Production Readiness for Our Project**: 70/100
- Ready: Core concepts are proven and valuable
- Adapt: Need to condense, make optional, validate thresholds
- Validate: Must experiment to find when planning overhead is worth it
- Don't adopt wholesale: Mandatory overhead conflicts with MINIMIZE principle
