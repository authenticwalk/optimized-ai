# Principle 2: SEPARATE

**Philosophy**: Different tasks need different context. Don't pollute context with irrelevant information.

## Core Concept

Context isolation through:
- **Skills** - Load instructions only when needed for specific tasks
- **Subagents** - Fresh context windows for complex tasks
- **Scripts** - Direct execution without middleware
- **No context pollution** - Keep each context focused

## Skills: On-Demand Loading

### What Are Skills?

Skills are focused instruction sets loaded only when relevant to the current task.

**Structure**:
```
skills/
├── firebase-auth.skill
├── supabase-rls.skill
├── testing.skill
├── refactoring.skill
└── performance.skill
```

**Skill Format**:
```yaml
name: firebase-auth
description: Firebase Authentication patterns
triggers:
  keywords: [firebase, auth, authentication, login, signup]
  files: [firebase.config.*, auth/*.ts, **/auth.ts]
content: |
  # Firebase auth-specific guidance (~100 lines)
  - Firebase initialization
  - Auth setup patterns
  - Error handling
  - Security best practices
  
scripts:
  check-auth: ./scripts/firebase-check-auth.sh
  test-connection: ./scripts/firebase-test.sh
```

### How Skills Work

**Automatic Detection**:
```
Task: "Implement Firebase authentication"

AI analyzes:
- Keywords: "Firebase", "authentication" → Load firebase-auth.skill
- Context: No testing mentioned → Don't load testing.skill

Result:
- Core: 80 lines
- firebase-auth.skill: 100 lines
- Total: 180 lines (vs 500+ if all loaded)
```

**Manual Override**:
```bash
# Explicitly load skills
optimized-ai task "Refactor auth" --skills firebase-auth,refactoring

# Explicitly exclude skills  
optimized-ai task "Fix typo" --no-skills
```

### Skills vs MCPs

**IMPORTANT**: Skills can call scripts directly. MCPs might be unnecessary middleware.

**Instead of MCP**:
```typescript
// OLD: MCP server for Firebase
mcp.call('firebase.queryData', { collection, query })

// NEW: Skill calls script directly
skill: firebase-auth
script: check-auth
action: ./scripts/firebase-check-auth.sh --collection users --query "active=true"
```

**Benefits**:
- ✅ No middleware overhead
- ✅ Direct command-line calls
- ✅ Simpler architecture
- ✅ Easy to test scripts independently
- ✅ No MCP protocol complexity

**When MCP Might Still Be Needed**:
- Complex state management
- Real-time data streaming
- Cross-tool coordination

**Default**: Try scripts first, only add MCP if proven necessary

## Subagents: Fresh Context Windows

### What Are Subagents?

Subagents are isolated AI instances with fresh contexts for specific subtasks.

**Use Cases**:
- Complex multi-step tasks
- Tasks requiring different expertise
- When main context is bloated
- Clear handoff points

**Example Flow**:
```
Main Agent (Orchestrator):
- Reads task: "Add user auth with tests and documentation"
- Breaks into subtasks
- Delegates to subagents

Subagent: Planner
- Fresh context (core + planning.skill)
- Creates detailed implementation plan
- Returns plan

Subagent: Implementer  
- Fresh context (core + firebase-auth.skill)
- Implements based on plan
- Returns code

Subagent: Tester
- Fresh context (core + testing.skill)
- Writes tests for implementation
- Returns test suite

Subagent: Documenter
- Fresh context (core + documentation.skill)
- Writes docs
- Returns documentation

Main Agent:
- Integrates all results
- Self-evaluates
- Creates PR
```

### Benefits of Subagents

- ✅ **No context pollution** - Each agent has clean, focused context
- ✅ **Parallel execution** - Multiple subagents can work simultaneously
- ✅ **Specialized focus** - Each agent optimized for its task
- ✅ **Easier debugging** - Smaller contexts, clearer failures
- ✅ **Scalable** - Add new agents without affecting existing ones

### When NOT to Use Subagents

- Simple, single-file changes
- Tasks that need shared context
- When handoff overhead > benefit

**Rule of Thumb**: Use subagents for tasks with 3+ distinct phases

## Direct Script Execution

### Philosophy

**Cut out the middleman**. If you can do it with a script, do it with a script.

### Command-Line Tools

**Firebase CLI**:
```bash
# Instead of: MCP wrapper around Firebase
# Just use: Firebase CLI directly

firebase firestore:get users/user123
firebase auth:export auth-users.json
firebase functions:log functionName
```

**Supabase CLI**:
```bash
supabase db dump
supabase functions list
supabase db reset
```

**TypeScript Execution**:
```bash
# For one-off scripts
ts-node scripts/analyze-data.ts

# Or use tsx (faster)
tsx scripts/analyze-data.ts
```

### Skill Script Integration

**In Skill Definition**:
```yaml
name: firebase-debug
scripts:
  check-connection: |
    firebase projects:list
  query-users: |
    firebase firestore:get users --limit 10
  check-rules: |
    firebase deploy --only firestore:rules --dry-run
```

**AI Usage**:
```
Task: "Debug Firebase auth issue"

AI loads firebase-debug skill
AI executes: firebase auth:export auth-dump.json
AI reads output
AI analyzes issue
AI fixes code
```

**No MCP needed!**

## Skill Loading Strategy

### Automatic Detection Algorithm

```typescript
function detectSkills(task: string, files: string[]): string[] {
  const skills: Set<string> = new Set();
  
  // Keyword detection
  const keywords = extractKeywords(task);
  for (const skill of availableSkills) {
    if (skill.triggers.keywords.some(kw => keywords.includes(kw))) {
      skills.add(skill.name);
    }
  }
  
  // File pattern detection
  for (const file of files) {
    for (const skill of availableSkills) {
      if (matchesPattern(file, skill.triggers.files)) {
        skills.add(skill.name);
      }
    }
  }
  
  // Historical pattern detection
  const similarTasks = findSimilarTasks(task);
  for (const pastTask of similarTasks) {
    skills.add(...pastTask.skillsUsed);
  }
  
  return Array.from(skills);
}
```

### Skill Combinations

Track which skills are commonly loaded together:

```json
{
  "firebase-auth + testing": { 
    "frequency": 45,
    "avgSuccess": 0.92
  },
  "firebase-auth + security": {
    "frequency": 30,
    "avgSuccess": 0.88
  },
  "refactoring + testing": {
    "frequency": 60,
    "avgSuccess": 0.95
  }
}
```

**Learning**: If firebase-auth loaded, suggest testing skill (high correlation)

## Edge Cases & Learnings

### Skill Too Broad
**Problem**: firebase.skill contained everything Firebase-related (auth, firestore, functions, hosting)

**Issue**: 
- Skill was 500 lines (defeats purpose)
- Loading unnecessary context
- Conflicts within skill

**Solution**: Split into focused skills
- firebase-auth.skill (100 lines)
- firebase-firestore.skill (100 lines)
- firebase-functions.skill (100 lines)
- firebase-hosting.skill (80 lines)

**Learning**: Skills should be single-purpose, ~100 lines

**Storage**: `.plan/initial-design/learnings/separate-skill-granularity.md`

### Skill Too Narrow
**Problem**: Created very specific skills (firebase-email-auth.skill, firebase-google-auth.skill)

**Issue**:
- Too many skills to manage
- Loading overhead
- Duplication between skills

**Solution**: Merge into firebase-auth.skill with sections
- Email/password auth
- Social auth (Google, Facebook, etc.)
- Custom auth
- Common patterns

**Learning**: Balance specificity with usability. ~100 lines is good target.

**Storage**: `.plan/initial-design/learnings/separate-too-narrow.md`

### Subagent Overhead
**Problem**: Used subagents for simple 3-line change

**Issue**:
- Subagent creation overhead > task time
- Added complexity for simple task
- No benefit from fresh context

**Solution**: Use subagents only for complex tasks (3+ distinct phases)

**Learning**: Don't over-architect. Simple tasks don't need subagents.

**Storage**: `.plan/initial-design/learnings/separate-subagent-overhead.md`

### MCP Was Unnecessary
**Problem**: Built MCP server to query Firebase

**Issue**:
- Added complexity
- Maintenance burden
- Firebase CLI already exists!

**Solution**: Skill just calls `firebase` CLI directly

**Learning**: Try direct CLI first. Only add MCP if actually needed.

**Storage**: `.plan/initial-design/learnings/separate-no-mcp-needed.md`

## Validation Experiments

### Exp-004: Skills vs Monolithic
**Hypothesis**: "On-demand skills reduce tokens 60% without quality loss"

**Design**:
- Control: 500-line monolithic config (all guidance)
- Treatment: 80-line core + on-demand skills
- Scenarios: 10 tasks requiring different skills
- Runs: 10 per scenario

**Metrics**: Tokens, quality, confusion rate

**Expected**: 60% token reduction, same quality

**Status**: After Phase 2

### Exp-005: Skill Granularity
**Hypothesis**: "~100 lines per skill is optimal"

**Design**:
- Test skills at different sizes (50, 100, 200, 400 lines)
- Measure loading overhead vs benefit
- Identify optimal granularity

**Status**: After Phase 2

### Exp-006: Subagents for Complex Tasks
**Hypothesis**: "Subagents improve quality for complex tasks"

**Design**:
- Control: Single agent handles complex task
- Treatment: Multiple subagents with fresh contexts
- Scenarios: 5 complex multi-phase tasks

**Metrics**: Quality, context size, debugging ease

**Status**: After Phase 3

### Exp-007: Direct Scripts vs MCP
**Hypothesis**: "Direct CLI calls simpler and faster than MCP"

**Design**:
- Control: MCP server for Firebase operations
- Treatment: Direct `firebase` CLI calls
- Measure: Complexity, speed, maintainability

**Expected**: Scripts are simpler and sufficient

**Status**: Phase 2 (validate before building MCPs)

## Implementation Notes

### Skill File Structure
```
skills/
├── firebase/
│   ├── auth.skill
│   ├── firestore.skill
│   └── functions.skill
├── supabase/
│   ├── rls.skill
│   └── queries.skill
├── patterns/
│   ├── testing.skill
│   ├── refactoring.skill
│   └── performance.skill
└── frameworks/
    ├── svelte.skill
    └── htmx.skill
```

### Script Directory
```
scripts/
├── firebase/
│   ├── check-auth.sh
│   ├── query-data.sh
│   └── validate-rules.sh
├── supabase/
│   ├── check-rls.sh
│   └── test-connection.sh
└── utils/
    ├── analyze-code.sh
    └── check-dependencies.sh
```

### Subagent Definitions
```
agents/
├── planner.agent.json
├── implementer.agent.json
├── reviewer.agent.json
└── tester.agent.json
```

## Success Metrics

- ✅ Skills loaded only when needed
- ✅ 60%+ token reduction with skills
- ✅ Clear skill boundaries
- ✅ No cross-skill conflicts
- ✅ Subagents improve complex task quality
- ✅ Direct scripts preferred over MCPs

## Related Principles

- [Principle 1: MINIMIZE](1-minimize.md) - Token efficiency
- [Principle 3: VALIDATE](3-validate.md) - Prove it works
- [Principle 4: LEARN](4-learn.md) - Optimize based on usage

## Learnings Repository

- `.plan/initial-design/learnings/separate-*.md`
- `.plan/initial-design/experiments/exp-00*.json`

## Future Work

### Dynamic Skill Loading
- Predict skills needed based on task analysis
- Load skills progressively as needed
- Unload skills when no longer relevant

### Skill Composition
- Combine multiple skills intelligently
- Resolve conflicts between skills
- Optimize skill loading order

### Smart Subagent Delegation
- Auto-detect when subagents beneficial
- Optimize subagent context size
- Learn best subagent strategies per task type

## References

- [CORE-PRINCIPLES.md](../CORE-PRINCIPLES.md)
- [EXPERIMENTAL-VALIDATION.md](../EXPERIMENTAL-VALIDATION.md)
- [REVISED-ROADMAP.md](../REVISED-ROADMAP.md)

