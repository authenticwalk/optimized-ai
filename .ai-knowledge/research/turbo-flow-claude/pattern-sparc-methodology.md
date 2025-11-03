# Pattern: SPARC Methodology (Specification, Pseudocode, Architecture, Refinement, Completion)

## Discovery Context
Found in SPARC_Methodology_Example.md (895 lines), doc-planner.md (lines 70-100), and CLAUDE.md as the core development workflow.
SPARC is presented as a systematic 5-phase approach with dedicated claude-flow commands for execution.
The methodology combines traditional SDLC phases with TDD and iterative refinement.

## The Core Insight
**"FIVE-PHASE STRUCTURED DEVELOPMENT: SPEC → PSEUDO → ARCH → REFINE → COMPLETE"** - Systematic progression from requirements through deployment with clear gates between phases.

### What They Do
```bash
# SPARC workflow commands available in claude-flow

# Individual phase execution
npx claude-flow@alpha sparc run spec-pseudocode "Task description"
npx claude-flow@alpha sparc run architect "Design system architecture"
npx claude-flow@alpha sparc tdd "Implement with test-driven development"
npx claude-flow@alpha sparc run integration "Complete integration and deployment"

# Complete pipeline
npx claude-flow@alpha sparc pipeline "Full project name"

# Parallel execution
npx claude-flow@alpha sparc batch spec-pseudocode,architect "Task"
```

**The Five Phases:**

### 1. SPECIFICATION Phase
```markdown
## Requirements Analysis
- **Functional Requirements**
  - User stories with acceptance criteria
  - Feature list with priorities
  - Use cases and workflows

- **Technical Requirements**
  - Architecture style (SPA, API, microservices)
  - Tech stack decisions
  - Non-functional requirements (performance, security, scalability)

- **Success Criteria**
  - Measurable completion indicators
  - Quality gates
  - Acceptance tests
```

### 2. PSEUDOCODE Phase
```markdown
## Algorithm Design

### Business Logic
```
FUNCTION clock_in(employee_id):
    current_time = get_current_timestamp()
    check IF employee already clocked in today:
        RETURN error "Already clocked in"

    CREATE attendance_record:
        employee_id = employee_id
        clock_in_time = current_time
        status = "active"

    SAVE to database
    RETURN success with record_id
```

### Data Flow
- Input validation
- Processing steps
- Output formatting
- Error handling paths
```

### 3. ARCHITECTURE Phase
```markdown
## System Design

### Architecture Diagram
┌─────────────────────────────────────┐
│         Client Layer                │
│  ┌──────────┐  ┌──────────┐        │
│  │ Employee │  │  Admin   │        │
│  │Dashboard │  │ Dashboard│        │
│  └──────────┘  └──────────┘        │
└─────────────────────────────────────┘
              ↓ HTTPS
┌─────────────────────────────────────┐
│    API Gateway / Load Balancer      │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Application Layer              │
│  ┌────────┐  ┌────────┐            │
│  │  Auth  │  │ Business│            │
│  │Service │  │ Logic   │            │
│  └────────┘  └────────┘            │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Data Layer                  │
│  ┌──────────┐  ┌──────────┐        │
│  │PostgreSQL│  │  Redis   │        │
│  └──────────┘  └──────────┘        │
└─────────────────────────────────────┘

### Database Schema
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

### API Endpoints
POST   /api/auth/login
GET    /api/attendance/status
POST   /api/attendance/clock-in
```

### 4. REFINEMENT Phase (TDD Implementation)
```markdown
## Test-Driven Development

### Test Structure
/tests
  /unit
    - attendance.service.test.js
    - auth.service.test.js
  /integration
    - api.test.js
  /e2e
    - workflows.test.js

### Sample Test
describe('Attendance Service', () => {
  test('should allow employee to clock in', async () => {
    const result = await attendanceService.clockIn(employeeId);
    expect(result.success).toBe(true);
    expect(result.recordId).toBeDefined();
  });

  test('should prevent duplicate clock-in', async () => {
    await attendanceService.clockIn(employeeId);
    await expect(attendanceService.clockIn(employeeId))
      .rejects.toThrow('Already clocked in');
  });
});

### Implementation Order (Sprints)
Sprint 1: Foundation (Week 1-2)
  - Setup project structure
  - Configure database
  - Implement authentication

Sprint 2: Core Features (Week 3-4)
  - Attendance module
  - Employee dashboard

Sprint 3: Advanced Features (Week 5-6)
  - Admin reporting
  - Export functionality
```

### 5. COMPLETION Phase
```markdown
## Integration & Deployment

### Technology Stack
{
  "frontend": "React 18",
  "backend": "Node.js + Express",
  "database": "PostgreSQL 15",
  "deployment": "Docker + Nginx"
}

### Environment Configuration
NODE_ENV=production
DATABASE_URL=postgresql://...
JWT_SECRET=...

### Build & Deploy
# Build
npm run build

# Docker
docker-compose -f docker-compose.prod.yml up -d

# Monitoring
pm2 logs attendance-api
```

## Deep Analysis

### Why This Approach Was Taken
1. **Systematic progression**: Each phase builds on previous, reducing rework
2. **Clear milestones**: Easy to track progress through phases
3. **Risk reduction**: Issues caught early in spec/design vs late in implementation
4. **Documentation**: Each phase produces artifacts for future reference
5. **Team coordination**: Clear handoffs between phases in team environment
6. **TDD integration**: Testing built into refinement phase, not bolted on
7. **Deployment readiness**: Completion phase ensures production viability

### What Problem It Solves
- **Scope creep**: Specification phase locks down requirements
- **Architecture drift**: Architecture phase provides blueprint to follow
- **Rushed testing**: TDD in refinement phase ensures coverage
- **Deployment surprises**: Completion phase validates production readiness
- **Knowledge silos**: Documentation artifacts preserve decisions
- **Rework cycles**: Early validation reduces late-stage changes
- **Communication gaps**: Phases provide shared vocabulary and checkpoints

### How It Compares to Alternatives
**Waterfall**:
- Similar: Sequential phases
- Different: SPARC has shorter cycles, includes TDD, more iterative within phases
- Pro: More flexible than traditional waterfall
- Con: Still sequential overhead

**Agile/Scrum**:
- Similar: Iterative development, working software emphasis
- Different: SPARC more structured, explicit design phase
- Pro: SPARC better for complex architecture
- Con: Agile faster for simple features

**Lean/Kanban**:
- Similar: Continuous flow
- Different: SPARC has distinct phase gates
- Pro: SPARC better documentation
- Con: Lean faster time-to-market

**Just Code It**:
- Similar: None (opposite approaches)
- Different: Everything (unstructured vs highly structured)
- Pro: SPARC catches issues earlier
- Con: Just coding is faster for prototypes

### Edge Cases and Considerations
1. **Small changes**: Full SPARC overkill for bug fixes or minor tweaks
2. **Exploratory work**: Research/prototyping doesn't fit phase structure
3. **Changing requirements**: Rigid phases struggle with pivots
4. **Phase parallelization**: Some phases can overlap (arch + pseudocode)
5. **Premature optimization**: Easy to over-design in architecture phase
6. **Documentation overhead**: 5 phases = lots of docs to maintain
7. **Team size**: SPARC valuable for teams, heavy for solo developers
8. **Project maturity**: New projects benefit most, mature projects less

## Implementation Details

### Examples from Their Codebase

**SPARC Commands Integration:**
```bash
# Complete workflow for new feature
npx claude-flow@alpha sparc run spec-pseudocode "Employee Attendance System"
# → Generates requirements.md, user-stories.md, algorithms.md

npx claude-flow@alpha sparc run architect "Design attendance system"
# → Generates architecture.md, database-schema.sql, api-spec.md

npx claude-flow@alpha sparc tdd "Implement attendance module"
# → Runs TDD workflow: write test, implement, refactor, repeat

npx claude-flow@alpha sparc run integration "Complete attendance system"
# → Deploys, configures, validates production readiness

# Or run full pipeline
npx claude-flow@alpha sparc pipeline "Employee Attendance System"
# → Executes all phases sequentially with gates between
```

**Real Example from SPARC_Methodology_Example.md:**
```markdown
# Phase 2: Pseudocode (Algorithm Design)

## 2.1 Attendance System Logic
FUNCTION clock_in(employee_id):
    current_time = get_current_timestamp()
    check IF employee already clocked in today:
        RETURN error "Already clocked in"

    CREATE attendance_record:
        employee_id = employee_id
        clock_in_time = current_time
        status = "active"

    SAVE to database
    RETURN success with record_id

## 2.4 SPARC Command
```bash
npx claude-flow sparc run spec-pseudocode "Design algorithms for
attendance tracking, fleet management, and reporting"
```

# Phase 3: Architecture (System Design)

## 3.2 Database Schema
CREATE TABLE attendance (
    attendance_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    clock_in_time TIMESTAMP NOT NULL,
    clock_out_time TIMESTAMP,
    total_hours DECIMAL(5,2),
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

## 3.4 SPARC Command
```bash
npx claude-flow sparc run architect "Design system architecture,
database schema, and API structure for attendance platform"
```

# Phase 4: Refinement (TDD Implementation)

## 4.1 Test-Driven Development Setup
describe('Attendance Service', () => {
  test('should allow employee to clock in', async () => {
    const result = await attendanceService.clockIn(employeeId);
    expect(result.success).toBe(true);
  });

  test('should prevent duplicate clock-in', async () => {
    await attendanceService.clockIn(employeeId);
    await expect(attendanceService.clockIn(employeeId))
      .rejects.toThrow('Already clocked in');
  });
});

## 4.3 SPARC Commands
```bash
npx claude-flow sparc tdd "Employee Attendance & Fleet Management Platform"
```
```

**Doc-Planner Integration:**
```markdown
## SPARC Workflow Implementation

Your primary purpose is to analyze codebases to create meticulously
organized documentation plans that:

1. **Specification Phase**
   - Define clear, unambiguous requirements
   - Create formal specifications with input/output contracts
   - Document invariants, preconditions, postconditions

2. **Pseudocode Phase**
   - Write high-level algorithmic descriptions
   - Create flow diagrams and state machines
   - Focus on logic clarity over syntax

3. **Architecture Phase**
   - Design system structure and component relationships
   - Define interfaces and boundaries
   - Document architectural decisions and trade-offs

4. **Refinement Phase**
   - Transform pseudocode into implementation-ready specifications
   - Add implementation details progressively
   - Validate against original specifications

5. **Completion Phase**
   - Verify all specifications are met
   - Ensure comprehensive test coverage
   - Create deployment and maintenance guides
```

## Applicability to Our Project

### Alignment with Our Goals
**PARTIAL ALIGNMENT** with significant concerns:

**MINIMIZE** (Principle 1):
- ❌ **CONFLICTS**: Five-phase process is heavyweight
- ❌ SPARC_Methodology_Example.md is 895 lines
- ❌ Each phase generates multiple documentation files
- ❌ Overhead is high even for medium features
- ✅ But systematic approach reduces rework (if overhead is worth it)

**SEPARATE** (Principle 2):
- ✅ **ALIGNS**: Clear separation of phases and concerns
- ✅ Each phase has distinct inputs/outputs
- ✅ Can hand off between team members at phase boundaries
- ✅ Artifacts separated by phase

**VALIDATE** (Principle 3):
- ⚠️ **UNKNOWN**: No evidence of experimental validation
- ⚠️ No data on when SPARC overhead pays off
- ⚠️ No metrics on phase time distribution
- ✅ But built-in validation at each phase gate

**LEARN** (Principle 4):
- ✅ **ALIGNS**: Documentation creates learning artifacts
- ✅ Each phase documents decisions and reasoning
- ✅ Reusable patterns emerge from repeated SPARC cycles
- ❌ But no evidence of adapting process based on learnings

### Specific Ways This Applies
1. **Complex features**: Multi-week features benefit from systematic approach
2. **Team coordination**: Clear phases help multiple developers/agents
3. **High-risk changes**: Architecture refactors warrant upfront design
4. **New domains**: Unfamiliar territory benefits from spec phase
5. **Documentation needs**: Regulated or complex systems need artifacts
6. **Onboarding**: New team members can follow phase structure

### Integration Points
```javascript
// Adaptive SPARC: Full vs Lite vs Skip based on complexity

// Simple change (Skip SPARC entirely):
[Single Message]:
  Task("Add field", "Add email field to user form", "coder")

// Medium feature (SPARC Lite - 2 phases):
[Single Message]:
  Read("skills/sparc-lite.skill")
  Task("Spec + Arch", "Define requirements and high-level design", "architect")
  Task("TDD Implementation", "Build with tests", "coder")

// Complex feature (Full SPARC - 5 phases):
[Single Message]:
  Read("skills/sparc-full.skill")
  Task("Phase 1: Specification", "Requirements and user stories", "planner")
  Task("Phase 2: Pseudocode", "Algorithm design", "planner")
  Task("Phase 3: Architecture", "System design and schema", "architect")
  Task("Phase 4: Refinement/TDD", "Implement with tests", "coder")
  Task("Phase 5: Completion", "Deploy and validate", "devops")
```

### What Would We Need to Change?
1. **Make SPARC optional**: Not mandatory for all work
2. **Create SPARC Lite**: 2-phase version (Spec+Arch → TDD)
3. **Adaptive selection**: Auto-suggest full SPARC only for complex projects
4. **Condense documentation**: 895-line example → ~200 lines for reference
5. **Validate experimentally**: Measure when SPARC overhead is worth it
6. **Allow phase overlap**: Some phases can run in parallel
7. **Skip for prototypes**: Exploratory work doesn't fit SPARC

## Experiments & Validation

### Hypothesis to Test
"Full 5-phase SPARC reduces total time and defects by >25% for complex features (>1 week) but adds >50% overhead for simple features (<1 day)"

### Experimental Design
**Independent Variable**: Development methodology
- Control: Informal approach (code with rough plan)
- Treatment 1: SPARC Lite (Spec+Arch → TDD)
- Treatment 2: Full SPARC (all 5 phases)
- Treatment 3: Adaptive SPARC (auto-select based on complexity)

**Feature Complexity**:
- Simple (<8 hours): "Add pagination", "Export to CSV"
- Medium (1-3 days): "User dashboard", "Email notifications"
- Complex (1+ week): "Admin panel", "Real-time chat", "Multi-tenant architecture"

**Metrics**:
- Planning time (upfront overhead)
- Implementation time (coding + testing)
- Rework time (fixes after initial implementation)
- Total time (planning + implementation + rework)
- Defect count (bugs found in testing/production)
- Documentation quality (completeness, usefulness)
- Developer satisfaction (subjective rating)

**Sample Size**: 8 features per complexity level per treatment = 96 features

### Expected Results
**Simple Features**:
- Control: Fastest total time, acceptable quality
- SPARC Lite: 30-50% overhead, minimal benefit
- Full SPARC: 100%+ overhead, no benefit
- Adaptive: Similar to control (skips SPARC)

**Medium Features**:
- Control: Moderate rework, decent efficiency
- SPARC Lite: Competitive total time, better quality
- Full SPARC: Some overhead but less rework
- Adaptive: Similar to SPARC Lite

**Complex Features**:
- Control: High rework, longest total time, most defects
- SPARC Lite: Better than control, still some rework
- Full SPARC: Best quality, competitive total time
- Adaptive: Similar to Full SPARC

**Phase Time Distribution**:
- Expect: Spec (15%), Pseudo (10%), Arch (20%), Refine/TDD (45%), Complete (10%)
- Reality: Likely Refine/TDD takes 60%+, other phases compressed

**Decision Criteria**:
- If Full SPARC reduces defects by >25% on complex features: **ADOPT FOR COMPLEX**
- If overhead on simple features is >50%: **MAKE OPTIONAL**
- If SPARC Lite performs within 10% of Full on medium features: **USE SPARC LITE AS DEFAULT**
- If adaptive correctly chooses approach >80% of time: **ADOPT ADAPTIVE**

## Related Learnings
- [learning-mandatory-planning-agents.md](./learning-mandatory-planning-agents.md) - Agents that implement SPARC
- [pattern-atomic-task-breakdown.md](./pattern-atomic-task-breakdown.md) - Tasks within SPARC phases
- [learning-operation-batching.md](./learning-operation-batching.md) - Executing SPARC phases efficiently
- [mistake-over-complexity.md](./mistake-over-complexity.md) - When SPARC becomes overhead

## Our Project Application

### Recommendations
1. **DON'T ADOPT FULL SPARC AS DEFAULT**: Too heavyweight for most work
2. **Create SPARC Lite skill**: 2-phase version for common use (~100 lines)
3. **Reserve Full SPARC**: Only for complex, high-risk, team-based features
4. **Validate thresholds**: Experiment to find when SPARC pays off
5. **Allow flexibility**: Phases can overlap or be skipped based on needs

### Implementation Strategy
**Phase 1**: Create SPARC Lite skill
```
skills/sparc-lite.skill (~100 lines)
├── When to Use
│   - Features estimated 1-3 days
│   - New architectural patterns
│   - Team coordination needed
├── Two Phases
│   1. Spec + Architecture (combined)
│      - Requirements (user stories)
│      - High-level design
│      - API contracts
│      - Database schema
│   2. TDD Implementation
│      - Write tests
│      - Implement
│      - Refactor
│      - Integrate
└── Artifacts
    - requirements.md
    - architecture.md
    - tests/
```

**Phase 2**: Create Full SPARC reference
```
.ai-knowledge/reference/sparc-full.md (~200 lines)
├── When to Use
│   - Complex features (>1 week)
│   - High-risk changes
│   - Regulatory requirements
│   - Large team coordination
├── Five Phases (detailed)
│   1. Specification
│   2. Pseudocode
│   3. Architecture
│   4. Refinement (TDD)
│   5. Completion (Deploy)
└── Examples
    - Attendance system (condensed)
    - API design
    - Database migration
```

**Phase 3**: Add to .cursorrules (3 lines)
```
# SPARC Methodology
- For complex features (>1 week), consider skills/sparc-lite.skill or full SPARC
- Always include architecture thinking for new patterns
```

**Phase 4**: Experimental validation
```javascript
Experiment: "sparc-threshold"
Setup:
  - Run features with Control, SPARC Lite, Full SPARC
  - Track time, rework, defects, satisfaction
Measure:
  - When does SPARC overhead pay off?
  - Is SPARC Lite sufficient for most medium features?
  - What percentage of features benefit from Full SPARC?
Refine:
  - Adjust complexity thresholds
  - Identify feature types that always/never need SPARC
  - Optimize phase structure based on time distribution
```

### Success Criteria
- ✅ SPARC Lite skill created (<150 lines)
- ✅ Full SPARC reference documented (<250 lines)
- ✅ Experimental validation on 10+ features
- ✅ SPARC Lite reduces rework on medium features by >15%
- ✅ Full SPARC reserved for <10% of features (the truly complex ones)
- ✅ No mandatory overhead on simple tasks

## Key Takeaways
1. **SPARC has value for complexity**: Systematic approach prevents chaos on large features
2. **Not for everything**: Simple and medium features don't need 5 phases
3. **SPARC Lite is sweet spot**: Spec+Arch → TDD covers most needs
4. **Phase overlap is OK**: Don't be dogmatic about sequential execution
5. **Documentation is overhead**: Only create artifacts that will be used
6. **TDD is the key phase**: Refinement/TDD is where value is created
7. **Validate before adopting**: Must prove overhead is worth it
8. **Adaptive is ideal**: Auto-select methodology based on complexity

## Quality Score
**Concept Quality**: 80/100
- Excellent: Systematic approach, clear phases, comprehensive coverage
- Good: TDD integration, documentation artifacts
- Concerns: Heavy overhead, rigid phases, no validation of benefit

**Implementation Quality**: 70/100
- Excellent: Detailed examples, command integration
- Good: Clear phase definitions
- Concerns: 895-line example, no adaptation, mandatory approach

**Production Readiness for Our Project**: 65/100
- Ready: Core concepts (phases, TDD, architecture thinking) are valuable
- Adapt: Need SPARC Lite version, make optional, validate thresholds
- Concerns: Full SPARC too heavy for most work, documentation overhead
- Validate: Must prove when systematic approach beats informal development
