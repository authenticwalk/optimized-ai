# Deep Dive: Anthropic's Claude Code Best Practices

## Source

**Blog Post**: https://www.anthropic.com/engineering/claude-code-best-practices
**Shared by**: Andreas Horn on LinkedIn
**Date**: Recent (referenced in 2025 posts)

## Why This Matters

This is **official guidance from Anthropic** on building effective AI agents. Unlike community projects, this represents the creators' recommended patterns.

## 7 Key Insights for Building Better AI Agents

### 1. Agent Design ≠ Prompting

**The Principle**:
Structure workflows enabling reasoning, action, reflection, retry, and escalation—not just clever prompts.

**Why It Matters**:
- Prompts alone are insufficient for complex tasks
- Need structured processes, not just instructions
- System architecture > prompt engineering

**Alignment with Our Project**:
✅ **VALIDATES our approach** - We focus on system design (skills, subagents, evaluation loops)
✅ **Not just .cursorrules** - We have workspace management, spin detection, self-evaluation

**Application**:
- Don't over-invest in prompt optimization alone
- Build systems that enable agent capabilities
- Workflow > words

**Our Implementation**:
```
✅ Reasoning: Planner agent creates approach
✅ Action: Implementer agent writes code
✅ Reflection: Reviewer agent self-evaluates
✅ Retry: Spin detection triggers alternative approaches
✅ Escalation: After 2-3 attempts, document blocker and ask user
```

### 2. Memory as Architecture

**The Principle**:
Manage context through summaries, structured files, and scoped retrieval rather than dumping full files into prompts.

**Why It Matters**:
- Context window is precious
- Full file dumps waste tokens
- Structured retrieval is more efficient

**Alignment with Our Project**:
✅ **VALIDATES** our `.ai-knowledge/` structured storage
✅ **VALIDATES** our skills-based on-demand loading
✅ **VALIDATES** our 40% token reduction goal

**Application**:
- Store learnings in structured format (SQLite + JSON)
- Retrieve only relevant patterns for current task
- Summarize past work instead of loading full context

**Our Implementation**:
```
.ai-knowledge/
├── knowledge.db (structured, queryable)
├── patterns.json (successful approaches)
├── failures.json (what didn't work)
├── preferences.json (user coding style)
└── plans/ (task history, not full code)
```

### 3. Planning is Essential

**The Principle**:
Implement explicit processes like plan > execute > review for multi-step problems across all models.

**Why It Matters**:
- Complex tasks need decomposition
- Ad-hoc execution leads to spinning
- All models benefit from planning

**Alignment with Our Project**:
✅ **VALIDATES** our planner agent
✅ **VALIDATES** `.plan/` folder approach
✅ **SUPPORTS** our multi-phase task breakdown

**Application**:
- Before coding, create explicit plan
- Break complex tasks into steps
- Review plan before execution

**Our Implementation**:
```
.plan/
├── current-task.md (what to build)
├── approach.md (how to build it)
├── progress.md (what's been done)
└── blockers.md (current issues)
```

Plus planner agent that:
1. Reads task
2. Checks `.ai-knowledge/` for similar tasks
3. Creates step-by-step plan
4. Identifies risks and edge cases

### 4. Real-World Tools Required

**The Principle**:
Agents need shell access, Git integration, and APIs to execute tasks, not merely explain them.

**Why It Matters**:
- Explanation ≠ execution
- Need to actually run tests, use Git, call APIs
- Read-only agents are limited

**Alignment with Our Project**:
✅ **VALIDATES** our emphasis on IDE integration
✅ **VALIDATES** our MCP servers (when needed)
✅ **SUPPORTS** ADR-003's script-based approach

**Application**:
- Give agent bash access
- Integrate with Git
- Provide API access (Firebase CLI, Supabase CLI)
- IDE operations (refactoring, testing)

**Our Implementation**:
```
Skills can call:
- Bash scripts
- Firebase CLI
- Supabase CLI
- TypeScript via tsx
- Git commands
- IDE operations (via future VSCode extension)
```

### 5. ReAct and CoT as System Patterns

**The Principle**:
Build systems enforcing structured reasoning before action and planning before code generation.

**Why It Matters**:
- **ReAct** (Reason → Act → Observe): Prevents impulsive actions
- **CoT** (Chain of Thought): Forces explicit reasoning
- System-level enforcement > hoping agent follows instructions

**Alignment with Our Project**:
✅ **VALIDATES** self-evaluation loop
✅ **SUPPORTS** planner → implementer → reviewer flow
⚠️ **NEEDS WORK** - Make ReAct/CoT more explicit

**Application**:
Make these patterns systemic, not optional:
- Force reasoning before actions
- Require plan before implementation
- Enforce observation after action
- Document thought process

**Our Implementation Enhancement**:
```markdown
Current approach:
1. Planner creates plan (implicit CoT)
2. Implementer writes code
3. Reviewer evaluates (implicit ReAct observation)

Enhanced approach:
1. Force explicit reasoning in plan.md
2. Log every action taken
3. Require observation/reflection after each action
4. Iterate based on observations
```

### 6. Autonomy Requires Boundaries

**The Principle**:
Define scopes and fallback behaviors—controlled autonomy outperforms random retries.

**Why It Matters**:
- Unbounded autonomy leads to chaos
- Need clear scope: what agent can/cannot do
- Fallback behavior when stuck

**Alignment with Our Project**:
✅ **VALIDATES** spin detection
✅ **VALIDATES** "never merge to main" rule
✅ **SUPPORTS** max retry limits

**Application**:
Define boundaries:
- What can agent change? (not main branch)
- How many retries? (max 2-3)
- When to escalate? (after retries exhausted)
- What requires approval? (PRs, architectural changes)

**Our Implementation**:
```
Boundaries:
✅ Work only on feature branches (never main)
✅ Max 3 retry attempts before escalation
✅ Self-evaluate before creating PR
✅ Never auto-merge (user must approve)
✅ Document blockers when stuck
✅ Spin detection auto-pauses after patterns detected

Scope:
✅ Can: Create files, edit code, run tests, create PRs
❌ Cannot: Merge to main, deploy to prod, delete branches
```

### 7. Orchestration is Core

**The Principle**:
Effective agents coordinate logic, memory, tools, and feedback rather than simply wrapping LLMs.

**Why It Matters**:
- Agent ≠ LLM + prompt
- Need orchestration layer
- Coordinate multiple components

**Alignment with Our Project**:
✅ **VALIDATES** our agent coordination approach
✅ **SUPPORTS** subagent architecture
✅ **CONFIRMS** need for learner agent

**Application**:
Build orchestration layer that coordinates:
- Logic (planner, implementer, reviewer agents)
- Memory (.ai-knowledge/ storage and retrieval)
- Tools (bash, git, IDE, APIs)
- Feedback (learner agent updates knowledge)

**Our Implementation**:
```
Orchestration flow:
1. Main agent receives task
2. Loads relevant skills from memory
3. Delegates to planner agent (logic)
4. Planner queries memory for similar tasks
5. Implementer agent uses tools (bash, git, IDE)
6. Reviewer agent provides feedback
7. If issues: Loop back to implementer
8. Learner agent updates memory with outcomes
9. Create PR for user review
```

## Cross-Cutting Themes

### Theme 1: Systems Over Prompts
**Message**: Don't just write better prompts, build better systems
**Our Response**: ✅ Already doing this (skills, agents, workflows)

### Theme 2: Structure Enables Autonomy
**Message**: Explicit structure (planning, reasoning) enables reliable autonomy
**Our Response**: ✅ Planned, need to make more explicit

### Theme 3: Memory is Critical
**Message**: Memory architecture determines capability
**Our Response**: ✅ Validates .ai-knowledge/ approach

### Theme 4: Boundaries Prevent Chaos
**Message**: Controlled autonomy > unbounded freedom
**Our Response**: ✅ Already implemented (ADR-008, spin detection)

### Theme 5: Tools Make It Real
**Message**: Agents need real tools, not just conversation
**Our Response**: ✅ Scripts, CLI tools, future IDE integration

## What This Validates in Our Approach

### ✅ Strongly Validated

1. **Skills-Based Architecture** (Principle 2: Memory as Architecture)
   - Structured files > full dumps
   - On-demand loading
   - Our approach: ✅ Correct

2. **Multi-Agent Coordination** (Principle 7: Orchestration)
   - Planner → Implementer → Reviewer → Learner
   - Our approach: ✅ Correct

3. **Explicit Planning** (Principle 3: Planning is Essential)
   - .plan/ folder with approach.md
   - Our approach: ✅ Correct

4. **Autonomy Boundaries** (Principle 6: Boundaries)
   - Never merge to main
   - Max retries
   - Our approach: ✅ Correct

5. **Real Tools** (Principle 4: Real-World Tools)
   - Bash, Git, CLI tools
   - Our approach: ✅ Correct

### ⚠️ Needs Enhancement

1. **ReAct/CoT Enforcement** (Principle 5)
   - We have implicit reasoning
   - Need: Explicit, system-enforced patterns
   - Action: Make reasoning/observation required steps

2. **Memory Retrieval** (Principle 2)
   - We have storage planned
   - Need: Scoped retrieval implementation
   - Action: Build query system for .ai-knowledge/

### ❌ Not Addressed in Our Current Approach

Nothing major - our approach already aligns well with Anthropic's guidance.

## Recommendations Based on Best Practices

### Immediate Actions

1. **Make ReAct Pattern Explicit**
   ```markdown
   # Required in every task execution:

   ## REASON (before action)
   - Why this approach?
   - What are the risks?
   - What could go wrong?

   ## ACT (execute)
   - Take action
   - Log all tool calls

   ## OBSERVE (after action)
   - What happened?
   - Did it work as expected?
   - What did we learn?

   ## ITERATE (based on observation)
   - Continue or adjust?
   ```

2. **Enforce Planning for Complex Tasks**
   ```
   If task has 3+ steps:
   1. REQUIRE plan in .plan/approach.md
   2. REQUIRE risk assessment
   3. REQUIRE user approval of plan (optional)
   4. THEN execute
   ```

3. **Document Memory Retrieval Strategy**
   ```
   Before task:
   1. Query .ai-knowledge/ for similar tasks
   2. Load only relevant patterns (not all)
   3. Summarize past approaches (don't dump full code)
   4. Use learned preferences
   ```

### Short-Term Enhancements

1. **Build Orchestration Layer**
   - Explicit coordinator agent
   - Routes to planner/implementer/reviewer/learner
   - Manages memory retrieval
   - Enforces boundaries

2. **Implement Scoped Retrieval**
   - Query `.ai-knowledge/knowledge.db` by task type
   - Retrieve top 5 relevant patterns
   - Load preferences automatically
   - Check for similar failures

3. **Formalize Tool Access**
   - Document which tools agents can use
   - Create tool access control
   - Log all tool usage
   - Audit trail for debugging

### Medium-Term Improvements

1. **CoT Enforcement System**
   - Require explicit reasoning in logs
   - Reject actions without reasoning
   - Quality check on thought process
   - Store reasoning for learning

2. **Boundary Enforcement**
   - Automated checks before actions
   - Prevent main branch changes
   - Limit retry attempts
   - Auto-escalate when stuck

3. **Memory Architecture Enhancement**
   - Add vector search for semantic retrieval
   - Implement summarization for long contexts
   - Build knowledge graph for relationships
   - Hybrid SQL + vector + graph

## Tools Mentioned in Best Practices

### Referenced Tools
- **Goose** - Mentioned as example
- **Aider** - AI pair programming
- **Cursor** - IDE with AI
- **GitHub Copilot Workspace** - Cloud dev environments

### Our Approach
We're building our own because:
1. ✅ More control over architecture
2. ✅ Can integrate learnings from all these tools
3. ✅ Customize to our specific needs (Firebase, Supabase, etc.)
4. ✅ Open-source and extensible

## Notable Community Comment

"AI agents include the past 50 years of information technology wisdom and aren't easily built by anyone, highlighting the complexity involved."

**Our Response**:
- ✅ We're aware of the complexity
- ✅ We're building on established patterns (MCP, skills, agents)
- ✅ We're validating everything experimentally (Principle 3: VALIDATE)
- ✅ We're not reinventing - we're composing proven patterns

## Experiments to Validate Best Practices

### Experiment 1: ReAct vs Ad-Hoc
**Hypothesis**: "Enforced ReAct pattern reduces errors by 40%"
**Design**:
- Control: No structured reasoning required
- Treatment: Enforce Reason → Act → Observe cycle
- Measure: Error rate, retry count, success rate
**Expected**: Fewer errors, clearer debugging

### Experiment 2: Planning for Complex Tasks
**Hypothesis**: "Explicit planning reduces implementation time by 30%"
**Design**:
- Control: Direct implementation on complex tasks
- Treatment: Required planning step first
- Measure: Time, code quality, number of iterations
**Expected**: Slower start, faster overall completion

### Experiment 3: Scoped vs Full Memory
**Hypothesis**: "Scoped retrieval is 60% more efficient than full dump"
**Design**:
- Control: Load all patterns and preferences
- Treatment: Query and load only relevant (top 5)
- Measure: Token usage, relevance, task success
**Expected**: Fewer tokens, same or better quality

## Key Takeaways

1. ✅ **Our approach is validated** by official Anthropic guidance
2. ⚠️ **Need to make patterns more explicit** (ReAct, CoT)
3. ✅ **Systems matter more than prompts** - we're already doing this
4. ✅ **Memory architecture is critical** - validates .ai-knowledge/
5. ✅ **Boundaries enable autonomy** - validates our constraints
6. ✅ **Tools are essential** - validates script-based approach
7. ✅ **Orchestration is core** - validates multi-agent design

## Updates to Our Approach

### SPEC.md Changes Needed
1. Add explicit ReAct enforcement
2. Document orchestration layer
3. Detail memory retrieval strategy
4. Formalize tool access policies

### DECISIONS.md New ADRs
1. **ADR-016**: Enforce ReAct Pattern System-Wide
2. **ADR-017**: Scoped Memory Retrieval Strategy
3. **ADR-018**: Orchestration Layer Architecture

### ROADMAP Adjustments
1. **Phase 1**: Add ReAct enforcement
2. **Phase 2**: Build orchestration layer
3. **Phase 2**: Implement scoped retrieval
4. **Phase 3**: Formalize tool access control

## Related Research

- [Agent Skill Creator](./agent-skill-creator.md) - Skills implementation
- [Mem0](./mem0-memory-layer.md) - Memory architecture
- [Cline](./cline-autonomous-coding.md) - Autonomous coding patterns
- [CrewAI](./crewai-orchestration.md) - Orchestration examples

## Status

**Research Complete**: ✅
**Priority**: CRITICAL - Official guidance from Anthropic
**Impact**: HIGH - Validates most of our approach, highlights ReAct enhancement
**Next Action**: Update SPEC.md and DECISIONS.md with explicit ReAct enforcement
