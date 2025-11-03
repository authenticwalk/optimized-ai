# Token Optimization & Context Window Management

**Status**: ✅ Validated by community best practices and production usage

---

## Table of Contents
1. [Why Token Optimization Matters](#why-token-optimization-matters)
2. [Core Commands](#core-commands)
3. [CLAUDE.md Optimization](#claudemd-optimization)
4. [Session Management](#session-management)
5. [MCP Server Impact](#mcp-server-impact)
6. [Advanced Techniques](#advanced-techniques)
7. [Measurement & Tracking](#measurement--tracking)

---

## Why Token Optimization Matters

### The Problem

```
Context Window: 200,000 tokens

After 20 exchanges:
├─ Conversation history: 50,000 tokens
├─ File contents read: 80,000 tokens
├─ Command outputs: 30,000 tokens
├─ Error logs: 20,000 tokens
└─ Remaining: 20,000 tokens (10%)

Result: Performance degradation, quality drops
```

### Impact on Performance

**Token Usage** → **Cost**
```
Monolithic approach:
- 500-line config always loaded
- All MCP servers active
- No context management
- Average task: 50,000 tokens
- Cost per task: $$$

Optimized approach:
- <250-line core config
- Skills loaded on-demand (30-50 tokens until needed)
- MCP servers scoped to project
- /clear between tasks
- Average task: 15,000 tokens (70% reduction)
- Cost per task: $
```

**Context Quality** → **Results**
```
Cluttered context:
- Old error logs mixed with current work
- Irrelevant files in context
- Claude can't distinguish important from noise
- Quality suffers

Clean context:
- Only relevant information
- Clear focus on current task
- Better decision making
- Higher quality output
```

---

## Core Commands

### /clear - Reset Context

**Purpose**: Clear conversation history and file contents

**When to use**:
- ✅ Between different tasks
- ✅ When switching contexts (frontend → backend)
- ✅ After completing a feature
- ✅ Before starting new work
- ✅ When context feels cluttered

**Example workflow**
```
User: [completes Firebase auth feature]
User: /clear
User: "Now implement Supabase RLS policies"
# Fresh context, no Firebase pollution
```

**Rule of thumb**: Use /clear every time you start something new

### /compact - Summarize Context

**Purpose**: Compress conversation history while keeping key points

**When to use**:
- ✅ At ~50% context window capacity
- ✅ Long conversations with important history
- ✅ When you need some context but not all details
- ✅ Before /clear if you want to preserve summary

**How it works**
```
Before /compact: 100,000 tokens
├─ Full conversation history
├─ All file contents
└─ Complete command outputs

After /compact: 15,000 tokens
├─ Summary of key decisions
├─ Current task state
└─ Important context preserved
```

**Example**
```
User: [after 40 messages working on feature]
User: /compact
# Claude summarizes: "We've implemented auth with email/password,
# added error handling, created tests. Next: add social providers."

User: "Add Google sign-in"
# Context preserved but compressed
```

### /context - Monitor Usage

**Purpose**: See current context window usage

**Output shows**
```bash
/context

# Output:
Context Window Usage:
├─ Conversation: 25,000 tokens (12.5%)
├─ Files read: 15,000 tokens (7.5%)
├─ MCP servers: 5,000 tokens (2.5%)
├─ Skills loaded: 2,000 tokens (1%)
├─ System prompt: 3,000 tokens (1.5%)
└─ Remaining: 150,000 tokens (75%)

Active MCP Servers:
├─ github (2,000 tokens)
├─ supabase (1,500 tokens)
└─ custom-learning (1,500 tokens)

Loaded Skills:
├─ firebase-auth (800 tokens)
└─ testing-patterns (1,200 tokens)
```

**Use for**:
- Monitoring when to /clear or /compact
- Identifying token-heavy MCP servers
- Seeing which skills are loaded
- Optimizing configuration

### Session Management Strategy

**The 20-Exchange Rule**
```
Seasoned developer wisdom:
"Performance craters after ~20 exchanges.
Reset context frequently for fresh code."
```

**Recommended pattern**
```
Task 1: Implement feature A (10 exchanges)
→ /clear
Task 2: Implement feature B (15 exchanges)
→ /clear
Task 3: Debug issue (5 exchanges)
→ /clear
Task 4: Code review (8 exchanges)
```

---

## CLAUDE.md Optimization

### What is CLAUDE.md?

**Auto-loaded file**: Claude reads this at conversation start

**Token impact**: ALWAYS in context
```
CLAUDE.md: 5,000 lines = 15,000 tokens
Every conversation starts with 15,000 tokens used!

CLAUDE.md: 500 lines = 1,500 tokens
Better baseline
```

### Best Practices

**1. Keep it Concise**
```markdown
# ❌ Bad: Everything including the kitchen sink
- 2000 lines of every possible pattern
- Detailed examples for everything
- Complete API documentation
- All possible edge cases
Token cost: 20,000+ always loaded

# ✅ Good: Essential information only
- Project overview (what it is)
- Tech stack (what we use)
- Key conventions (how we work)
- Links to detailed docs (where to learn more)
Token cost: 1,000-2,000 tokens
```

**2. Use Progressive Disclosure**
```markdown
# CLAUDE.md (always loaded - keep minimal)
## Firebase
We use Firebase for:
- Authentication (see .claude/skills/firebase-auth/)
- Firestore (see .claude/skills/firebase-firestore/)
For detailed patterns, skills load automatically.

# Skills load on-demand when needed
firebase-auth/SKILL.md - Detailed auth patterns
firebase-firestore/SKILL.md - Query patterns
```

**3. Specify What to Read/Ignore**
```markdown
## File Access
✅ Read freely:
- src/**/*.ts
- tests/**/*.test.ts
- docs/**/*.md

❌ Never read:
- node_modules/
- dist/
- .git/
- coverage/
- **/*.log

This prevents Claude from wasting tokens on irrelevant files.
```

**4. Link, Don't Duplicate**
```markdown
# ❌ Bad
## Firebase Security Rules
[100 lines of Firebase rules patterns]
[50 lines of examples]
[30 lines of anti-patterns]

# ✅ Good
## Firebase Security Rules
See: .claude/skills/firebase-security/SKILL.md
Load when working with security rules.
```

### Optimal CLAUDE.md Template

```markdown
# [Project Name]

## What This Is
[2-3 sentence description]

## Tech Stack
- **Language**: TypeScript
- **Framework**: Svelte
- **Backend**: Firebase
- **Testing**: Vitest

## Key Conventions
1. No React/Tailwind (use Svelte + CSS)
2. Separate structure from styling
3. Test coverage >90%
4. Refactor on third use

## Skills Available
Skills load automatically when relevant:
- `firebase-auth` - Firebase authentication patterns
- `supabase-rls` - Supabase security policies
- `testing-patterns` - Testing best practices
- See `.claude/skills/` for complete list

## File Organization
```
src/
├── features/     # Feature modules
├── components/   # Shared components
├── lib/          # Utilities
└── types/        # TypeScript types
```

## Commands
- `/init` - Project setup
- `/test` - Run tests
- See `.claude/commands/` for all commands

## Important Files
- `.cursorrules` - Core development rules
- `firebase.json` - Firebase configuration
- See docs/ for detailed documentation

## Context Management
- Only read files in src/, tests/, docs/
- Skip node_modules/, dist/, .git/, coverage/

---
Total: ~300 lines, ~1,000 tokens
```

---

## Session Management

### Strategy 1: Task-Based Sessions

**Pattern**
```
Session 1: Feature A
├─ Implement
├─ Test
├─ Review
└─ /clear

Session 2: Feature B
├─ Implement
├─ Test
├─ Review
└─ /clear
```

**Benefits**
- Clean slate per feature
- No cross-contamination
- Easier to track token usage per feature

### Strategy 2: 50% Threshold

**Pattern**
```
Monitor context usage:
├─ 0-25%: Continue working
├─ 25-50%: Consider /compact soon
├─ 50-75%: Use /compact now
└─ 75%+: Use /clear and restart

Check with /context regularly
```

### Strategy 3: Time-Based

**Pattern**
```
Every 30-60 minutes:
1. Check /context
2. If >50%: /compact
3. If switching focus: /clear
4. Restart with clean context
```

---

## MCP Server Impact

### Token Cost per Server

**Typical MCP server**:
```
Server definition: ~500 tokens
├─ Server metadata
├─ Tool definitions (name, description, schema)
├─ Resource definitions
└─ Prompt definitions

5 servers = 2,500 tokens
10 servers = 5,000 tokens (2.5% of context!)
```

### Optimization Strategies

**1. Scope to Project**

```json
// ❌ Bad: Global config with all servers
// ~/.claude/settings.json
{
  "mcpServers": {
    "github": {...},
    "gitlab": {...},
    "bitbucket": {...},
    "aws": {...},
    "gcp": {...},
    "azure": {...},
    // 15+ servers globally enabled
  }
}

// ✅ Good: Project-specific
// project-a/.mcp.json
{
  "mcpServers": {
    "github": {...},
    "firebase": {...}
  }
}

// project-b/.mcp.json
{
  "mcpServers": {
    "github": {...},
    "supabase": {...}
  }
}
```

**2. Disable Unused Servers**

```bash
# List active servers
claude mcp list

# Check token impact
/context

# Disable unused
claude mcp remove unused-server

# Re-enable when needed
claude mcp add needed-server
```

**3. Monitor with /context**

```bash
/context

# Shows:
Active MCP Servers:
├─ github (2,000 tokens) ← Maybe needed
├─ aws (3,000 tokens) ← Not using AWS now, disable?
└─ custom (1,000 tokens) ← Required

# Disable AWS for this project
claude mcp remove aws
```

---

## Advanced Techniques

### 1. Skill-Based Loading

**Instead of loading everything in CLAUDE.md**

```yaml
# firebase-auth/SKILL.md
---
name: firebase-auth
description: Firebase authentication setup and patterns. Use when implementing login, signup, or auth features.
---

[Detailed auth patterns - only loaded when needed]
[~3,000 tokens, but only when auth is mentioned]
```

**Token savings**
```
Monolithic CLAUDE.md: 20,000 tokens always

Skill-based approach:
├─ CLAUDE.md: 1,000 tokens (always)
├─ Skill descriptions: 10 × 50 = 500 tokens (always)
├─ Loaded skill: 3,000 tokens (only when needed)
└─ Total when skill needed: 4,500 tokens (77% savings!)
```

### 2. Progressive Disclosure in Skills

```
skill-name/
├── SKILL.md (400 lines - quick reference, always loaded when skill activates)
├── ADVANCED.md (1,000 lines - only referenced when needed)
├── EXAMPLES.md (800 lines - only loaded if requested)
└── TROUBLESHOOTING.md (600 lines - only for problems)
```

**In SKILL.md**
```markdown
## Basic Patterns
[Essential patterns here]

## Advanced Usage
For complex scenarios, see [ADVANCED.md](ADVANCED.md).

## Examples
See [EXAMPLES.md](EXAMPLES.md) for complete examples.
```

**Claude only loads ADVANCED.md if needed**

### 3. Reference Files, Don't Read Them

```markdown
# ❌ Bad: Read entire file into context
User: "Check the API documentation"
Claude: [reads 5,000-line API.md into context]
Result: +15,000 tokens used

# ✅ Good: Read specific sections
User: "Check authentication in API docs"
Claude: [reads just the auth section using grep]
Result: +500 tokens used
```

### 4. Use Grep Instead of Read for Large Files

```bash
# ❌ Bad
Read entire 2,000-line file to find one function
Token cost: 6,000 tokens

# ✅ Good
Grep for specific function
Token cost: 200 tokens (just the function)
```

### 5. Structured Communication

**Clear, numbered steps prevent exploration**

```markdown
# ❌ Vague request
User: "Make the auth better"
Claude: [explores many possibilities, asks many questions]
Result: 10 messages, 30,000 tokens

# ✅ Clear request
User: "Add Google OAuth to existing Firebase auth:
1. Update firebase config
2. Add Google provider button
3. Handle Google sign-in flow
4. Test with Google account"
Claude: [follows steps directly]
Result: 3 messages, 8,000 tokens
```

### 6. Batch Related Operations

```markdown
# ❌ Inefficient
"Add user model"
"Add user service"
"Add user tests"
"Add user documentation"
Result: 4 separate contexts

# ✅ Efficient
"Add user feature:
- Model with TypeScript types
- Service with CRUD operations
- Tests with >90% coverage
- JSDoc documentation"
Result: 1 context, shared understanding
```

---

## Measurement & Tracking

### Baseline Measurement

**Before optimization**
```
Monolithic approach:
├─ CLAUDE.md: 20,000 tokens
├─ .cursorrules: 10,000 tokens
├─ All MCP servers: 5,000 tokens
├─ Per task average: 50,000 tokens
└─ Cost per task: $$$
```

**After optimization**
```
Skills-based approach:
├─ CLAUDE.md: 1,000 tokens
├─ .cursorrules: 500 tokens
├─ Project MCP servers: 2,000 tokens
├─ Skills (on-demand): 3,000 tokens
├─ Per task average: 15,000 tokens (70% reduction)
└─ Cost per task: $
```

### Track These Metrics

**Per Task**
```json
{
  "task": "implement-firebase-auth",
  "tokens_used": 15240,
  "context_clears": 0,
  "compactions": 1,
  "skills_loaded": ["firebase-auth", "testing-patterns"],
  "mcp_servers_used": ["github"],
  "time_elapsed": "18m",
  "quality_score": 9.5
}
```

**Aggregate**
```
Total tasks: 100
Average tokens per task: 14,567
Total savings vs baseline: 3,543,300 tokens
Cost savings: $425
Token reduction: 71%
```

### OpenTelemetry Integration

**For Optimized AI project**
```typescript
// Track token usage with OpenTelemetry
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('optimized-ai');

function trackTaskTokens(task: string, tokens: number) {
  const span = tracer.startSpan('claude_task');
  span.setAttribute('task.name', task);
  span.setAttribute('task.tokens', tokens);
  span.setAttribute('task.timestamp', Date.now());
  span.end();
}
```

**Analyze trends**
```sql
SELECT
  task_type,
  AVG(tokens_used) as avg_tokens,
  MIN(tokens_used) as min_tokens,
  MAX(tokens_used) as max_tokens
FROM task_metrics
GROUP BY task_type
ORDER BY avg_tokens DESC;
```

---

## Key Takeaways for Optimized AI

### 1. Use /clear Aggressively
- Between every task
- Rule: "New task = /clear"
- Measure impact

### 2. Keep CLAUDE.md Minimal
- Target: <500 lines
- Essential info only
- Link to skills for details

### 3. Skills Over Monolithic Config
- Skills: 30-50 tokens until loaded
- Monolithic: Always all loaded
- Savings: 60-70%

### 4. Monitor Context
- Use /context regularly
- /compact at 50%
- Track MCP server impact

### 5. Scope MCP Servers
- Project-level configs
- Only enable what's needed
- Disable when not in use

### 6. Measure Everything
- Baseline before optimization
- Track tokens per task
- Prove 60%+ reduction claim

---

## Experimental Validation Plan

### Hypothesis
"Skills-based on-demand loading reduces token usage by 60%+ vs monolithic approach without quality degradation"

### Experiment Design

**Control Group**
- Monolithic CLAUDE.md (2,000 lines)
- All instructions loaded upfront
- All MCP servers enabled
- No skills

**Treatment Group**
- Minimal CLAUDE.md (500 lines)
- Skills load on-demand
- MCP servers scoped to project
- Regular /clear usage

**Scenarios** (10 common tasks)
1. Add Firebase authentication
2. Implement Supabase RLS
3. Create test suite
4. Refactor component
5. Debug production issue
6. Add API endpoint
7. Optimize query performance
8. Update documentation
9. Review pull request
10. Deploy to production

**Metrics**
- Tokens used per task
- Time to completion
- Quality score (tests pass, linting, requirements)
- Number of clarifications needed
- /clear and /compact usage

**Success Criteria**
- ✅ 60%+ token reduction
- ✅ Equal or better quality
- ✅ No increase in clarifications
- ✅ Similar or faster completion time

---

## Resources

### Official
- [Claude Code Context Windows](https://docs.claude.com/en/docs/build-with-claude/context-windows)

### Community
- [ClaudeLog: Token Optimization](https://claudelog.com/faqs/how-to-optimize-claude-code-token-usage/)
- [Token Management for Pro Users](https://richardporter.dev/blog/claude-code-token-management)
- [76% Token Reduction Case Study](https://web-werkstatt.at/aktuell/breaking-the-claude-context-limit-how-we-achieved-76-token-reduction-without-quality-loss/)

---

**Next**: Read [05-HOOKS-CONFIGURATION.md](05-HOOKS-CONFIGURATION.md) for automation with hooks
