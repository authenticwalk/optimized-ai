# Core Principles

These principles drive every architectural decision in the Optimized AI system.

## 1. MINIMIZE - Maximum Results, Minimum Overhead

**Philosophy**: Every token counts. Every instruction adds cognitive load and potential conflicts. Less is more.

### Token Efficiency
- **Small, tight prompts** - Every word justified by experiments
- **Load only what's needed** - Don't load instructions that won't be used
- **Compressed knowledge** - Store learnings efficiently
- **Avoid repetition** - Don't say the same thing multiple ways

### Instruction Optimization
- **Claude.md must be tiny** - Core principles only, < 200 lines
- **`.cursorrules` minimal** - Essential rules only, < 50 lines
- **Test every line** - Prove each instruction adds value
- **Remove conflicting rules** - More instructions = more conflicts

### Step Efficiency
- **Minimize tool calls** - Each call has overhead
- **Batch operations** - Do multiple things at once when possible
- **Direct solutions** - Avoid unnecessary intermediate steps
- **Learn optimal paths** - Record and reuse efficient approaches

### Why This Matters
```
Scenario: Add Firebase auth

BAD (bloated approach):
- 500-line .cursorrules with every possible rule
- AI reads all instructions even though most don't apply
- Conflicting rules about file organization, testing, etc.
- 50k tokens used, 20 minutes, lots of confusion
- Token cost: $$$

GOOD (minimized approach):
- 50-line core .cursorrules
- Load "firebase-auth" skill on-demand (adds specific guidance)
- Only relevant instructions in context
- 25k tokens used, 10 minutes, clear execution
- Token cost: $
```

### Experimental Validation Required
Every instruction must prove its value:
- Does removing it degrade results? (If no, remove it)
- Does it conflict with other instructions? (If yes, resolve or remove)
- Does it improve metrics? (If no, remove it)
- Is it used in this scenario? (If no, don't load it)

## 2. SEPARATION OF CONCERNS - Context Isolation

**Philosophy**: Different tasks need different context. Don't pollute context with irrelevant information.

### Skills - Load On-Demand
**Skills are contextual instructions loaded only when needed.**

```
Core context (always loaded):
- .cursorrules (50 lines - core principles only)
- claude.md (200 lines - project basics)

Skills (loaded on-demand):
- firebase-auth.skill - Loaded when working on auth
- supabase-rls.skill - Loaded when working on RLS
- testing.skill - Loaded when writing tests
- refactoring.skill - Loaded when refactoring
- performance.skill - Loaded when optimizing
```

**How Skills Work:**
```bash
# AI determines task needs auth
# Automatically loads firebase-auth.skill
# Skill adds 100 lines of specific auth guidance
# After task complete, skill unloaded

# Next task: optimize queries
# Loads performance.skill instead
# Different context for different task
```

**Benefits:**
- ✅ Only relevant instructions in context
- ✅ No conflicting instructions from unrelated domains
- ✅ Fewer tokens used
- ✅ Clearer focus on current task
- ✅ Can update skills independently

### Subagents - Fresh Context Windows
**Subagents are isolated AI instances for specific subtasks.**

```
Main Agent (PM/Orchestrator):
- Reads task
- Breaks into subtasks
- Delegates to subagents
- Integrates results

Subagent: Planner
- Fresh context
- Loads planning.skill
- Creates task plan
- Returns plan to main agent

Subagent: Implementer
- Fresh context
- Loads relevant implementation skills
- Writes code
- Returns code to main agent

Subagent: Reviewer
- Fresh context
- Loads review.skill
- Reviews code
- Returns feedback to main agent
```

**Why Subagents:**
- ✅ Each gets fresh context window
- ✅ No context pollution between tasks
- ✅ Parallel execution possible
- ✅ Specialized focus
- ✅ Clearer responsibilities
- ✅ Easier to debug (smaller contexts)

**When to Use Subagents:**
- Complex multi-step tasks
- Tasks requiring different skills/context
- When context is getting bloated
- When task has clear handoff points
- When parallelization possible

### MCP Skills Architecture

**Core MCP Server** (`@optimized-ai/mcp-core`):
```typescript
// Core operations (always available)
- getPattern(type)
- savePattern(type, data)
- getPreferences()
- recordSuccess(task)
- recordFailure(task)
```

**Skill MCPs** (loaded on-demand):
```typescript
// @optimized-ai/skill-firebase
- getFirebaseAuthPattern()
- getFirestoreQueryPattern()
- validateSecurityRules()
- debugFirebaseData()

// @optimized-ai/skill-supabase
- getSupabaseRLSPattern()
- getSupabaseQueryPattern()
- validateRLSPolicies()
- debugSupabaseData()

// @optimized-ai/skill-testing
- getTestPattern(framework)
- getEdgeCaseExamples()
- validateTestCoverage()

// @optimized-ai/skill-refactoring
- getRefactorPattern(type)
- detectDuplication()
- suggestExtraction()
```

### Skill Loading Strategy

**Automatic Detection:**
```
AI analyzes task:
"Implement Firebase authentication"

Detects keywords: "Firebase", "authentication"

Loads skills:
- @optimized-ai/skill-firebase
- @optimized-ai/skill-auth

Context now includes:
- Core rules (50 lines)
- Project basics (200 lines)
- Firebase patterns (100 lines)
- Auth patterns (100 lines)
Total: 450 lines vs 2000+ lines if all loaded
```

**Manual Override:**
```bash
# User can explicitly load skills
optimized-ai task "Implement auth" --skills firebase,auth,testing

# Or exclude skills
optimized-ai task "Refactor code" --no-skills testing
```

**Skill Combinations:**
```
Task: "Add authenticated Firebase query"
Skills: firebase + auth + security
Context optimized for this specific task
```

## 3. Experimental Validation - Prove Everything

**Every optimization must be validated empirically.**

### Validation for Minimize Principle

**Hypothesis**: "Reducing .cursorrules from 500 to 50 lines improves performance without degrading quality."

**Experiment:**
- Control: 500-line .cursorrules
- Treatment: 50-line .cursorrules
- Scenarios: 10 common tasks
- Metrics: tokens, time, quality, confusion rate

**Expected Results:**
- 40% fewer tokens
- 30% faster completion
- Equal or better quality
- No increase in errors

**If validated**: Adopt minimal approach
**If not validated**: Identify what's missing, refine

### Validation for Separation of Concerns

**Hypothesis**: "Loading skills on-demand vs loading all instructions reduces tokens by 60% without degrading quality."

**Experiment:**
- Control: All instructions loaded upfront
- Treatment: Skills loaded on-demand
- Scenarios: 10 tasks requiring different skills
- Metrics: tokens, time, quality, confusion, context pollution

**Expected Results:**
- 60% fewer tokens per task
- Clearer focus (qualitative)
- No quality degradation
- Fewer conflicting instructions

**If validated**: Adopt skill-based architecture
**If not validated**: Refine skill boundaries

### Validation for Subagents

**Hypothesis**: "Using subagents for complex tasks improves quality and reduces main context pollution."

**Experiment:**
- Control: Single agent handles entire task
- Treatment: Subagents for plan/implement/review
- Scenarios: 5 complex multi-step tasks
- Metrics: quality, context size, clarity, debugging ease

**Expected Results:**
- Higher quality code
- Clearer debugging
- Better separation
- No significant time increase

**If validated**: Use subagents for complex tasks
**If not validated**: Identify overhead issues

## 4. Learn and Iterate - Continuous Optimization

**The system optimizes itself over time.**

### What to Learn
- Which instructions are actually useful (remove unused)
- Which skills are commonly loaded together (merge?)
- Which patterns work best (promote to core)
- Which rules conflict (resolve or remove)
- Optimal skill granularity (split or merge)

### Metrics to Track
- **Instruction usage** - Which rules are referenced?
- **Skill load frequency** - Which skills used most?
- **Token efficiency** - Tokens per successful task
- **Step efficiency** - Tool calls per successful task
- **Quality consistency** - Success rate per configuration

### Optimization Loop
```
1. Track: Record what's loaded and used
2. Analyze: Identify patterns in usage
3. Optimize: Remove unused, merge common, split bloated
4. Validate: Test optimizations experimentally
5. Deploy: Update configs if validated
6. Repeat: Continuous improvement
```

### Example: Self-Optimizing .cursorrules

```
Initial .cursorrules (50 lines):
- Rule A: Use TypeScript strict mode
- Rule B: Separate concerns
- Rule C: No React/Tailwind
- Rule D: Minimal boilerplate
- Rule E: Test edge cases
... (50 rules total)

After 100 tasks, metrics show:
- Rule A: Referenced 95 times (KEEP - high value)
- Rule B: Referenced 80 times (KEEP - high value)
- Rule C: Referenced 2 times (KEEP - when violated, bad results)
- Rule D: Referenced 60 times (KEEP - medium value)
- Rule E: Referenced 10 times (CONSIDER - low usage)
- Rule F: Referenced 0 times (REMOVE - unused)
- Rule G: Conflicts with Rule H 5 times (RESOLVE)

Optimized .cursorrules (45 lines):
- Removed Rule F (never used)
- Resolved Rules G/H conflict
- Kept all high-value rules
- Result: 10% fewer tokens, same quality
```

## 5. Quality Over Speed - But Both Matter

**Fast execution with poor quality is worthless. High quality that takes forever is impractical.**

### Balance:
- Optimize for speed AFTER ensuring quality
- Never sacrifice correctness for speed
- But don't tolerate unnecessary slowness
- Measure both dimensions independently

### Quality Metrics (non-negotiable):
- Tests pass
- Linter clean
- Requirements met
- No security issues
- Maintainable code

### Speed Metrics (optimize while maintaining quality):
- Time to completion
- Token usage
- Tool call count
- Context window efficiency

## Architecture Implications

### Core System Components

**1. Minimal Core** (Always Loaded)
```
.cursorrules (50 lines max)
- Core principles
- Critical rules
- Project structure basics

claude.md (200 lines max)
- Project overview
- Tech stack
- Key conventions
```

**2. Skill Library** (Load on-demand)
```
skills/
├── firebase/
│   ├── auth.skill
│   ├── firestore.skill
│   └── functions.skill
├── supabase/
│   ├── rls.skill
│   ├── queries.skill
│   └── rpc.skill
├── patterns/
│   ├── testing.skill
│   ├── refactoring.skill
│   └── performance.skill
└── frameworks/
    ├── svelte.skill
    ├── htmx.skill
    └── ionic.skill
```

**3. Subagent Definitions**
```
agents/
├── planner.agent - Task breakdown
├── implementer.agent - Code writing
├── reviewer.agent - Quality check
├── tester.agent - Test creation
└── optimizer.agent - Performance tuning
```

**4. Knowledge Base** (Learned patterns)
```
.ai-knowledge/
├── patterns.json - Successful approaches
├── anti-patterns.json - What to avoid
├── skill-usage.json - Which skills used when
└── optimization-metrics.json - Performance data
```

### Execution Flow

**Simple Task:**
```
1. Load core (.cursorrules + claude.md) - 250 lines
2. Analyze task → determine needed skills
3. Load relevant skills (e.g., firebase.skill) - 100 lines
4. Total context: 350 lines
5. Execute task
6. Unload skills
```

**Complex Task:**
```
1. Load core - 250 lines
2. Main agent analyzes task
3. Create subagent: Planner
   - Fresh context
   - Load planning.skill
   - Create plan
   - Return to main
4. Create subagent: Implementer
   - Fresh context
   - Load implementation skills
   - Write code
   - Return to main
5. Create subagent: Reviewer
   - Fresh context
   - Load review.skill
   - Review code
   - Return feedback
6. Main agent integrates results
```

## Validation Requirements

### Every Instruction Must Prove Value
- Does it improve metrics?
- Is it actually used?
- Does it conflict with others?
- Can it be removed without degradation?

### Every Skill Must Justify Existence
- Does it serve a clear purpose?
- Is it loaded frequently enough?
- Does it improve task-specific performance?
- Is it the right granularity?

### Every Subagent Must Add Value
- Does it improve quality?
- Does it reduce context pollution?
- Is the overhead justified?
- Could it be merged or split?

## Success Metrics for Principles

### Minimize:
- ✅ Core config < 250 lines total
- ✅ 40%+ token reduction vs monolithic approach
- ✅ No quality degradation
- ✅ Faster execution

### Separation of Concerns:
- ✅ Skills loaded only when needed
- ✅ Clear skill boundaries
- ✅ No cross-skill conflicts
- ✅ Easy to add/remove skills

### Experimental Validation:
- ✅ Every optimization has data
- ✅ Failed experiments documented
- ✅ A/B testing proves improvements
- ✅ Can defend every choice with evidence

### Learn and Iterate:
- ✅ System improves over time
- ✅ Unused instructions removed
- ✅ Optimal skill loading learned
- ✅ Measurable efficiency gains

---

## Implementation Priority

1. **Build experimental framework FIRST** ✅
   - Cannot optimize without measurement
   - Must validate every decision

2. **Start minimal, add incrementally**
   - 50-line .cursorrules to start
   - Prove each addition adds value
   - Remove what doesn't help

3. **Skills before features**
   - Architecture for loading skills
   - Test skill loading performance
   - Validate separation benefits

4. **Subagents for complex tasks only**
   - Start with single agent
   - Add subagents when proven needed
   - Measure overhead vs benefit

5. **Continuous optimization**
   - Track usage continuously
   - Optimize based on data
   - Never stop improving

---

## Bottom Line

**MINIMIZE**: Every token, every instruction, every step must justify its existence through experimental validation.

**SEPARATE**: Different contexts for different tasks. Load only what's needed. Isolate concerns.

**VALIDATE**: Prove everything with data. No theoretical optimizations. Only measured, reproducible improvements.

**ITERATE**: Learn from usage. Remove what doesn't help. Optimize continuously.

This is not just a coding project. It's a scientific pursuit of maximum AI efficiency.

