# Pattern: Atomic Task Breakdown (10-Minute Methodology)

## Discovery Context
Found throughout doc-planner.md (line 48-68) and microtask-breakdown.md (line 57-70) as a **core requirement**.
Every phase must include numbered atomic tasks (task_000 through task_099+) with "FAILURE TO INCLUDE ATOMIC TASK BREAKDOWNS = INCOMPLETE DOCUMENTATION".
The 10-minute rule appears as a strict constraint: "Each task must be completable in 10 minutes" with breakdown of 2min test, 5min implement, 3min verify.

## The Core Insight
**"10-MINUTE ATOMIC TASKS WITH TDD STRUCTURE"** - Every task sized to fit: 2 minutes write failing test, 5 minutes implement minimal solution, 3 minutes verify and refactor.

### What They Do
```markdown
## Atomic Task Breakdown (000-099)

### Environment Setup (000-019)
- **task_000**: Verify Rust installation (10 min)
  - RED: Test version check (2min)
  - GREEN: Install if missing (5min)
  - REFACTOR: Verify toolchain (3min)

- **task_001**: Create project structure (10 min)
  - RED: Test directories exist (2min)
  - GREEN: Run cargo init (5min)
  - REFACTOR: Add .gitignore (3min)

### Core Implementation (020-039)
- **task_020**: Implement search method (10 min)
  - RED: Write failing test for basic search (2min)
  - GREEN: Minimal implementation returning results (5min)
  - REFACTOR: Clean up error handling (3min)

- **task_021**: Add query parsing (10 min)
  - RED: Test query parser with valid input (2min)
  - GREEN: Implement basic parser (5min)
  - REFACTOR: Add edge case handling (3min)

### Integration Tests (040-059)
- **task_040**: Basic integration test (10 min)
  - RED: Test full search workflow (2min)
  - GREEN: Wire components together (5min)
  - REFACTOR: Add assertions (3min)
```

**Task Numbering Convention:**
```
00a-00z: Foundation/Prerequisites (types, structs, basic setup)
01-09:   Core implementation methods (one method per task)
10-19:   Unit tests (grouped by functionality)
20-29:   Integration tests
30-39:   Error handling and validation
40-49:   Documentation and examples
50+:     Performance and optimization (only if needed)
```

**Task Template Structure:**
```markdown
# Task [Number]: [Specific Action]
**Estimated Time: [6-10] minutes**

## Context
[What exists NOW and what this task adds]

## Current System State
- [Files/types/methods that exist]
- [What has been verified to work]
- [Integration points confirmed]

## Your Task
[ONE specific thing - a single method, test, or small feature]

## Test First (RED Phase)
```language
[The FAILING TEST - must actually test the feature]
```

## Minimal Implementation (GREEN Phase)
```language
[SIMPLEST code that makes test pass - no extras]
```

## Refactored Solution (REFACTOR Phase)
```language
[Cleaned up version - better names, extracted methods]
```

## Verification Commands
```bash
cargo test test_name
cargo build
```

## Success Criteria
- [ ] Test written and initially fails with expected error
- [ ] Implementation makes test pass
- [ ] Code compiles without warnings
- [ ] No mocks or stubs - real implementation only
- [ ] Integration point verified (if applicable)

## Dependencies Confirmed
- [Library version in Cargo.toml]
- [API endpoint verified]
- [File/module present]

## Next Task
[What logically follows this atomic unit]
```

## Deep Analysis

### Why This Approach Was Taken
1. **Cognitive load management**: 10 minutes is sustainable focus period
2. **Continuous progress**: Many small wins vs long uncertain tasks
3. **Easy estimation**: Predictable 10-minute blocks for planning
4. **Natural breakpoints**: Can stop/start without losing context
5. **TDD enforcement**: Structure forces test-first development
6. **Verifiable completion**: Clear success criteria for each task
7. **Parallelizable**: Multiple developers can work on different tasks
8. **Reduces overwhelm**: Complex project becomes series of simple tasks

### What Problem It Solves
- **Scope uncertainty**: No vague "implement feature X" tasks
- **Procrastination**: Small tasks reduce activation energy
- **Context switching**: Complete task in one sitting
- **Progress tracking**: Easy to count completed tasks
- **Estimation errors**: 10-minute blocks are easier to estimate accurately
- **Incomplete work**: Every task has working code at end
- **Testing neglect**: TDD structure baked into every task
- **Hidden complexity**: Forces explicit breakdown of dependencies

### How It Compares to Alternatives
**User story sizing** (traditional agile):
- Pro: User-focused, end-to-end thinking
- Pro: Flexible sizing (1pt, 2pt, 5pt, etc.)
- Con: Still requires breakdown for implementation
- Con: Points are subjective and vary by team
- Con: No built-in TDD structure

**Hour-based estimation**:
- Pro: Direct time correlation
- Pro: Easier to schedule
- Con: Longer tasks increase uncertainty
- Con: 4-hour task can have many unknowns
- Con: Harder to track incremental progress

**"Small, medium, large" buckets**:
- Pro: Simple categorization
- Pro: Fast to assign
- Con: Too coarse for detailed planning
- Con: Large tasks still need breakdown
- Con: No guidance on actual size

**Their 10-minute atomic tasks**:
- Pro: Ultra-specific sizing (6-10 minutes)
- Pro: Built-in TDD structure (2-5-3 split)
- Pro: Verifiable completion criteria
- Pro: Easy progress tracking
- Con: Breakdown overhead (100+ tasks for complex feature)
- Con: Can feel constraining for experienced devs
- Con: Not all work fits 10-minute mold

### Edge Cases and Considerations
1. **Research/learning tasks**: May not fit 10-minute mold (learning is non-linear)
2. **Integration complexity**: Some integrations just take longer than 10 minutes
3. **Debugging**: Real bugs don't respect time boxes
4. **Creative work**: UI design, architecture decisions need thinking time
5. **External dependencies**: Waiting for API responses, builds, deployments
6. **First-time tasks**: 10 minutes assumes competence with tools
7. **Context-heavy work**: Some tasks need 20+ minutes of context loading
8. **Meeting interruptions**: Hard to protect 10-minute blocks in practice

## Implementation Details

### Examples from Their Codebase

**Microtask-Breakdown Agent Sizing Guidance:**
```markdown
## Task Sizing (10-Minute Rule)
Each microtask must be completable in 10 minutes:
- 2 minutes: Write failing test
- 5 minutes: Implement minimal solution
- 3 minutes: Verify and refactor

## Task Categories and Numbering
00a-00z: Foundation/Prerequisites (types, structs, basic setup)
01-09:   Core implementation methods (one method per task)
10-19:   Unit tests (grouped by functionality)
20-29:   Integration tests
30-39:   Error handling and validation
40-49:   Documentation and examples
50+:     Performance and optimization (only if needed)
```

**Doc-Planner Enforcement:**
```markdown
## MANDATORY ATOMIC TASK BREAKDOWN REQUIREMENT
CRITICAL: For EVERY phase documentation you create, you MUST include
a complete atomic task breakdown section with numbered tasks
(e.g., task_000 through task_099 or higher).

Each atomic task must:
- Take no more than 10-30 minutes to complete
- Have a specific, measurable outcome
- Follow the RED-GREEN-REFACTOR cycle
- Be independently verifiable
- Include clear dependencies

FAILURE TO INCLUDE ATOMIC TASK BREAKDOWNS = INCOMPLETE DOCUMENTATION
```

**Actual Task Example from Their System:**
```markdown
# Task 020: Implement Search Method

**Estimated Time: 8 minutes**

## Context
SearchEngine struct exists with index reader. Need to add search method
that accepts query string and returns results.

## Current System State
- SearchEngine struct defined in src/search/engine.rs
- Index reader initialized and verified working
- Query parser available but not integrated

## Your Task
Add search() method to SearchEngine that performs basic keyword search.

## Test First (RED Phase)
```rust
#[test]
fn test_search_returns_results() {
    let engine = SearchEngine::new("test_index");
    let results = engine.search("rust programming").unwrap();
    assert!(!results.is_empty());
}
```

## Minimal Implementation (GREEN Phase)
```rust
pub fn search(&self, query: &str) -> Result<Vec<Document>> {
    let searcher = self.reader.searcher();
    let query_obj = self.parser.parse_query(query)?;
    let top_docs = searcher.search(&query_obj, &TopDocs::with_limit(10))?;

    let results = top_docs.iter()
        .map(|(_, addr)| searcher.doc(*addr))
        .collect::<Result<Vec<_>>>()?;

    Ok(results.into_iter()
        .map(|doc| Document::from_tantivy(doc))
        .collect())
}
```

## Refactored Solution (REFACTOR Phase)
```rust
pub fn search(&self, query: &str) -> Result<Vec<Document>> {
    let searcher = self.reader.searcher();
    let parsed_query = self.parse_query(query)?;
    let scored_docs = self.execute_search(&searcher, &parsed_query)?;
    self.convert_to_documents(&searcher, scored_docs)
}

// Extracted helper (more testable)
fn execute_search(&self, searcher: &Searcher, query: &Query)
    -> Result<Vec<(f32, DocAddress)>> {
    searcher.search(query, &TopDocs::with_limit(10))
        .map_err(Into::into)
}
```

## Verification Commands
```bash
cargo test test_search_returns_results
cargo build
cargo clippy
```

## Success Criteria
- [x] Test written and fails with "method not found"
- [x] Implementation makes test pass
- [x] Code compiles without warnings
- [x] Real tantivy search (no mocks)
- [x] Returns actual documents from index

## Dependencies Confirmed
- tantivy = "0.21.0" in Cargo.toml
- Index created in previous task (task_019)
- Query parser initialized in SearchEngine

## Next Task
task_021: Add query syntax support (AND, OR, NOT operators)
```

## Applicability to Our Project

### Alignment with Our Goals
**STRONG ALIGNMENT** with adaptation needed:

**MINIMIZE** (Principle 1):
- ✅ **ALIGNS**: Small tasks minimize scope and complexity per unit
- ✅ Small tasks easier to load into context
- ✅ Clear boundaries reduce cognitive load
- ⚠️ But 100+ task files = filesystem overhead
- ⚠️ Breakdown process itself takes time

**SEPARATE** (Principle 2):
- ✅ **ALIGNS**: Each task is independent, verifiable unit
- ✅ Clear separation of concerns (one task = one thing)
- ✅ Dependencies explicitly stated
- ✅ Easy to parallelize across developers/agents

**VALIDATE** (Principle 3):
- ✅ **ALIGNS**: Every task has success criteria
- ✅ Verifiable completion (tests pass, builds work)
- ✅ Easy to measure progress (X/100 tasks complete)
- ⚠️ No evidence they validated 10-minute timing empirically
- ⚠️ No data on when breakdown overhead is worth it

**LEARN** (Principle 4):
- ✅ **ALIGNS**: Task templates create reusable patterns
- ✅ Actual time vs estimated provides learning
- ✅ Success criteria evolve with experience
- ⚠️ No evidence of adapting based on learnings

### Specific Ways This Applies
1. **Feature development**: Break multi-hour features into 10-minute tasks
2. **Onboarding**: New developers can pick up single tasks
3. **Progress tracking**: Count completed tasks for status updates
4. **Parallelization**: Multiple agents work on different tasks simultaneously
5. **Estimation**: 10 tasks = ~100 minutes, easier to predict
6. **Testing**: TDD structure ensures test coverage
7. **Documentation**: Each task documents its piece of system

### Integration Points
```javascript
// Our approach: Adaptive task sizing based on complexity

// Simple change (single task, no breakdown needed):
[Single Message]:
  // No formal task breakdown for <30min work
  Task("Add logout button", "Implement logout button with Firebase signOut", "coder")

// Medium feature (5-10 tasks):
[Single Message]:
  Read("skills/task-breakdown.skill")  // Lightweight version
  Task("Break down feature", "Create 5-8 tasks for password reset flow", "planner")

  // Tasks defined in memory, executed sequentially
  Task("task_01: Create reset UI", "...", "coder")
  Task("task_02: Wire Firebase", "...", "coder")
  Task("task_03: Add tests", "...", "tester")

// Complex feature (20+ tasks):
[Single Message]:
  Read("skills/planning.skill")  // Full breakdown
  Task("SPARC planning", "Create detailed phase plan with 20+ atomic tasks", "planner")

  // Create actual task files for tracking
  Write("tasks/phase1/task_000.md")
  Write("tasks/phase1/task_001.md")
  // ... etc
```

### What Would We Need to Change?
1. **Make breakdown optional**: Not every task needs formal 10-minute decomposition
2. **Adaptive sizing**: Simple (no breakdown), medium (in-memory tasks), complex (task files)
3. **Flexible timing**: 10-minute is guideline, not rigid constraint
4. **Reduce filesystem overhead**: Keep tasks in memory or single file for medium features
5. **Validate timing**: Experiment to see if 10-minute is optimal for all work types
6. **Allow creativity**: Some tasks (design, architecture) need exploration time

## Experiments & Validation

### Hypothesis to Test
"Breaking complex features (>2 hours) into 10-minute atomic tasks reduces total time and rework by 20-30% compared to informal breakdown"

### Experimental Design
**Independent Variable**: Task breakdown approach
- Control: Informal breakdown (developer's discretion)
- Treatment 1: Formal 10-minute atomic tasks (their approach)
- Treatment 2: Adaptive sizing (10-min for complex, informal for simple)

**Feature Complexity Levels**:
- Simple (<30 min): "Add field to form", "Fix validation bug"
- Medium (30min-2hr): "Implement pagination", "Add export to CSV"
- Complex (>2hr): "Build admin dashboard", "Real-time notifications"

**Metrics**:
- Breakdown time (upfront planning cost)
- Implementation time (actual coding)
- Total time (breakdown + implementation + rework)
- Rework needed (changes after first attempt)
- Quality (tests pass, requirements met)
- Developer experience (subjective rating)
- Actual task timing (vs 10-minute estimate)

**Sample Size**: 10 features per complexity level per treatment = 90 features

### Expected Results
**Simple Features**:
- Control: Fast, no overhead
- Treatment 1: Slower due to breakdown overhead (30-50% overhead)
- Treatment 2: Similar to control (skips breakdown)

**Medium Features**:
- Control: Some rework, moderate efficiency
- Treatment 1: Less rework but breakdown overhead
- Treatment 2: Best of both (light breakdown, no overhead)

**Complex Features**:
- Control: Significant rework, highest total time
- Treatment 1: Lower total time (breakdown pays off)
- Treatment 2: Similar to treatment 1

**Timing Accuracy**:
- Expect actual times to vary: some tasks 5min, others 20min
- Identify task types that consistently exceed 10min
- Find patterns in what makes tasks longer/shorter

**Decision Criteria**:
- If formal breakdown reduces total time on complex features by >20%: **ADOPT FOR COMPLEX**
- If overhead on simple features is >30%: **MAKE OPTIONAL**
- If adaptive performs within 10% of optimal: **ADOPT ADAPTIVE**
- If many tasks exceed 10min: **ADJUST GUIDELINE** (e.g., 15-minute target)

## Related Learnings
- [learning-mandatory-planning-agents.md](./learning-mandatory-planning-agents.md) - Planning agents that create these tasks
- [pattern-sparc-methodology.md](./pattern-sparc-methodology.md) - SPARC phases broken into tasks
- [learning-operation-batching.md](./learning-operation-batching.md) - Executing tasks in batches
- [mistake-over-complexity.md](./mistake-over-complexity.md) - When breakdown becomes overhead

## Our Project Application

### Recommendations
1. **ADAPT, DON'T MANDATE**: Atomic tasks are valuable but not for all work
2. **Create task-breakdown skill**: Lightweight guide (~50 lines) vs full agent
3. **Adaptive sizing**: Auto-suggest breakdown only for complex features
4. **Flexible timing**: 10-minute is guideline, allow 5-20 minute range
5. **Validate empirically**: Track actual times to refine estimates

### Implementation Strategy
**Phase 1**: Create lightweight skill (50 lines)
```
skills/task-breakdown.skill
├── When to Use
│   - Features estimated >2 hours
│   - Multiple developers/agents
│   - High-risk changes
│   - Uncertain complexity
├── Core Principles
│   - One task = one testable thing
│   - Target 10-15 minutes per task
│   - Include RED-GREEN-REFACTOR structure
│   - Clear success criteria
│   - Document dependencies
├── Template (simplified)
│   - Task: [Action]
│   - Test: [What to verify]
│   - Implement: [How to build]
│   - Verify: [Commands to run]
└── Numbering (optional)
    - Use if >10 tasks
    - Group by category (00-09: setup, 10-19: core, etc.)
```

**Phase 2**: Add to .cursorrules (3 lines)
```
# Task Decomposition
- For complex features (>2hr), break into 10-15 minute atomic tasks
- Each task: write test, implement minimally, verify
```

**Phase 3**: Experimental validation
```javascript
Experiment: "atomic-task-timing"
Setup:
  - Track actual task completion times
  - Compare estimated vs actual
  - Identify patterns in overruns
Measure:
  - Does 10-minute target hold?
  - What task types take longer?
  - When does breakdown pay off?
Refine:
  - Adjust timing guideline (maybe 15-minute is better)
  - Identify task types that don't fit mold
  - Create exceptions (research, creative work)
```

**Phase 4**: Create simple tracking
```markdown
# features/admin-dashboard/tasks.md

## Task Breakdown for Admin Dashboard
Estimated total: 20 tasks × 15min = 5 hours

### Setup (3 tasks)
- [ ] task_01: Create dashboard route (10min)
- [ ] task_02: Add layout component (15min)
- [ ] task_03: Setup data fetching (12min)

### Core Features (12 tasks)
- [ ] task_04: User list view (15min)
- [ ] task_05: User detail view (15min)
- ...

### Testing (5 tasks)
- [ ] task_16: Unit tests for components (20min)
- [ ] task_17: Integration test for data flow (15min)
- ...

## Progress: 5/20 complete (25%)
## Actual time so far: 78min (vs 75min estimated)
```

### Success Criteria
- ✅ Task breakdown skill created (<100 lines)
- ✅ Used on 3+ complex features with tracking
- ✅ Actual timing data collected (20+ tasks)
- ✅ Breakdown reduces rework by >15% on complex features
- ✅ No mandatory overhead on simple tasks
- ✅ Timing guideline refined based on data (10min? 15min? varies by type?)

## Key Takeaways
1. **Atomic tasks work**: Small, focused units reduce overwhelm and increase progress
2. **10-minute is aggressive**: May work for some tasks, too tight for others (15-20min more realistic)
3. **TDD structure is gold**: RED-GREEN-REFACTOR baked into every task is excellent
4. **Not all work is atomic**: Research, design, architecture need exploration time
5. **Breakdown has cost**: Overhead of planning must be worth it (validated for complex features only)
6. **Flexible > rigid**: Guideline of 10-15min better than strict 10min requirement
7. **Track actual times**: Real data informs better estimates and patterns
8. **Optional > mandatory**: Breakdown when it helps, skip when it doesn't

## Quality Score
**Concept Quality**: 90/100
- Excellent: Atomic sizing, TDD structure, clear success criteria
- Good: Numbering system, dependency tracking
- Minor: 10-minute may be too aggressive, doesn't fit all work types

**Implementation Quality**: 75/100
- Excellent: Detailed templates, clear methodology
- Good: Enforced through repetition
- Concerns: No empirical validation, mandatory approach, heavy overhead

**Production Readiness for Our Project**: 85/100
- Ready: Core concept is proven (atomic tasks with TDD)
- Adapt: Make optional, adjust timing to 10-15min range, lightweight skill not full agent
- Validate: Must track actual times to refine estimates
- High value when applied correctly to complex features
