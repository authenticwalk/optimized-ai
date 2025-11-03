# Agent Tools

**Tools Covered**: Task (subagent dispatch)

## Overview

The Task tool is "Claude's most powerful tool" - enabling parallel execution and context isolation through subagents. This aligns perfectly with the SEPARATE principle.

## Task Tool

### How It Works

**Purpose**: Launch specialized subagents with fresh context windows

**Technical Implementation**:
- Spawns new Claude Code instance (separate conversation)
- Fresh context window (no pollution from main agent)
- Each subagent has access to own set of tools
- Returns final result to main agent (no back-and-forth)
- Stateless - each invocation is independent

**Key insight**: Subagents ARE Claude Code instances, just with:
- Fresh context
- Specific instructions
- Limited scope
- Cannot spawn their own subagents (no recursion)

### Parameters

```typescript
{
  subagent_type: string;    // REQUIRED: Type of agent to launch
  prompt: string;           // REQUIRED: Detailed task description
  description: string;      // REQUIRED: Short 3-5 word description
}
```

### Available Subagent Types

**1. general-purpose**
- **Tools**: All tools (Read, Write, Edit, Bash, Glob, Grep, WebFetch, etc.)
- **Use for**: Complex multi-step tasks, code implementation, research
- **When**: Need full capabilities but want fresh context

**2. Explore**
- **Tools**: Glob, Grep, Read, Bash
- **Use for**: Codebase exploration, finding patterns, understanding structure
- **When**: Open-ended search, learning about codebase
- **Thoroughness levels**: "quick", "medium", "very thorough"

**3. statusline-setup**
- **Tools**: Read, Edit
- **Use for**: Configuring status line settings
- **When**: User wants status line customization

**4. output-style-setup**
- **Tools**: Read, Write, Edit, Glob, Grep
- **Use for**: Creating output styles
- **When**: User wants custom output formatting

### Parallel Execution

**Maximum concurrency**: 10 agents simultaneously

**Queuing behavior**:
- Can request 100+ tasks
- System queues excess beyond 10
- Waits for batch completion before starting next batch

**Example**:
```typescript
// Launch 7 agents in parallel
Promise.all([
  Task({ subagent_type: "general-purpose", prompt: "Create auth component", description: "Create auth component" }),
  Task({ subagent_type: "general-purpose", prompt: "Create styling", description: "Create styling" }),
  Task({ subagent_type: "general-purpose", prompt: "Write tests", description: "Write tests" }),
  Task({ subagent_type: "general-purpose", prompt: "Create types", description: "Create types" }),
  Task({ subagent_type: "general-purpose", prompt: "Create hooks", description: "Create hooks" }),
  Task({ subagent_type: "general-purpose", prompt: "Integration", description: "Integration" }),
  Task({ subagent_type: "general-purpose", prompt: "Remaining work", description: "Remaining work" })
])
// All 7 run simultaneously
```

### Why Use Subagents?

**1. Context Isolation** (SEPARATE principle):
- Main agent context stays clean
- Each subagent has focused context
- No cross-contamination
- Easier debugging (smaller contexts)

**2. Parallel Execution**:
- 10x faster for independent tasks
- Main agent continues working
- Results gathered when needed

**3. Specialization**:
- Each agent optimized for specific task
- Can use different system prompts (if configured)
- Explore agent specialized for codebase exploration

**4. Token Efficiency**:
- Avoid loading irrelevant context in main agent
- Each subagent has minimal context
- Results summarized, not full transcript

### When to Use Subagents

**Use Task tool when**:
- ✅ Task has 3+ distinct phases
- ✅ Tasks can run in parallel (independent)
- ✅ Main context is getting bloated
- ✅ Need focused expertise (e.g., exploration)
- ✅ Complex multi-file operations

**Don't use when**:
- ❌ Simple, single-file changes
- ❌ Need shared context between operations
- ❌ Handoff overhead > benefit
- ❌ Sequential dependencies (A requires B's output)

### Prompt Design for Subagents

**Critical**: Subagents are autonomous. Prompt must be complete.

**Good prompt** (detailed, autonomous):
```typescript
Task({
  subagent_type: "general-purpose",
  prompt: `Create a user authentication component in TypeScript.

Requirements:
- Use Firebase Authentication
- Support email/password login
- Support Google OAuth
- Handle errors gracefully
- Add loading states
- Write tests using Vitest
- Follow project patterns in src/components/

Files to create:
- src/components/Auth/LoginForm.tsx
- src/components/Auth/AuthProvider.tsx
- src/components/Auth/types.ts
- src/components/Auth/__tests__/LoginForm.test.ts

Return a summary of what was created and any issues encountered.`,
  description: "Create auth component"
})
```

**Bad prompt** (vague, assumes context):
```typescript
Task({
  subagent_type: "general-purpose",
  prompt: "Add authentication",  // Too vague
  description: "Add auth"
})
// Subagent won't know what to do
```

### Result Handling

**Subagents return**:
- Final message/report (not full transcript)
- Summary of actions taken
- Any blockers or issues
- Relevant output

**You cannot**:
- Send follow-up messages to subagent
- Get intermediate updates
- Debug subagent's process (no visibility)

**Implications**:
- Prompt must be complete and clear
- Include all necessary context
- Specify exactly what to return

### Explore Agent Deep Dive

**Special-purpose agent for codebase exploration**

**Thoroughness parameter**:
```typescript
// Quick exploration
Task({
  subagent_type: "Explore",
  prompt: "Quick: Find all API endpoints. thoroughness: quick",
  description: "Find API endpoints"
})

// Medium exploration
Task({
  subagent_type: "Explore",
  prompt: "Find authentication logic. thoroughness: medium",
  description: "Find auth logic"
})

// Thorough exploration
Task({
  subagent_type: "Explore",
  prompt: "Comprehensive analysis of state management across the entire codebase. thoroughness: very thorough",
  description: "Analyze state management"
})
```

**When to use Explore**:
- Don't know where code is
- Need to understand codebase structure
- Looking for patterns across many files
- Ambiguous search ("find all auth logic")

**When NOT to use**:
- Know exact file path (just Read it)
- Simple pattern search (use Glob/Grep)
- Need to modify code (use general-purpose)

### Parallel Task Patterns

**Pattern 1: Component Creation**
```typescript
// Create entire feature in parallel
const components = [
  "Header", "Footer", "Sidebar",
  "MainContent", "Navigation"
]

Promise.all(
  components.map(name =>
    Task({
      subagent_type: "general-purpose",
      prompt: `Create ${name} component following project patterns. Include TypeScript types, tests, and styling.`,
      description: `Create ${name}`
    })
  )
)
```

**Pattern 2: Multi-File Refactoring**
```typescript
// Refactor multiple files independently
Promise.all([
  Task({
    subagent_type: "general-purpose",
    prompt: "Refactor auth.ts to use new error handling pattern",
    description: "Refactor auth.ts"
  }),
  Task({
    subagent_type: "general-purpose",
    prompt: "Refactor user-service.ts to use new error handling pattern",
    description: "Refactor user-service"
  }),
  Task({
    subagent_type: "general-purpose",
    prompt: "Refactor data-service.ts to use new error handling pattern",
    description: "Refactor data-service"
  })
])
```

**Pattern 3: Research + Implementation**
```typescript
// Parallel research and implementation
Promise.all([
  Task({
    subagent_type: "Explore",
    prompt: "Find all places where we handle user authentication. thoroughness: medium",
    description: "Find auth code"
  }),
  Task({
    subagent_type: "Explore",
    prompt: "Find all API endpoints related to users. thoroughness: medium",
    description: "Find user APIs"
  }),
  Task({
    subagent_type: "general-purpose",
    prompt: "Create new authentication middleware following existing patterns",
    description: "Create auth middleware"
  })
])
```

### Avoiding Write Conflicts

**Problem**: Multiple subagents writing to same file simultaneously

**Solution**: Coordinate write operations

**Bad** (potential conflict):
```typescript
Promise.all([
  Task({
    subagent_type: "general-purpose",
    prompt: "Add function A to utils.ts",
    description: "Add function A"
  }),
  Task({
    subagent_type: "general-purpose",
    prompt: "Add function B to utils.ts",
    description: "Add function B"
  })
])
// Both might read, edit, write simultaneously - conflict!
```

**Good** (separate files):
```typescript
Promise.all([
  Task({
    subagent_type: "general-purpose",
    prompt: "Add function A to utils-a.ts",
    description: "Add function A"
  }),
  Task({
    subagent_type: "general-purpose",
    prompt: "Add function B to utils-b.ts",
    description: "Add function B"
  })
])
// No conflict - different files
```

**Good** (one agent for shared file):
```typescript
Task({
  subagent_type: "general-purpose",
  prompt: "Add functions A and B to utils.ts",
  description: "Add utils functions"
})
// Single agent handles all changes to utils.ts
```

### Cost-Performance Balance

**Token costs increase with subagents**:
```
Main agent context: 10,000 tokens
Each subagent startup: ~5,000 tokens (system prompt, initial context)
Each subagent operation: 2,000-20,000 tokens

Total for 10 parallel subagents: 50,000-200,000 tokens
```

**Balance**:
- More subagents = more parallelism = faster
- More subagents = more cost
- More subagents = more complexity

**Optimization**:
```typescript
// Too granular (wasteful):
components.map(c => Task({ ... }))  // 20 tiny tasks

// Too monolithic (slow):
Task({ prompt: "Create all 20 components" })  // 1 massive task

// Optimal (grouped):
Task({ prompt: "Create Header, Nav, Footer" })     // 3 components
Task({ prompt: "Create Sidebar, Menu, Profile" })  // 3 components
// ... 6-7 balanced tasks
```

### Immediate Delegation Pattern

**Anti-pattern** (seek clarification first):
```typescript
User: "Build a user dashboard"
Claude: "What features do you want in the dashboard?"
// Wastes time - could have started working
```

**Best practice** (launch immediately):
```typescript
User: "Build a user dashboard"
Claude: "Launching parallel tasks to build dashboard..."
Promise.all([
  Task({ prompt: "Explore existing user components. thoroughness: quick" }),
  Task({ prompt: "Create dashboard layout component" }),
  Task({ prompt: "Create user profile card component" }),
  Task({ prompt: "Create activity feed component" })
])
```

**Why**: User can always provide feedback; better to show progress

### Debugging Subagent Issues

**Limited visibility**:
- Can't see subagent's thinking process
- Can't debug step-by-step
- Only see final result

**Strategies**:
1. **Request detailed reporting**: Include in prompt "Explain all steps taken and decisions made"
2. **Smaller tasks**: Easier to understand what went wrong
3. **Retry with more context**: If fails, provide more specific instructions
4. **Fall back to main agent**: For debugging, use main agent directly

### Integration with Skills

**Subagents can use skills**:
```typescript
Task({
  subagent_type: "general-purpose",
  prompt: `Create Firebase authentication component.

Use firebase-auth skill for best practices.
Follow testing skill for test structure.`,
  description: "Create Firebase auth"
})
```

**Skills loaded in subagent context**:
- Core .cursorrules (minimal)
- Relevant skills (auto-detected or specified)
- Total: 80-180 lines

**Benefit**: Each subagent gets focused context

## Summary: Agent Tool Optimization

### Context Isolation (SEPARATE)

1. **Use subagents for distinct phases**: 3+ separate tasks
2. **Keep main context clean**: Delegate heavy operations
3. **Parallel when possible**: 10 concurrent agents max
4. **Avoid write conflicts**: Coordinate file access

### Token Efficiency (MINIMIZE)

1. **Group related tasks**: Don't over-split
2. **Detailed prompts**: Avoid back-and-forth
3. **Request summaries**: Not full transcripts
4. **Use Explore agent**: For codebase discovery

### Performance

1. **Immediate delegation**: Don't wait for clarification
2. **Batch operations**: Launch all tasks together
3. **Balance parallelism**: 5-10 agents is sweet spot
4. **Monitor costs**: Track token usage per subagent

### Reliability

1. **Complete prompts**: Include all context
2. **Specify deliverables**: What to return
3. **Handle failures**: Retry with more context
4. **Avoid dependencies**: Tasks should be independent

---

**Next**: [Advanced Features (Skills, Hooks, MCP)](./08-ADVANCED-FEATURES.md)
