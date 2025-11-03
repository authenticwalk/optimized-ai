# Skills Analysis - Claude Code Tresor

**Focus**: Autonomous background helpers that work automatically

---

## Overview

Skills are the foundation of Tresor's v2.0.0 architecture - lightweight, always-on helpers that provide real-time feedback without requiring user invocation. This is the most innovative aspect of the Tresor system.

## Skill Architecture

### Core Pattern

```yaml
---
name: skill-name
description: Trigger keywords here. Use when [conditions]. Analyzes [what].
allowed-tools: Read, Grep, Glob
---

# Skill Documentation
[Full guide with examples]
```

### Key Components

1. **YAML Frontmatter**
   - `name`: Unique identifier
   - `description`: Activation triggers + purpose (critical for model-invoked activation)
   - `allowed-tools`: Limited to safe, fast operations

2. **Trigger Keywords** (in description)
   - "Use when files modified"
   - "Analyzes code style"
   - "Triggers on git diff"
   - These keywords activate the skill when mentioned in context

3. **Tool Restrictions**
   - Read, Write, Edit, Grep, Glob (safe subset)
   - NO Bash, Task, WebFetch (expensive operations)
   - Ensures fast, non-blocking execution

## 8 Core Skills Analyzed

### Development Skills

#### 1. code-reviewer
**Purpose**: Real-time code quality checks
**Triggers**: File edits, saves, git diff, "code quality" mentioned
**Checks**: Style, anti-patterns, basic security, imports, unused variables
**Output**: Quick wins (< 1 second analysis)

**Key Pattern**: Defers to @code-reviewer sub-agent for deep analysis
- Skill: "⚠️ Potential N+1 query detected"
- User: "@code-reviewer explain N+1 issue"
- Sub-agent: [Comprehensive analysis with examples]

**Applicability**: Our `learner` skill should follow this pattern - detect success, suggest pattern capture, defer to detailed logging

#### 2. test-generator
**Purpose**: Suggest missing tests automatically
**Triggers**: New functions, untested code mentioned
**Checks**: Coverage gaps, missing edge cases
**Output**: Test suggestions without writing code

**Key Pattern**: Lightweight detection, detailed generation deferred to `/test-gen` command or @test-engineer agent

**Applicability**: Our `spin-detector` skill should similarly detect (lightweight) but defer resolution to agent

#### 3. git-commit-helper
**Purpose**: Generate conventional commit messages
**Triggers**: `git diff --staged`, "commit" mentioned
**Checks**: Staged changes, conventional commit format
**Output**: Suggested commit message

**Key Pattern**: Reads git state, generates suggestion, doesn't execute
**Applicability**: Our `cleaner` skill should read `.plan/` state and suggest archiving, not auto-execute

### Security Skills

#### 4. security-auditor
**Purpose**: OWASP Top 10 vulnerability scanning
**Triggers**: Auth code, API endpoints, database queries
**Checks**: SQL injection, XSS, CSRF, exposed secrets
**Output**: Security warnings with severity

**Key Insight**: Overlaps with @security-auditor agent BUT:
- Skill: Quick scan for obvious issues (hardcoded secrets, SQL injection patterns)
- Agent: Deep security audit with attack scenarios

**Applicability**: Our skills should have similar overlap with agents - quick check vs deep analysis

#### 5. secret-scanner
**Purpose**: Detect exposed secrets before commit
**Triggers**: Pre-commit, API keys in code
**Checks**: Patterns matching API keys, tokens, passwords
**Output**: Blocks commit with exposed secrets

**Key Pattern**: Preventative - stops bad action before it happens
**Applicability**: Our `spin-detector` skill should preventatively stop AI from continuing stuck pattern

#### 6. dependency-auditor
**Purpose**: Check dependencies for CVEs
**Triggers**: package.json changes, dependency updates
**Checks**: Known vulnerabilities in dependencies
**Output**: CVE warnings with severity

**Applicability**: Less relevant for our use case (we're not managing dependencies)

### Documentation Skills

#### 7. api-documenter
**Purpose**: Auto-generate OpenAPI specs from code
**Triggers**: API routes added, endpoint modified
**Checks**: API endpoint structure, parameters, responses
**Output**: OpenAPI spec updates

**Key Pattern**: Automatic documentation maintenance
**Applicability**: Our system should auto-update `.ai-knowledge/` documentation similarly

#### 8. readme-updater
**Purpose**: Keep README current with changes
**Triggers**: Project changes, new features added
**Checks**: README outdated sections
**Output**: README update suggestions

**Applicability**: Our `learner` skill should auto-update knowledge base docs when patterns change

## Activation Mechanism

### How Skills Activate

1. **Model-Invoked**: Claude decides when to activate based on context matching description
2. **Trigger Keywords**: Specific phrases in description enable activation
3. **Context Sharing**: Skills share conversation context (efficient)
4. **Non-Blocking**: Multiple skills can activate simultaneously

### Example Activation Flow

```
User writes code:
function getUser(id) {
  return db.query('SELECT * FROM users WHERE id = ' + id)
}

Context contains: "code", "function", "database", "query"

Skills that activate:
1. code-reviewer - "files modified" matches
2. security-auditor - "database queries" matches
3. test-generator - "new functions" matches

All provide feedback simultaneously without blocking
```

## Critical Patterns for Our Project

### Pattern 1: Lightweight Detection + Deep Analysis Separation

**Tresor Approach**:
- Skill: Quick scan, flag issue
- Sub-Agent: Comprehensive analysis, examples, recommendations
- Command: Orchestrate multiple analyses

**Our Application**:
```
learner skill: Detect task completion
@learner agent: Analyze what to capture in .ai-knowledge/
/capture command: Comprehensive knowledge update workflow

spin-detector skill: Detect repetitive pattern (3+ same actions)
@debugger agent: Analyze why AI is stuck
/unstick command: Alternative approach generation
```

### Pattern 2: Tool Restriction Enables Fast Execution

**Tresor Implementation**:
- Skills: Read, Grep, Glob only (< 100ms activation)
- Agents: Full tool access (30s - 5min execution)

**Our Application**:
- Background skills must be fast → limit to safe tools
- Explicit agents can be slower → allow Task, Bash, WebFetch

### Pattern 3: Trigger Keywords Drive Activation

**Tresor Examples**:
- "Use when files modified, saved, or committed"
- "Triggers on git diff, code edits, quality mentions"
- "Analyzes code style, patterns, potential bugs"

**Our Application**:
```yaml
---
name: learner
description: Automatically captures successful patterns after task completion. Use when PR created, commit successful, or user provides positive feedback. Triggers on completion, success, working code.
allowed-tools: Read, Write, Edit, Grep, Glob
---
```

### Pattern 4: Non-Intrusive Feedback

**Tresor Approach**:
- Skills provide suggestions, don't interrupt flow
- User decides whether to act on suggestions
- Severity levels guide prioritization (🚨 CRITICAL, ⚠️ HIGH, 📋 MEDIUM, 💡 LOW)

**Our Application**:
- `spin-detector` should flag issues but not auto-fix
- `learner` should suggest patterns to capture but not auto-commit
- `cleaner` should suggest archiving but not auto-delete

## Skill Implementation Checklist

For implementing our own skills based on Tresor patterns:

### ✅ Structure
- [ ] YAML frontmatter with name, description, allowed-tools
- [ ] Trigger keywords in description for model activation
- [ ] Comprehensive documentation with examples
- [ ] Clear relationship to related sub-agents/commands

### ✅ Behavior
- [ ] Limited tool access (Read, Write, Edit, Grep, Glob)
- [ ] Fast execution (< 1 second target)
- [ ] Non-blocking (provide suggestions, don't interrupt)
- [ ] Severity levels for prioritization

### ✅ Integration
- [ ] Defers to sub-agent for deep analysis
- [ ] Works with commands for orchestrated workflows
- [ ] Shares context efficiently
- [ ] Multiple skills can activate simultaneously

### ✅ Documentation
- [ ] SKILL.md with YAML + full guide
- [ ] README.md with quick reference
- [ ] Examples in action
- [ ] Clear trigger conditions

## Recommended Skills for Optimized AI

### High Priority

#### 1. learner (Background Knowledge Capture)
```yaml
---
name: learner
description: Automatically captures successful patterns after task completion. Use when PR created, commit successful, user provides feedback. Updates .ai-knowledge/ with patterns, preferences, and corrections. Triggers on completion, success, working code.
allowed-tools: Read, Write, Edit, Grep, Glob
---
```

**Why**: Core to self-learning vision, should happen automatically

#### 2. spin-detector (Stuck Pattern Detection)
```yaml
---
name: spin-detector
description: Monitors for repetitive patterns indicating AI is stuck. Use when same file edited 3+ times, same error encountered repeatedly, no progress for 5 minutes. Triggers on repetition, loops, stuck behavior.
allowed-tools: Read, Grep
---
```

**Why**: Critical for preventing token waste, directly addresses auto-detection goal

#### 3. cleaner (Workspace Maintenance)
```yaml
---
name: cleaner
description: Maintains clean workspace by archiving completed tasks. Use when PR created successfully, task marked complete. Archives .plan/ folder, removes temporary files. Triggers on completion, success, PR created.
allowed-tools: Read, Write, Grep, Glob, Bash
---
```

**Why**: Supports clean workspace principle

### Medium Priority

#### 4. config-validator (Configuration Safety)
```yaml
---
name: config-validator
description: Validates configuration changes for safety. Use when .cursorrules, claude.md, or optimized-ai.config.json modified. Checks for risky changes, requires justification. Triggers on config file edits.
allowed-tools: Read, Grep
---
```

**Why**: Prevents configuration-induced performance degradation

### Low Priority

#### 5. test-suggester (Coverage Gaps)
```yaml
---
name: test-suggester
description: Suggests missing tests for new code. Use when new functions added, untested code paths detected. Triggers on code additions, missing test coverage.
allowed-tools: Read, Grep, Glob
---
```

**Why**: Nice to have but not core to optimization goals

## Key Takeaways

1. **Skills = Automatic Helpers**: Always on, no manual invocation required
2. **Trigger Keywords = Activation**: Description text drives when skills activate
3. **Tool Limits = Fast Execution**: Restricted tools enable < 1s latency
4. **Complementary to Agents**: Skills detect, agents analyze deeply
5. **Non-Intrusive**: Suggest, don't interrupt user flow

## Differences from Our Current Plan

| Current Plan | Tresor Pattern | Recommendation |
|--------------|----------------|----------------|
| Learner is an agent | Should be skill | ✅ Change to skill |
| Cleaner is an agent | Should be skill | ✅ Change to skill |
| Spin detection in hooks | Should be skill | ✅ Add skill for real-time detection |
| All agents manual | Mix of skills + agents | ✅ Adopt 3-tier |

---

**Next Steps**:
1. Implement `learner` skill using Tresor SKILL.md pattern
2. Implement `spin-detector` skill with trigger keyword optimization
3. Test skill activation in Phase 0 experimental scenarios
4. Validate tool restrictions don't limit effectiveness

**See Also**:
- [agents/README.md](../agents/README.md) - Sub-agent patterns
- [commands/README.md](../commands/README.md) - Command orchestration
- [patterns/README.md](../patterns/README.md) - Meta patterns
