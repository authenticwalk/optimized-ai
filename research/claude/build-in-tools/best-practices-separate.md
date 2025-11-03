# Best Practices for Context Separation

**Aligns with**: Principle 2 - SEPARATE (Context isolation via skills and subagents)

## Overview

Different tasks need different context. Polluting context with irrelevant information wastes tokens and confuses the model. This document provides strategies for maintaining clean, focused contexts.

## Core Concept

**Problem**: Monolithic context
```
Main agent context:
- Firebase auth patterns (100 lines)
- Supabase RLS patterns (100 lines)
- Testing strategies (100 lines)
- Refactoring guidelines (100 lines)
- CSS best practices (100 lines)
- API design patterns (100 lines)

Total: 600 lines loaded for EVERY task
Even when only 20 lines are relevant
```

**Solution**: Isolated contexts
```
Main agent context:
- Core principles (80 lines)
+ firebase-auth skill (loaded only for auth tasks)

Total: 180 lines of RELEVANT context
```

## Strategy 1: Skills for On-Demand Loading

### Automatic Skill Loading

**How it works**:
1. User provides task
2. Claude analyzes keywords and files
3. Loads only relevant skills
4. Executes with focused context

**Example**:
```
Task: "Implement Firebase authentication"

Claude detects:
- Keywords: "Firebase", "authentication"
- Loads: firebase-auth.skill

Context:
- Core .cursorrules: 80 lines
- firebase-auth.skill: 100 lines
- Total: 180 lines

NOT loaded:
- supabase-rls.skill (not relevant)
- testing.skill (not mentioned)
- refactoring.skill (not needed)
```

### Skill Organization

**Directory structure**:
```
.claude/skills/
├── firebase/
│   ├── auth/SKILL.md           (100 lines)
│   ├── firestore/SKILL.md      (100 lines)
│   └── functions/SKILL.md      (100 lines)
├── supabase/
│   ├── rls/SKILL.md            (100 lines)
│   └── queries/SKILL.md        (100 lines)
├── patterns/
│   ├── testing/SKILL.md        (100 lines)
│   ├── refactoring/SKILL.md    (100 lines)
│   └── performance/SKILL.md    (100 lines)
└── frameworks/
    ├── svelte/SKILL.md         (100 lines)
    └── htmx/SKILL.md           (100 lines)
```

**Benefits**:
- Each skill focused on single topic
- Load only what's needed
- No cross-contamination
- Easy to maintain

### Skill Trigger Design

**Good triggers** (specific):
```yaml
---
name: firebase-auth
triggers:
  keywords:
    - firebase authentication
    - firebase auth
    - firebase login
    - firebase signup
  files:
    - firebase.config.*
    - auth/*.ts
    - **/firebase-auth.ts
---
```

**Bad triggers** (too broad):
```yaml
---
name: general-patterns
triggers:
  keywords:
    - code
    - function
    - component
  files:
    - "**/*.ts"
---
```
**Problem**: Matches everything, always loaded

### Skill Combinations

**Track successful combinations**:
```json
{
  "skill_combinations": {
    "firebase-auth + testing": {
      "frequency": 45,
      "avg_tokens": 720,
      "avg_success": 0.92
    },
    "firebase-firestore + testing": {
      "frequency": 38,
      "avg_tokens": 740,
      "avg_success": 0.90
    },
    "refactoring + testing": {
      "frequency": 60,
      "avg_tokens": 700,
      "avg_success": 0.95
    }
  }
}
```

**Auto-suggestion**:
```
Task: "Implement Firebase auth"
Claude loads: firebase-auth.skill

Context injection:
"Based on historical patterns, consider also using the testing skill.
Firebase auth tasks typically benefit from test coverage."
```

## Strategy 2: Subagents for Fresh Contexts

### When to Use Subagents

**Use subagents for**:
- ✅ Tasks with 3+ distinct phases
- ✅ Parallel independent operations
- ✅ When main context is bloated
- ✅ Specialized exploration (Explore agent)
- ✅ Clear handoff points

**Don't use for**:
- ❌ Simple, single-file changes
- ❌ Tasks requiring shared context
- ❌ When handoff overhead > benefit

### Subagent Context Isolation

**Main agent**:
```
Context:
- Core .cursorrules (80 lines)
- Current task plan (50 lines)
- Recent file reads (1,000 tokens)
- Previous tool outputs (2,000 tokens)

Total: ~4,000 tokens
```

**Subagent** (fresh start):
```
Context:
- Core .cursorrules (80 lines)
- Subagent-specific instructions (from Task prompt)
- Relevant skill (100 lines if needed)

Total: ~800 tokens

NOT included:
- Main agent's file reads
- Main agent's previous operations
- Irrelevant history
```

**Benefit**: Each subagent starts clean

### Parallel Subagent Pattern

**Example: Create feature with 5 components**

```typescript
// Launch 5 subagents in parallel
const tasks = [
  "CreateHeader",
  "CreateFooter",
  "CreateSidebar",
  "CreateMainContent",
  "CreateNavigation"
]

Promise.all(
  tasks.map(component =>
    Task({
      subagent_type: "general-purpose",
      prompt: `
Create ${component} component following project patterns.

Requirements:
- TypeScript with proper types
- Svelte component
- Vitest tests
- Accessible (ARIA)

Location: src/components/${component}.svelte

Use the svelte skill for best practices.
Use the testing skill for test structure.

Return summary of what was created.
      `,
      description: `Create ${component}`
    })
  )
)
```

**Each subagent**:
- Fresh context
- Loads only relevant skills (svelte, testing)
- No cross-contamination
- Runs in parallel
- Returns summary only

**Main agent**:
- Context stays clean
- Receives 5 summaries (~500 tokens each)
- Can integrate results

### Subagent Orchestration

**Bad** (shared context, sequential):
```typescript
// All in main agent
Read("src/components/Header.svelte")
// Main context: +800 tokens
Read("src/components/Footer.svelte")
// Main context: +800 tokens
Read("src/components/Sidebar.svelte")
// Main context: +800 tokens
// ... create new components
// Main context: Now 10,000+ tokens
```

**Good** (isolated contexts, parallel):
```typescript
// Subagent 1
Task({
  prompt: "Analyze Header.svelte and create NewHeader.svelte"
})
// Context: 800 tokens, returns summary

// Subagent 2
Task({
  prompt: "Analyze Footer.svelte and create NewFooter.svelte"
})
// Context: 800 tokens, returns summary

// Main agent context: Still clean!
```

## Strategy 3: Skills vs Subagents vs Scripts

### Decision Matrix

| Need | Use | Why |
|------|-----|-----|
| Teach patterns/best practices | **Skill** | Instructions for Claude |
| Automate repetitive tasks | **Hook** | Runs automatically |
| Query external data | **Script** | Direct CLI calls |
| Complex external integration | **MCP** | When scripts insufficient |
| Fresh context for task | **Subagent** | Isolation + parallelism |

### Example: Firebase Authentication

**Skill** (firebase-auth/SKILL.md):
```markdown
---
name: firebase-auth
description: Firebase Authentication patterns
---

# Setup Pattern
1. Initialize Firebase config
2. Create auth service
3. Handle errors properly
4. Add loading states

# Security Best Practices
- Never expose API keys
- Use environment variables
- Validate auth state
```
**Use**: Teaching Claude how to implement Firebase auth

**Hook** (.claude/settings.json):
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [{
          "type": "command",
          "command": "~/.claude/hooks/firebase-check.sh"
        }]
      }
    ]
  }
}
```
**Use**: Auto-check Firebase CLI availability

**Script** (direct CLI):
```bash
# Query auth users
firebase auth:export users.json

# Check Firebase connection
firebase projects:list
```
**Use**: Get actual data from Firebase

**Subagent**:
```typescript
Task({
  subagent_type: "Explore",
  prompt: "Find all Firebase authentication usage in codebase. thoroughness: medium"
})
```
**Use**: Explore codebase without polluting main context

### No MCP Needed!

**Why**: Firebase CLI provides everything
```bash
firebase auth:export          # Get users
firebase firestore:get        # Query data
firebase functions:log        # Check logs
firebase deploy --dry-run     # Test deploy
```

**When you'd need MCP**:
- Real-time Firebase listeners
- Complex state synchronization
- Custom Firebase operations not in CLI

**Default**: Try scripts first

## Strategy 4: Context Cleanup

### Avoid Context Bloat

**Problem**: Context grows over time
```
Session start: 4,000 tokens
After 10 reads: 14,000 tokens
After 20 operations: 30,000 tokens
After 30 operations: Context limit exceeded!
```

**Solution 1: Subagents**
```typescript
// Instead of reading 20 files in main agent
Task({
  subagent_type: "Explore",
  prompt: "Analyze all 20 test files and return summary of patterns used",
  description: "Analyze test patterns"
})
// Main context: +500 tokens (summary only)
// vs. +16,000 tokens (all files)
```

**Solution 2: Summaries**
```typescript
// After reading many files
"I've analyzed 15 auth-related files. Here's the summary:
- All use Firebase auth
- 3 different error handling patterns
- 2 files need refactoring

Main patterns:
[200-word summary]

Let me know which file to focus on."
```

**Solution 3: Fresh sessions**
```bash
# For new major task, start fresh session
claude  # New context, clean slate
```

### TodoWrite for Context Anchoring

**Problem**: Long sessions lose track

**Solution**: Use TodoWrite to anchor context
```typescript
// At session start
TodoWrite({
  todos: [
    {
      content: "Analyze existing auth code",
      activeForm: "Analyzing auth code",
      status: "in_progress"
    },
    {
      content: "Implement new auth flow",
      activeForm: "Implementing auth flow",
      status: "pending"
    },
    {
      content: "Write tests for auth",
      activeForm: "Writing auth tests",
      status: "pending"
    }
  ]
})

// System reminders inject todo list periodically
// Keeps focus even as context grows
```

## Strategy 5: Skill Scope Management

### Single Responsibility

**Bad** (multiple responsibilities):
```markdown
---
name: backend
description: All backend patterns
---

# Firebase
[100 lines]

# Supabase
[100 lines]

# API Design
[100 lines]

# Error Handling
[100 lines]

# Security
[100 lines]

Total: 500 lines
```
**Problem**: Loads unnecessary context

**Good** (focused skills):
```markdown
# firebase-auth/SKILL.md (100 lines)
Firebase authentication only

# supabase-rls/SKILL.md (100 lines)
Supabase RLS policies only

# api-design/SKILL.md (100 lines)
REST API design only

# error-handling/SKILL.md (100 lines)
Error handling patterns only
```
**Benefit**: Load only what's needed

### Skill Composition

**For complex tasks, load multiple focused skills**:
```
Task: "Implement secure user registration with Firebase"

Claude loads:
- firebase-auth.skill (auth patterns)
- security.skill (security best practices)
- testing.skill (test structure)

Total: 80 (core) + 300 (skills) = 380 lines
Still less than 500-line monolithic config!
```

## Strategy 6: Explore Agent for Discovery

### Why Explore Agent Exists

**Problem**: Discovery pollutes main context
```
User: "Find all authentication logic"

Main agent:
1. Glob(**/*.ts)                    // +1,000 tokens
2. Grep("auth")                     // +500 tokens
3. Grep("login")                    // +500 tokens
4. Grep("authenticate")             // +500 tokens
5. Read(auth.ts)                    // +800 tokens
6. Read(user-service.ts)            // +800 tokens
7. Read(middleware.ts)              // +800 tokens

Main context: +4,900 tokens
```

**Solution**: Explore agent with fresh context
```
Task({
  subagent_type: "Explore",
  prompt: "Find all authentication logic. thoroughness: medium",
  description: "Find auth logic"
})

Explore agent (fresh context):
1. Glob, Grep, Read (in subagent context)
2. Analyzes patterns
3. Returns summary (~500 tokens)

Main context: +500 tokens (94% savings!)
```

### Thoroughness Levels

**Quick** (~1,000 tokens):
```typescript
Task({
  subagent_type: "Explore",
  prompt: "Quick: Find API endpoints. thoroughness: quick",
  description: "Find APIs"
})
// Fast, surface-level, limited reads
```

**Medium** (~3,000 tokens):
```typescript
Task({
  subagent_type: "Explore",
  prompt: "Find authentication logic. thoroughness: medium",
  description: "Find auth"
})
// Balanced, reasonable depth, selective reads
```

**Very Thorough** (~10,000 tokens):
```typescript
Task({
  subagent_type: "Explore",
  prompt: "Comprehensive analysis of entire state management. thoroughness: very thorough",
  description: "Analyze state"
})
// Deep dive, many reads, comprehensive
```

**Use cases**:
- Quick: Known location, just need confirmation
- Medium: Don't know where code is, need to find it
- Very thorough: Need complete understanding of complex system

## Strategy 7: Avoid Context Leakage

### Problem: Shared References

**Bad**:
```typescript
// Main agent reads file
Read("config.ts")
// Main context: +800 tokens

// Later, spawn subagent
Task({
  prompt: "Update the config file"
  // Which config file?
  // Subagent doesn't have context!
})
```

**Good**:
```typescript
// Explicit in subagent prompt
Task({
  prompt: `
Update the config file at src/config.ts

Current issues:
- Missing DATABASE_URL
- Old API version

Add these fields with proper defaults.
  `
  // Self-contained, no assumed context
})
```

### Problem: Assuming Shared State

**Bad**:
```typescript
// Main agent
const results = Grep("getUserId")

Task({
  prompt: "Refactor those files"
  // Which files? Subagent doesn't know!
})
```

**Good**:
```typescript
// Main agent
const results = Grep("getUserId")
// results: ["auth.ts", "user-service.ts"]

Task({
  prompt: `
Refactor these files to use getAccountId instead:
- src/auth.ts
- src/user-service.ts

Replace all instances of getUserId with getAccountId.
Use replace_all for efficiency.
  `
})
```

## Summary: Context Separation Checklist

### Skills (On-Demand Loading)
- ✅ Skills ~100 lines each
- ✅ Specific triggers (keywords + files)
- ✅ Single responsibility
- ✅ Load only what's needed
- ✅ Track successful combinations

### Subagents (Fresh Contexts)
- ✅ Use for 3+ phase tasks
- ✅ Parallel independent operations
- ✅ Fresh context per subagent
- ✅ Return summaries, not full data
- ✅ Self-contained prompts

### Context Management
- ✅ Avoid bloat in main agent
- ✅ Use Explore agent for discovery
- ✅ TodoWrite for anchoring
- ✅ Summaries over full data
- ✅ Fresh sessions for major tasks

### Integration
- ✅ Skill for patterns
- ✅ Hook for automation
- ✅ Script for data
- ✅ Subagent for isolation
- ✅ MCP only when necessary

### Validation
- ✅ Track context sizes
- ✅ Measure token usage per approach
- ✅ A/B test (Phase 0)
- ✅ Optimize based on data

---

**Target**: 80-180 lines loaded per task (vs. 500+ monolithic)

**Method**: Skills + Subagents + Clean handoffs

**Benefit**: Faster, cheaper, clearer execution

**Next**: [Performance Analysis](./11-PERFORMANCE-ANALYSIS.md)
