# CLAUDE.md Research & Best Practices

**Research Date**: 2025-11-03
**Focus**: Comprehensive analysis of CLAUDE.md files aligned with MINIMIZE and VALIDATE principles

---

## Executive Summary

### Key Findings

1. **CLAUDE.md should be < 5,000 tokens** (official recommendation)
2. **Skills are more efficient**: Each skill uses only 30-50 tokens until loaded, vs CLAUDE.md which is loaded in full at session start
3. **Subdirectory loading is BUGGY**: Despite documentation, subdirectory CLAUDE.md files don't load as expected
4. **Context trimming risk**: CLAUDE.md can be lost during /compact operations
5. **Progressive disclosure is key**: Use Skills and Agents for on-demand loading instead of monolithic CLAUDE.md

### Alignment with MINIMIZE Principle

The research validates your core principle: **keep CLAUDE.md tiny and use Skills/Agents for context-specific instructions**. Every line in CLAUDE.md consumes tokens at session start, while Skills only load when needed.

### Critical Validation Required

Before implementing any CLAUDE.md strategy, you MUST validate through experiments:
- Does a 200-line CLAUDE.md perform better than a 50-line core + skills?
- At what point does CLAUDE.md size impact performance?
- How does context compaction affect CLAUDE.md adherence?

---

## Important Update: Main Branch Reorganization

**Date**: 2025-11-03 (after initial research)

The main branch has been reorganized with significant structural changes. This section clarifies how the research aligns with the new structure.

### Main Branch Changes

**File Structure Reorganization:**
```
OLD (research branch):                 NEW (main branch):
├── .plan/initial-design/              ├── .plan/initial-design/
│   ├── CORE-PRINCIPLES.md            │   ├── PLAN.md (new Phase 0 plan)
│   ├── EXPERIMENTAL-VALIDATION.md    │   ├── principles/
│   ├── REVISED-ROADMAP.md            │   │   ├── 1-minimize.md
│   ├── ROADMAP.md                    │   │   ├── 2-separate.md
│   └── ...                           │   │   ├── 3-validate.md
                                       │   │   └── 4-learn.md
                                       │   └── ...
                                       └── experiments/
                                           ├── README.md
                                           ├── EXPERIMENTS-LOG.md
                                           └── LEARNINGS.md
```

**Key Changes:**
1. **Principles split**: Monolithic CORE-PRINCIPLES.md → 4 separate principle files
2. **New experiments/ folder**: Structured location for experimental validation
3. **PLAN.md created**: Detailed Phase 0 implementation plan
4. **Consolidated roadmap**: Multiple roadmap files merged

### .cursorrules vs CLAUDE.md Clarification

**CRITICAL INSIGHT from main branch README:**

> "Key Architecture Change: Single minimal `.cursorrules` file (< 100 lines) with on-demand skill loading. **Cursor auto-includes claude.md**, so just one config to optimize!"

**What this means:**

| Tool | Primary Config | Auto-loaded Files | Notes |
|------|---------------|-------------------|-------|
| **Claude Code** | CLAUDE.md | CLAUDE.md (system memory) | Official Anthropic CLI tool |
| **Cursor** | .cursorrules | .cursorrules + CLAUDE.md | Cursor includes both automatically |
| **Either** | Both work | Depends on tool | CLAUDE.md = memory, .cursorrules = rules |

**Implications for this research:**

1. **This research focuses on CLAUDE.md** (Claude Code's memory system)
2. **Main branch focuses on .cursorrules** (Cursor's rule system)
3. **Both approaches are valid** - same principles apply:
   - Keep core config minimal (< 100-200 lines)
   - Use Skills for on-demand loading
   - Validate everything experimentally

4. **Can use both together:**
   ```
   .cursorrules (< 100 lines):  Core rules and constraints
   CLAUDE.md (< 200 lines):     Project context and patterns
   Skills:                      Domain-specific on-demand instructions
   ```

### Two Skill Systems: Official vs Custom

**IMPORTANT DISTINCTION discovered:**

#### Official Claude Code Skills (this research documents this)

**Location:** `.claude/skills/`
**Format:** SKILL.md with YAML frontmatter + Markdown
**Discovery:** Automatic by Claude Code
**Loading:** Progressive disclosure (30-50 tokens until loaded)
**Documentation:** https://docs.claude.com/en/docs/claude-code/skills

**Example:**
```markdown
---
name: firebase-auth
description: Firebase authentication patterns. Use when implementing login/signup.
allowed-tools: Read,Write,Bash(npm:*)
---

# Firebase Authentication Instructions
[Instructions here]
```

#### Custom .skill System (main branch proposes this)

**Location:** `skills/` (custom)
**Format:** YAML-only .skill files
**Discovery:** Custom loader (to be built)
**Loading:** Custom implementation
**Documentation:** See `.plan/initial-design/principles/2-separate.md`

**Example:**
```yaml
name: firebase-auth
description: Firebase Authentication patterns
triggers:
  keywords: [firebase, auth, authentication]
  files: [firebase.config.*, auth/*.ts]
content: |
  # Firebase auth guidance
  [Instructions here]
scripts:
  check-auth: ./scripts/firebase-check-auth.sh
```

**Key Differences:**

| Feature | Official Claude Code Skills | Custom .skill System (main) |
|---------|---------------------------|---------------------------|
| **Built-in** | ✅ Yes (Claude Code native) | ❌ No (custom build required) |
| **Works today** | ✅ Yes | ❌ No (Phase 2 implementation) |
| **Format** | Markdown + YAML frontmatter | YAML only |
| **Scripts** | Can include scripts in directory | Direct script execution emphasis |
| **Discovery** | Automatic by Claude Code | Custom loader needed |
| **Token efficiency** | 30-50 tokens until loaded | TBD (needs validation) |
| **MCP integration** | Supported | Intentionally avoiding (prefer scripts) |

**Recommendation:**

1. **Short term (Phase 0-1):** Use official Claude Code Skills
   - Works immediately, no build required
   - Token efficiency proven (30-50 tokens)
   - Well-documented and supported

2. **Long term (Phase 2+):** Consider custom system IF:
   - Official skills prove limiting
   - Custom triggers provide measurable benefit
   - Validated through experiments

3. **Pragmatic:** Can use official Skills while evaluating custom approach

### Updated References for Main Branch

**When main branch files are referenced in this research:**

| This Research References | Main Branch Equivalent |
|-------------------------|----------------------|
| `.plan/initial-design/CORE-PRINCIPLES.md` | `.plan/initial-design/principles/1-minimize.md` (and 2,3,4) |
| `.plan/initial-design/EXPERIMENTAL-VALIDATION.md` | `.plan/initial-design/principles/3-validate.md` + `experiments/README.md` |
| `.plan/initial-design/REVISED-ROADMAP.md` | `.plan/initial-design/PLAN.md` |
| N/A | `experiments/EXPERIMENTS-LOG.md` (new) |
| N/A | `experiments/LEARNINGS.md` (new) |

### How This Research Aligns with Main Branch

**Alignment:**

1. ✅ **MINIMIZE principle**: Both target < 100-200 line core config
2. ✅ **SEPARATE principle**: Both propose on-demand skill loading
3. ✅ **VALIDATE principle**: Both require experimental validation
4. ✅ **Progressive disclosure**: Both recognize benefits of loading only what's needed

**Differences:**

1. **Config file focus:**
   - This research: CLAUDE.md (Claude Code memory)
   - Main branch: .cursorrules (Cursor rules)
   - Reality: Both work, use both if needed

2. **Skill system:**
   - This research: Official Claude Code Skills (available now)
   - Main branch: Custom .skill system (Phase 2, needs building)
   - Pragmatic: Start with official, validate need for custom

3. **MCP philosophy:**
   - This research: Documents MCP as official approach
   - Main branch: Prefers direct script execution
   - Best: Validate both approaches experimentally (Exp-007 on main)

**Recommendation:**

- **Use this research for:** Understanding official Claude Code mechanics, CLAUDE.md behavior, official Skills system
- **Use main branch for:** Overall architecture, Phase 0 implementation, experiment structure
- **Validate:** Run experiments comparing official Skills vs future custom .skill system

### Experiments Folder Structure (Main Branch)

**New in main branch:**

```
experiments/
├── README.md              # Experimental philosophy and quick start
├── EXPERIMENTS-LOG.md     # Chronological record (never delete)
├── LEARNINGS.md           # Distilled insights (updated after each exp)
├── scenarios/             # Test scenario definitions
├── results/               # Experiment artifacts
├── runner/                # CLI tool (Phase 0 deliverable)
├── validation/            # Validation engine (Phase 0 deliverable)
└── ab-testing/            # Statistical tools (Phase 0 deliverable)
```

**This aligns with Part 8 of this research** (Validation Experiments), providing the infrastructure to run the experiments proposed in this document.

---

## Part 1: How CLAUDE.md Actually Works

### File Discovery Mechanism

**From official documentation and testing:**

1. **Startup behavior**: Claude Code recursively searches UP from current working directory (CWD) to find CLAUDE.md files
2. **Hierarchy**: Loads files from root → current directory
3. **Multiple files**: Can load multiple CLAUDE.md files from parent directories

**Example directory structure:**
```
/project/
├── CLAUDE.md                    # Always loaded (root)
├── src/
│   ├── CLAUDE.md                # Loaded when in /project/src/
│   └── auth/
│       └── CLAUDE.md            # Should load when in /project/src/auth/
```

### Context Injection Timing

**When CLAUDE.md is loaded:**
- ✅ At session start (automatic)
- ✅ When launching from a directory
- ❌ NOT dynamically when editing files in subdirectories (BUG)

**Where it lives in context:**
- Part of the system prompt/memory
- Supposed to persist through conversation
- **WARNING**: May be lost during /compact operations

### Import System

**CLAUDE.md supports imports:**
```markdown
@path/to/additional-context.md
@~/.claude/my-global-instructions.md
```

**Import rules:**
- Relative and absolute paths supported
- Home directory access via `@~/`
- Recursive imports allowed (max depth: 5)
- Imports inside code blocks are ignored

**Token implication**: Every imported file adds to your baseline token usage!

---

## Part 2: Critical Bugs & Limitations

### 🚨 Subdirectory Loading Bug

**What the documentation says:**
> "When working in /project/src/credits/ or reading files in that directory, Claude should load /project/CLAUDE.md (recursively up) and /project/src/credits/CLAUDE.md (current directory)"

**What actually happens:**
- ✅ Loads /project/CLAUDE.md (parent directories work)
- ❌ Does NOT load /project/src/credits/CLAUDE.md (subdirectory ignored)
- ❌ Requires manual attachment or explicit reference

**GitHub Issues:**
- [#2571](https://github.com/anthropics/claude-code/issues/2571): Subdirectories not loading
- [#3529](https://github.com/anthropics/claude-code/issues/3529): Subfolder CLAUDE.md ignored
- [#4275](https://github.com/anthropics/claude-code/issues/4275): Memory system should discover subdirectories

**Workaround**: Use Skills instead of subdirectory CLAUDE.md files

### 🚨 Context Compaction Loses Instructions

**Reported behavior:**
- CLAUDE.md instructions appear to be lost during /compact operations
- System follows behavioral patterns from training but forgets specific instructions
- Context loss between sessions forces re-establishing project context

**GitHub Issues:**
- [#5502](https://github.com/anthropics/claude-code/issues/5502): System prompt adherence
- [#2954](https://github.com/anthropics/claude-code/issues/2954): Context persistence disruption

**Implication**: You cannot rely on CLAUDE.md to persist indefinitely in long sessions

### 🚨 Optional Guidance Problem

**Reported behavior:**
- Claude treats CLAUDE.md as "optional guidance" rather than strict requirements
- Doesn't consistently follow CLAUDE.md without explicit user prompting
- Loses context when starting new instances

**Workaround**: Use emphasis ("IMPORTANT", "YOU MUST") in CLAUDE.md, but this adds token bloat

---

## Part 3: Real-World Examples

### Example 1: Chorus App (1,200 words, ~1,600 tokens)

**Source**: [GitHub Gist by cbh123](https://gist.github.com/cbh123/75dcd353b354b1eb3398c6d2781a502f)

**Structure:**
```markdown
# What is Chorus?
- Product description
- Tech stack: Tauri, React, TypeScript, TanStack Query, SQLite

# Workflow
- Setup → Development → Review → Issue resolution
- Emphasis: "Commit often"

# Project Structure
- UI layer, Core layer, Tauri layer

# Important Files
- Key file paths with brief descriptions

# Data Model
- Change procedures

# Coding Style
- Standards and constraints
```

**Analysis:**
- ✅ Well-organized with clear sections
- ✅ Scannable with hierarchical headings
- ❌ At 1,600 tokens, this is 32% of recommended 5k limit for ONE file
- ❌ Includes information that could be in separate docs

**Alignment with MINIMIZE**: Could be reduced to 400 tokens with:
- Core principles only in CLAUDE.md
- Project structure → separate doc
- Workflow details → Skills or slash commands

### Example 2: Next.js Stack Guide

**Source**: [GitHub Gist by gregsantos](https://gist.github.com/gregsantos/2fc7d7551631b809efa18a0bc4debd2a)

**Content type**: Comprehensive guide for TypeScript + Next.js + Tailwind + shadcn + React Query

**Analysis:**
- Likely similar length to Chorus example
- Stack-specific patterns and conventions
- Strong candidate for Skills-based approach

**Better approach**:
```
CLAUDE.md (core): 200 tokens
.claude/skills/nextjs/SKILL.md: 1,000 tokens (loaded on-demand)
.claude/skills/shadcn/SKILL.md: 500 tokens (loaded on-demand)
```

### Example 3: Claude-Flow Templates

**Source**: [ruvnet/claude-flow](https://github.com/ruvnet/claude-flow/wiki/CLAUDE-MD-Templates)

**Key insight**: "CLAUDE.md is the heart of Claude-Flow configuration"

**Provides specialized templates for**:
- Different project types
- Different tech stacks
- Different workflows

**Analysis**: Template approach shows ONE SIZE DOES NOT FIT ALL
- ✅ Validates the need for project-specific CLAUDE.md
- ❌ Large templates contradict minimalism
- ✅ Could be adapted to Skills-based architecture

---

## Part 4: Best Practices for Minimal CLAUDE.md

### Official Anthropic Recommendations

**From official docs:**

1. **No required format** - keep concise and human-readable
2. **Recommended sections:**
   - Common bash commands
   - Core files and utilities
   - Code style guidelines
   - Testing instructions
   - Repository etiquette (branch naming, merge strategies)

3. **Refinement is critical:**
   - "A common mistake is adding extensive content without iterating on its effectiveness"
   - Use the `#` key to add instructions that Claude auto-incorporates
   - Periodically run through prompt improvement tools
   - Include changes in commits for team benefit

4. **Use emphasis for critical rules:**
   - "IMPORTANT", "YOU MUST", etc.
   - But beware: this adds tokens!

### Community Best Practices

**From multiple sources:**

1. **Keep under 5,000 tokens** (absolute max)
2. **Target 2,000 tokens** for optimal balance
3. **Link to external docs** for detailed information
4. **Specify boundaries**: Which files Claude can read, which to ignore
5. **Write like a spec**: Clear, direct, structured
6. **Use XML tags for clarity** (but adds tokens)
7. **Bullet points over prose**
8. **Regular maintenance**: Review and update as project evolves

### The 50-Line Challenge (Your Goal)

**Aligned with CORE-PRINCIPLES.md: < 200 lines CLAUDE.md**

**What to include in core CLAUDE.md:**
```markdown
# Project: [Name]

## Stack
- Language: TypeScript
- Infrastructure: Firebase
- Framework: Svelte

## Core Principles
- No React/Tailwind
- Separate design from structure
- Refactor on reuse
- Minimal boilerplate

## Structure
- lib/ - Core logic
- routes/ - Pages
- .plan/ - Git-ignored workspace

## Critical Rules
- Check .ai-knowledge/ before starting tasks
- Work in .plan/ folder
- Self-evaluate before PRs
- Never merge to main
```

**Estimated tokens**: ~200 tokens

**Everything else goes into:**
- `.claude/skills/` - Domain-specific patterns
- `.ai-knowledge/` - Learned patterns
- `docs/` - Reference documentation (loaded on-demand with @)

---

## Part 5: Skills System Deep Dive

### Why Skills Beat Monolithic CLAUDE.md

**Token efficiency comparison:**

| Approach | Baseline Tokens | Per-Task Tokens | Total (10 tasks) |
|----------|----------------|-----------------|------------------|
| Monolithic CLAUDE.md (5,000 tokens) | 5,000 | 0 | 50,000 |
| Core CLAUDE.md (500 tokens) + Skills | 500 | +1,500/task | 15,500 |
| **Savings** | | | **69%** |

**Why this works:**
- Skills only load when needed
- Each skill: 30-50 tokens until loaded
- Full content: ~1,500 tokens when active
- Most tasks only need 1-2 skills

### How Skills Work Internally

**Progressive disclosure mechanism:**

1. **Metadata phase**: All skill names + descriptions in system prompt (~30-50 tokens each)
2. **Selection phase**: Claude reads descriptions, decides which skill to invoke
3. **Loading phase**: Full SKILL.md injected into context (~1,500 tokens)
4. **Execution phase**: Claude follows skill instructions
5. **Resource phase**: Scripts and references loaded only if needed

**Technical architecture:**
- Skills are a meta-tool, not executable code
- Appears in `tools` array alongside Read, Write, Bash
- Two-message injection:
  - Message 1 (visible): `<command-message>The "pdf" skill is loading</command-message>`
  - Message 2 (hidden): Full skill prompt with `isMeta: true`

### Skills Directory Structure

**Location:**
- Personal: `~/.claude/skills/` (available across all projects)
- Project: `.claude/skills/` (checked into git, team-shared)

**Basic structure:**
```
.claude/skills/
└── firebase-auth/
    ├── SKILL.md              # Core prompt (500-5,000 words)
    ├── scripts/              # Python/Bash automation
    ├── references/           # Docs loaded into context
    └── assets/               # Templates, referenced by path
```

**SKILL.md format:**
```markdown
---
name: firebase-auth
description: Guides Firebase authentication implementation. Use when implementing auth, login, or user management.
allowed-tools: Read,Write,Bash(npm:*),Bash(git:*)
model: sonnet
---

# Firebase Authentication Implementation

## When to Use
- Implementing user authentication
- Setting up Firebase Auth
- Managing user sessions

## Instructions
1. Install Firebase SDK
2. Initialize Firebase config
3. Implement auth methods
...

## Examples
[Include code examples]
```

### Skills Best Practices

**From official sources and community:**

1. **Keep skills focused**: One capability per skill
2. **Write specific descriptions**: Include activation triggers
3. **Separate supporting materials**: Don't bloat SKILL.md
4. **Use allowed-tools**: Restrict tool access for safety
5. **Target 500-2,000 words**: Sweet spot for effectiveness
6. **Include examples**: Show expected patterns
7. **Use imperative language**: "Extract data..." not "You should extract..."
8. **Reference with {baseDir}**: Never hardcode paths

### Token Budget Constraints

**15,000-character limit for skill listings:**
- This is for ALL skill descriptions combined
- Forces concise descriptions
- Skills exceeding budget are truncated or excluded
- Average: ~200 characters per skill = ~75 skills max

---

## Part 6: Agents System Deep Dive

### Agents vs Skills

| Feature | Skills | Agents |
|---------|--------|--------|
| **Purpose** | Teach HOW to do tasks | Delegate WHO does tasks |
| **Context** | Modifies main agent context | Fresh, isolated context |
| **Token impact** | Adds to main conversation | Separate conversation |
| **Best for** | Repeatable patterns | Complex multi-step tasks |
| **Execution** | Main agent follows skill instructions | Subagent executes independently |

### When to Use Agents

**Use subagents for:**
- Complex multi-step tasks
- Tasks requiring different skills/context
- When main context is getting bloated
- Tasks with clear handoff points
- When parallelization is possible

**Example workflow:**
```
Main Agent (PM):
  → Reads task
  → Analyzes requirements

  → Spawns: Planner Subagent
      Fresh context + planning.skill
      Creates detailed plan
      Returns plan

  → Spawns: Implementer Subagent
      Fresh context + relevant implementation skills
      Writes code
      Returns code

  → Spawns: Reviewer Subagent
      Fresh context + review.skill
      Reviews code
      Returns feedback

  → Integrates results
```

### Agents Directory Structure

**Location:**
- Personal: `~/.claude/agents/`
- Project: `.claude/agents/` (takes precedence, versioned)

**File format:**
```markdown
---
name: laravel-planner
description: Senior Laravel architect who creates implementation plans. Use when planning Laravel features.
tools: Read,Grep,Glob
model: sonnet
---

You are a Senior Laravel architect specializing in planning implementations.

## Your Role
- Analyze requirements
- Create step-by-step implementation plans
- Consider Laravel best practices
- Identify potential issues

## Output Format
Provide a numbered list of implementation steps with:
1. File to create/modify
2. Specific changes needed
3. Testing considerations

## Constraints
- Do NOT edit files (you're a planner, not implementer)
- Do NOT run commands
- Focus on planning only
```

### Agent Configuration Options

**Model selection:**
- `model: sonnet` - Use Sonnet (fast, cheaper)
- `model: opus` - Use Opus (highest quality)
- `model: haiku` - Use Haiku (fastest, cheapest)
- `model: inherit` - Use same model as main conversation
- Omit field: Use default model

**Tools:**
- Omit field: Inherit all tools (default)
- Specify: `tools: Read,Write,Grep` (restrict tools)

### Community Agent Collections

**Available resources:**
- 100+ production-ready agents (0xfurai/claude-code-subagents)
- 60+ specialized agents (lst97/claude-code-sub-agents)
- Organized by domain: Development, Language, Infrastructure, Business, Marketing

---

## Part 7: Recommendations for Optimized AI Project

### Core Architecture

**Based on CORE-PRINCIPLES.md goals:**

```
Core (Always Loaded):
├── CLAUDE.md (< 200 lines, ~500 tokens)
│   ├── Project overview
│   ├── Tech stack
│   ├── Core principles
│   └── Critical rules
│
Skills (Load on-demand):
├── .claude/skills/firebase/
│   ├── auth.skill
│   ├── firestore.skill
│   └── functions.skill
├── .claude/skills/patterns/
│   ├── testing.skill
│   ├── refactoring.skill
│   └── performance.skill
└── .claude/skills/frameworks/
    ├── svelte.skill
    └── ionic.skill
│
Agents (For complex tasks):
├── .claude/agents/
│   ├── planner.agent
│   ├── implementer.agent
│   ├── reviewer.agent
│   └── learner.agent
│
Knowledge (Learned patterns):
└── .ai-knowledge/
    ├── patterns.json
    ├── anti-patterns.json
    ├── skill-usage.json
    └── optimization-metrics.json
```

### Minimal CLAUDE.md Template

**For Optimized AI project specifically:**

```markdown
# Optimized AI - Self-Learning Coding Assistant

## Stack
- **Language**: TypeScript (strict mode)
- **Infrastructure**: Firebase, Supabase
- **Frameworks**: Svelte, HTMX, Ionic+Angular
- **Testing**: Vitest
- **Runtime**: Node.js, Deno

## Core Principles
1. **MINIMIZE**: < 250 line configs, prove every line's value
2. **SEPARATE**: Load skills on-demand, use subagents for isolation
3. **VALIDATE**: Every claim backed by experiments
4. **LEARN**: System optimizes itself over time

## Project Structure
- `.plan/` - Current work (git-ignored)
- `.ai-knowledge/` - Learned patterns (git-tracked)
- `.claude/` - Skills and agents
- `research/` - Experimental findings

## Critical Rules
- ✅ Check `.ai-knowledge/` before starting any task
- ✅ Work in `.plan/` folder, keep workspace clean
- ✅ Use IDE operations over manual refactoring
- ✅ Self-evaluate before creating PRs
- ✅ Only commit passing code
- ❌ NEVER merge to main automatically
- ❌ NEVER use React or Tailwind
- ❌ NEVER spin without detecting and switching approaches

## Workflow
1. Read task from `.plan/current-task.md`
2. Check `.ai-knowledge/patterns.json` for similar tasks
3. Load relevant skills (e.g., `@firebase` for auth work)
4. Create plan in `.plan/approach.md`
5. Implement following learned patterns
6. Self-evaluate continuously
7. Create PR when all checks pass
8. Update `.ai-knowledge/` with learnings

## References
- Core principles: @.plan/initial-design/CORE-PRINCIPLES.md
- Validation framework: @.plan/initial-design/EXPERIMENTAL-VALIDATION.md
- Global learnings: @~/.optimized-ai/global-knowledge/patterns.md
```

**Token count**: ~650 tokens (well under 5k limit)

### Skills to Create

**Priority 1 (Core Patterns):**
```
.claude/skills/
├── firebase/
│   ├── auth.skill          - Firebase authentication patterns
│   ├── firestore.skill     - Query patterns, indexing
│   └── security-rules.skill - Security rules validation
├── supabase/
│   ├── rls.skill           - Row Level Security patterns
│   └── queries.skill       - Supabase query optimization
├── testing/
│   ├── unit.skill          - Unit test patterns
│   ├── integration.skill   - Integration test patterns
│   └── edge-cases.skill    - Edge case identification
└── patterns/
    ├── refactoring.skill   - When and how to refactor
    ├── performance.skill   - Performance optimization
    └── spin-detection.skill - Self-monitoring for spinning
```

**Priority 2 (Framework Specific):**
```
.claude/skills/
├── svelte/
│   ├── components.skill    - Svelte component patterns
│   └── stores.skill        - Svelte store management
├── htmx/
│   └── interactions.skill  - HTMX patterns
└── typescript/
    ├── types.skill         - TypeScript type patterns
    └── strict-mode.skill   - Strict mode best practices
```

### Agents to Create

**Workflow agents:**
```
.claude/agents/
├── planner.agent           - Breaks down tasks into steps
├── implementer.agent       - Writes code following patterns
├── tester.agent            - Creates and runs tests
├── reviewer.agent          - Self-reviews code quality
├── cleaner.agent           - Cleans up workspace
└── learner.agent           - Updates knowledge base
```

### Migration Strategy

**Phase 0: Baseline Measurement**
1. Create minimal CLAUDE.md (200 lines)
2. Run 10 test scenarios
3. Measure: tokens, time, quality, spin rate
4. Document baseline in `.ai-knowledge/metrics.json`

**Phase 1: Skills Implementation**
1. Create 3 core skills (firebase-auth, testing, refactoring)
2. Run same 10 scenarios with skills
3. Compare: tokens, time, quality, spin rate
4. VALIDATE: Does skills approach improve metrics?

**Phase 2: Agents Implementation**
1. Create 3 core agents (planner, implementer, reviewer)
2. Run 5 complex multi-step scenarios
3. Compare: quality, context pollution, clarity
4. VALIDATE: Do agents improve complex task handling?

**Phase 3: Optimization**
1. Track skill usage frequency
2. Merge rarely-used skills
3. Split frequently-used skills if they're bloated
4. Remove unused instructions from CLAUDE.md
5. VALIDATE: Measure improvement

---

## Part 8: Validation Experiments

### Experiment 1: Monolithic vs Modular

**Hypothesis**: "Minimal CLAUDE.md + Skills reduces token usage by 60% without degrading quality"

**Design:**
- Control: 500-line CLAUDE.md with all instructions
- Treatment: 50-line CLAUDE.md + 5 skills
- Scenarios: 10 tasks requiring different skills
- Metrics: tokens/task, time/task, quality score, spin rate

**Success criteria:**
- ✅ 60%+ token reduction
- ✅ Equal or better quality
- ✅ No increase in spin rate
- ✅ Same or faster completion time

**If fails**: Analyze which skills are loaded too often (merge into core)

### Experiment 2: Context Persistence

**Hypothesis**: "CLAUDE.md instructions persist through /compact operations"

**Design:**
- Create CLAUDE.md with 10 specific instructions
- Run long task requiring /compact
- After compaction, test adherence to each instruction
- Measure: How many instructions are still followed?

**Success criteria:**
- ✅ 100% instruction adherence after compaction
- ✅ No degradation in quality

**If fails**: Document in knowledge base, explore alternatives (Skills? Hooks?)

### Experiment 3: Subdirectory Loading

**Hypothesis**: "Subdirectory CLAUDE.md files load when editing files in those directories"

**Design:**
- Create structure:
  ```
  /project/CLAUDE.md (instruction A)
  /project/lib/CLAUDE.md (instruction B)
  /project/lib/auth/CLAUDE.md (instruction C)
  ```
- Test: Edit file in /project/lib/auth/
- Verify: Which instructions does Claude follow?

**Expected result (per docs)**: All three instructions
**Actual result (per bugs)**: Only instruction A

**Action**: Document bug, use Skills instead of subdirectory CLAUDE.md

### Experiment 4: Skill Loading Overhead

**Hypothesis**: "Skill loading adds < 2s overhead per invocation"

**Design:**
- Measure time to:
  1. Start task with monolithic CLAUDE.md
  2. Start task with Skills (measure from invocation to loaded)
- Run 20 times each
- Statistical comparison

**Success criteria:**
- ✅ Skill loading adds < 2s
- ✅ Perception: Not noticeably slower

**If fails**: Investigate caching, pre-loading strategies

### Experiment 5: Optimal CLAUDE.md Size

**Hypothesis**: "50-line CLAUDE.md is sufficient for core instructions"

**Design:**
- Test sizes: 25, 50, 100, 200, 500 lines
- Same 10 test scenarios for each
- Measure: Quality, tokens, adherence

**Find the knee in the curve**: Where does adding more lines stop improving quality?

**Action**: Set that as your target size

---

## Part 9: Critical Issues Summary

### ⚠️ Subdirectory Loading Doesn't Work

**Status**: Known bug, multiple open issues
**Impact**: Cannot organize CLAUDE.md files by directory
**Workaround**: Use Skills instead
**Validation**: Experiment 3 (above)

### ⚠️ Context Compaction Loses Instructions

**Status**: Reported issue
**Impact**: Long sessions may lose CLAUDE.md adherence
**Workaround**: Periodically remind Claude of critical rules
**Validation**: Experiment 2 (above)

### ⚠️ Optional Guidance Problem

**Status**: Behavioral issue
**Impact**: Claude treats CLAUDE.md as suggestions, not requirements
**Workaround**: Use strong emphasis ("MUST", "NEVER")
**Cost**: Additional tokens for emphasis

### ⚠️ Token Budget Unknown

**Status**: No official documentation on limits
**Impact**: Don't know exact token count for CLAUDE.md
**Recommendation**: Assume ~4 tokens per word, measure with /context command

---

## Part 10: Action Items for Optimized AI

### Immediate Actions

1. **Create minimal CLAUDE.md** (< 200 lines)
   - Use template from Part 7
   - Focus on core principles only
   - Link to external docs with @imports

2. **DO NOT create subdirectory CLAUDE.md files**
   - Subdirectory loading is broken
   - Use Skills instead

3. **Set up Skills directory structure**
   - Create `.claude/skills/`
   - Plan out 8-10 core skills
   - Start with 3 highest-priority skills

4. **Set up Agents directory structure**
   - Create `.claude/agents/`
   - Plan out 6 workflow agents
   - Start with planner and implementer

5. **Create baseline measurement**
   - Before implementing Skills/Agents
   - Measure current performance
   - Document in `.ai-knowledge/metrics.json`

### Validation Experiments to Run

**Week 1-2:**
- Experiment 1: Monolithic vs Modular
- Experiment 3: Subdirectory Loading (confirm bug)
- Experiment 5: Optimal CLAUDE.md Size

**Week 3-4:**
- Experiment 2: Context Persistence
- Experiment 4: Skill Loading Overhead

**Week 5-6:**
- Custom experiments based on your specific workflow
- Measure learned patterns effectiveness

### Research Gaps to Fill

1. **Token counting**: Exact formula for CLAUDE.md token usage
2. **Import performance**: Do imports slow session startup?
3. **Skill selection accuracy**: How often does Claude choose the right skill?
4. **Agent overhead**: Cost/benefit of agent spawning
5. **Knowledge base integration**: How to surface learnings to Claude

---

## Part 11: Resources & References

### Official Documentation

- [Claude Code Settings](https://docs.claude.com/en/docs/claude-code/settings.md)
- [Claude Code Memory](https://docs.claude.com/en/docs/claude-code/memory.md)
- [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills.md)
- [Subagents](https://docs.claude.com/en/docs/claude-code/sub-agents.md)
- [Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)

### Example Repositories

**CLAUDE.md Examples:**
- [ArthurClune/claude-md-examples](https://github.com/ArthurClune/claude-md-examples)
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- [ruvnet/claude-flow](https://github.com/ruvnet/claude-flow/wiki/CLAUDE-MD-Templates)

**Skills Collections:**
- [anthropics/skills](https://github.com/anthropics/skills) (official examples)
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)

**Agents Collections:**
- [0xfurai/claude-code-subagents](https://github.com/0xfurai/claude-code-subagents) (100+ agents)
- [lst97/claude-code-sub-agents](https://github.com/lst97/claude-code-sub-agents) (60+ agents)
- [wshobson/agents](https://github.com/wshobson/agents)

**Best Practices Guides:**
- [awattar/claude-code-best-practices](https://github.com/awattar/claude-code-best-practices)
- [zebbern/claude-code-guide](https://github.com/zebbern/claude-code-guide)
- [wesammustafa/Claude-Code-Everything](https://github.com/wesammustafa/Claude-Code-Everything-You-Need-to-Know)

### GitHub Issues (Known Bugs)

- [#2571](https://github.com/anthropics/claude-code/issues/2571): Subdirectories not loading
- [#3529](https://github.com/anthropics/claude-code/issues/3529): Subfolder CLAUDE.md ignored
- [#4275](https://github.com/anthropics/claude-code/issues/4275): Memory system subdirectories
- [#5502](https://github.com/anthropics/claude-code/issues/5502): System prompt adherence
- [#2954](https://github.com/anthropics/claude-code/issues/2954): Context persistence
- [#7533](https://github.com/anthropics/claude-code/issues/7533): Context preservation priority

### Technical Deep Dives

- [Claude Skills: First Principles](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)
- [Inside Claude Code Skills](https://mikhail.io/2025/10/claude-code-skills/)
- [Managing Context](https://www.cometapi.com/managing-claude-codes-context/)
- [Token Optimization](https://gist.github.com/artemgetmann/74f28d2958b53baf50597b669d4bce43)

---

## Part 12: Conclusion

### Key Takeaways

1. **CLAUDE.md should be minimal**: < 5,000 tokens absolute max, target < 1,000 tokens
2. **Skills are more efficient**: 30-50 tokens until loaded vs full CLAUDE.md at startup
3. **Subdirectory loading is broken**: Use Skills instead of subdirectory organization
4. **Context compaction risks losing instructions**: Design for this limitation
5. **Progressive disclosure is the winning strategy**: Load only what's needed, when it's needed

### Alignment with MINIMIZE Principle

Your goal of **< 200 line CLAUDE.md** is well-supported by:
- Official recommendation: "keep concise"
- Community consensus: < 5k tokens (your target is ~500 tokens)
- Skills architecture: Enables separation of concerns
- Real-world pain: Large CLAUDE.md files cause token bloat

### Alignment with VALIDATE Principle

Every recommendation in this document must be validated:
- ✅ Run Experiment 1: Measure actual token savings
- ✅ Run Experiment 2: Verify context persistence
- ✅ Run Experiment 3: Confirm subdirectory bug
- ✅ Run Experiment 5: Find optimal size for YOUR workflow

**Do not assume these findings apply to your specific use case without validation.**

### Next Steps

1. ✅ Read this document thoroughly
2. ✅ Create minimal CLAUDE.md using template in Part 7
3. ✅ Set up `.claude/skills/` and `.claude/agents/` directories
4. ✅ Run baseline measurements (Phase 0)
5. ✅ Implement Skills (Phase 1)
6. ✅ Run validation experiments
7. ✅ Document findings in `.ai-knowledge/`
8. ✅ Iterate based on data

### Final Recommendation

**For Optimized AI project:**

```
CLAUDE.md: 50-100 lines core instructions (~200-500 tokens)
Skills: 8-10 focused skills (~400-500 tokens each when loaded)
Agents: 6 workflow agents (fresh context per agent)
Knowledge: .ai-knowledge/ for learned patterns

Expected savings: 60-70% token reduction vs monolithic approach
Validation required: Yes, through experiments
```

This architecture aligns perfectly with your MINIMIZE and VALIDATE principles while working within Claude Code's actual capabilities and limitations.

---

## Part 13: Path Import Clarifications & Workarounds

### Import Path Syntax Explained

**Critical clarification needed from the research:**

| Syntax | Meaning | Example | Use Case |
|--------|---------|---------|----------|
| `@~/` | User's home directory | `@~/.claude/my-settings.md` | Personal preferences not in repo |
| `@` | Project root (relative) | `@docs/api.md` | Project documentation |
| `@./` | Project root (explicit relative) | `@./lib/README.md` | Same as `@` |

**Important distinctions:**

1. **`@~/` is NOT project root** - it's the user's home directory (e.g., `/home/username/.claude/`)
   - Use for: Personal instructions that shouldn't be in the repo
   - Example: `@~/.claude/personal-coding-style.md`
   - Team benefit: Each developer can have personal preferences without conflicts

2. **`@` and `@./` are project-relative** - relative to project root, NOT to the CLAUDE.md file doing the importing
   - Use for: Project documentation, shared instructions
   - Example: `@docs/architecture.md` loads from `/project/docs/architecture.md`
   - Works from any CLAUDE.md in the project

3. **Relative path resolution bug (#4754)**: Imports from user's `~/.claude/CLAUDE.md` sometimes fail because they're resolved relative to CWD instead of relative to the importing file
   - Status: Known bug on Linux systems
   - Workaround: Use absolute paths in `~/.claude/CLAUDE.md`

### Example Import Usage

**In `/project/CLAUDE.md`:**
```markdown
# Project Configuration

@README.md
@docs/coding-standards.md
@docs/git-workflow.md
@~/.claude/personal-preferences.md

## Core Principles
...
```

**Token impact:**
- Each import adds the full content of that file to your context
- Recursive imports allowed (max depth: 5)
- Check loaded files with `/memory` command

---

## Part 14: Subdirectory Bug Workarounds

### The Problem (Detailed)

**What should work according to docs:**
```
/project/
├── CLAUDE.md                    # Loaded ✅
├── lib/
│   ├── CLAUDE.md                # Should load when in lib/ ❌
│   └── auth/
│       ├── CLAUDE.md            # Should load when in auth/ ❌
│       └── login.ts
```

**When editing `/project/lib/auth/login.ts`:**
- ✅ SHOULD load: `/project/CLAUDE.md`, `/project/lib/CLAUDE.md`, `/project/lib/auth/CLAUDE.md`
- ❌ ACTUALLY loads: Only `/project/CLAUDE.md`

### Workaround Option 1: Explicit @imports

**Strategy**: Import subdirectory CLAUDE.md files in your root CLAUDE.md

**Root `/project/CLAUDE.md`:**
```markdown
# Project Core Configuration

## Core Rules
[Core instructions here]

## Domain-Specific Instructions
@lib/CLAUDE.md
@lib/auth/CLAUDE.md
@components/CLAUDE.md
```

**Pros:**
- ✅ Simple to implement
- ✅ All context loaded at session start
- ✅ No need to remember to load manually

**Cons:**
- ❌ Loads ALL subdirectory context even if not needed
- ❌ Defeats the purpose of subdirectory organization
- ❌ Token bloat (all files loaded upfront)
- ❌ Goes against MINIMIZE principle

**Verdict**: Not recommended for large projects

### Workaround Option 2: Explicit Read Instructions

**Strategy**: Add instructions in root CLAUDE.md to read subdirectory CLAUDE.md when working in those areas

**Root `/project/CLAUDE.md`:**
```markdown
# Project Configuration

## Domain-Specific Instructions

When working on authentication code (in `lib/auth/`):
- IMPORTANT: Read and follow `@lib/auth/CLAUDE.md` before starting

When working on database code (in `lib/db/`):
- IMPORTANT: Read and follow `@lib/db/CLAUDE.md` before starting

When working on API endpoints (in `routes/api/`):
- IMPORTANT: Read and follow `@routes/api/CLAUDE.md` before starting
```

**Pros:**
- ✅ Explicit instructions remind Claude to check subdirectory files
- ✅ Only loads context when needed
- ✅ Simple to implement

**Cons:**
- ❌ Relies on Claude following instructions (not always reliable)
- ❌ Adds overhead to root CLAUDE.md
- ❌ User might need to prompt Claude explicitly

**Verdict**: Reasonable temporary workaround

### Workaround Option 3: Use Skills Instead (RECOMMENDED)

**Strategy**: Don't use subdirectory CLAUDE.md files at all - use Skills system

**Instead of:**
```
/project/
├── CLAUDE.md
├── lib/
│   ├── auth/
│   │   └── CLAUDE.md            # Don't do this
```

**Do this:**
```
/project/
├── CLAUDE.md
└── .claude/
    └── skills/
        └── auth/
            └── SKILL.md         # Do this instead
```

**SKILL.md content:**
```markdown
---
name: auth-patterns
description: Authentication implementation patterns. Use when implementing login, signup, password reset, or session management.
allowed-tools: Read,Write,Bash(npm:*)
---

# Authentication Patterns

## When to Use
- Implementing user authentication
- Working on login/signup flows
- Managing sessions
- Password reset functionality

## Instructions
[Your auth-specific instructions here]
```

**Pros:**
- ✅ Works with current Claude Code (no bugs)
- ✅ Only loads when needed (30-50 tokens until invoked)
- ✅ Aligns with MINIMIZE principle
- ✅ Better organization and discoverability
- ✅ Can restrict tool access per skill

**Cons:**
- ❌ Different mental model (skills vs subdirectories)
- ❌ Requires learning Skills system

**Verdict**: STRONGLY RECOMMENDED - most reliable and efficient

### Workaround Option 4: SessionStart Hooks

**Strategy**: Use hooks to inject context based on current working directory

**`.claude/hooks/session-start.sh`:**
```bash
#!/bin/bash

# Check current directory and load relevant CLAUDE.md
if [[ "$PWD" == *"/lib/auth"* ]]; then
    echo "Loading auth-specific instructions from lib/auth/CLAUDE.md"
    cat lib/auth/CLAUDE.md
fi

if [[ "$PWD" == *"/lib/db"* ]]; then
    echo "Loading database-specific instructions from lib/db/CLAUDE.md"
    cat lib/db/CLAUDE.md
fi
```

**Pros:**
- ✅ Automatic based on CWD
- ✅ Only loads when in relevant directory
- ✅ Transparent to user

**Cons:**
- ❌ Requires hooks knowledge
- ❌ Adds complexity
- ❌ May not trigger when editing files from different directory
- ❌ Experimental approach

**Verdict**: Interesting but unproven

### Recommended Approach for Optimized AI

**For your project, use a hybrid approach:**

1. **Root CLAUDE.md** (< 200 lines): Core principles only
2. **Skills** (not subdirectory CLAUDE.md): Domain-specific patterns
3. **Agents**: Complex workflows
4. **@imports**: For existing docs (README, architecture, etc.)

**Example structure (with main branch references):**
```
/project/
├── CLAUDE.md                    # Core (200 lines, ~500 tokens)
│   ├── @README.md               # Import existing docs
│   ├── @.plan/initial-design/principles/1-minimize.md
│   └── Core instructions
│
├── .cursorrules                 # Optional: Cursor-specific rules (< 100 lines)
│
├── .claude/
│   ├── skills/                  # Official Claude Code Skills (works now)
│   │   ├── firebase/
│   │   │   ├── auth/
│   │   │   │   └── SKILL.md     # Instead of lib/firebase/CLAUDE.md
│   │   │   └── firestore/
│   │   │       └── SKILL.md     # Instead of lib/firebase/db/CLAUDE.md
│   │   └── patterns/
│   │       └── testing/
│   │           └── SKILL.md     # Instead of tests/CLAUDE.md
│   │
│   └── agents/
│       ├── planner.agent
│       └── implementer.agent
│
├── skills/                      # Custom .skill system (main branch, Phase 2)
│   ├── firebase-auth.skill      # Custom YAML format (future)
│   └── testing.skill            # Custom YAML format (future)
│
├── .ai-knowledge/               # Learned patterns (git-tracked)
│   └── patterns.json
│
└── experiments/                 # Validation infrastructure (main branch)
    ├── README.md
    ├── EXPERIMENTS-LOG.md
    └── LEARNINGS.md
```

---

## Part 15: Bug Tracking & TODO

### Known Bugs with GitHub Issues

**Track these issues - they affect your architecture decisions:**

| Issue # | Title | Impact | Workaround | Status |
|---------|-------|--------|------------|--------|
| [#2571](https://github.com/anthropics/claude-code/issues/2571) | CLAUDE.md in subdirectories not loaded | HIGH | Use Skills instead | Open |
| [#3529](https://github.com/anthropics/claude-code/issues/3529) | Subfolder CLAUDE.md ignored | HIGH | Use Skills instead | Open |
| [#4275](https://github.com/anthropics/claude-code/issues/4275) | Memory system should discover subdirectories | HIGH | Use Skills instead | Open |
| [#4754](https://github.com/anthropics/claude-code/issues/4754) | Relative imports from user CLAUDE.md fail | MEDIUM | Use absolute paths | Open |
| [#5502](https://github.com/anthropics/claude-code/issues/5502) | System prompt adherence issues | MEDIUM | Use emphasis keywords | Open |
| [#2954](https://github.com/anthropics/claude-code/issues/2954) | Context persistence across sessions | MEDIUM | Document critical rules | Open |
| [#722](https://github.com/anthropics/claude-code/issues/722) | Documentation inconsistent | LOW | Rely on testing | Open |
| [#2901](https://github.com/anthropics/claude-code/issues/2901) | Violates explicit instructions | MEDIUM | Use strong language | Open |

### Revisit Schedule

**Add to your project's maintenance tasks:**

```markdown
## TODO: Review Claude Code Bug Status

**Frequency**: Quarterly or when Claude Code releases major updates

**Tickets to check:**
- [ ] #2571 - Subdirectory loading
- [ ] #3529 - Subfolder CLAUDE.md
- [ ] #4275 - Memory system subdirectories
- [ ] #4754 - Relative path imports

**When a bug is fixed:**
1. Update this research document (research/claude/CLAUDE.md)
2. Re-evaluate workarounds - can we simplify?
3. Run validation experiments with new functionality
4. Update .claude/ structure if beneficial
5. Document changes in .ai-knowledge/

**How to check:**
- Visit: https://github.com/anthropics/claude-code/issues
- Filter by issue numbers
- Check status and recent comments
- Test functionality if closed
```

### Workaround Deprecation Strategy

**When subdirectory loading is fixed:**

1. **Assess impact**:
   - Does the fix work reliably?
   - Does it introduce new issues?
   - Is it better than Skills approach?

2. **Compare approaches**:
   ```
   Option A: Keep Skills (proven, efficient)
   Option B: Move to subdirectory CLAUDE.md (native support)
   Option C: Hybrid approach
   ```

3. **Run experiments**:
   - Measure token usage: Skills vs subdirectory CLAUDE.md
   - Test reliability: Does subdirectory loading work 100%?
   - Evaluate DX: Which is easier for your team?

4. **Make data-driven decision**:
   - Skills may STILL be better even when bug is fixed
   - Progressive disclosure is inherently more efficient
   - Don't migrate just because the feature works

### Feature Request to Watch

**Issue #3146**: Configure additional directories via settings files

**Proposed feature:**
```json
{
  "directories": [
    {
      "path": "lib/auth",
      "readClaudeMd": true
    },
    {
      "path": "lib/db",
      "readClaudeMd": true
    }
  ]
}
```

**If implemented:**
- Would allow explicit control over subdirectory loading
- Could enable selective subdirectory CLAUDE.md inclusion
- Re-evaluate Skills vs subdirectory approach

---

## Part 16: Quick Reference Card

### Path Import Syntax

```markdown
# In any CLAUDE.md file:

@README.md                               # Project root file
@docs/architecture.md                    # Project subdirectory
@.plan/initial-design/CORE-PRINCIPLES.md # Nested project path
@~/.claude/personal-prefs.md             # User home directory
```

### Subdirectory Bug Status

```
✅ Works: Parent directory CLAUDE.md files (recursive up)
❌ Broken: Subdirectory CLAUDE.md files (doesn't load)
✅ Workaround: Use Skills instead of subdirectory CLAUDE.md
📅 TODO: Check issues #2571, #3529, #4275 quarterly
```

### Decision Tree: Where to Put Instructions?

```
Is it core to ALL work?
├─ YES → CLAUDE.md (root)
└─ NO ──┐
        │
        Is it domain-specific (auth, db, API)?
        ├─ YES → .claude/skills/[domain]/SKILL.md
        └─ NO ──┐
                │
                Is it a complex multi-step workflow?
                ├─ YES → .claude/agents/[workflow].agent
                └─ NO ──┐
                        │
                        Is it learned from experience?
                        ├─ YES → .ai-knowledge/patterns.json
                        └─ NO ──┐
                                │
                                Is it existing documentation?
                                ├─ YES → Keep as-is, @import in CLAUDE.md
                                └─ NO → Maybe you don't need it?
```

### Token Budget Quick Math

```
CLAUDE.md (root):           ~500 tokens (always loaded)
Each @import:               +full file size (always loaded)
Each skill (inactive):      ~40 tokens (just metadata)
Each skill (active):        ~1,500 tokens (when loaded)
Each agent:                 Fresh context (isolated)

Example session:
- Root CLAUDE.md:           500 tokens
- 10 skills (inactive):     400 tokens
- 2 skills loaded:          3,000 tokens
- Total:                    3,900 tokens

vs Monolithic:
- Single CLAUDE.md:         5,000 tokens (all upfront)

Savings: 22% even with 2 skills loaded
```

---

**Document Version**: 1.2
**Last Updated**: 2025-11-03
**Added in v1.1**: Path clarifications, workarounds, bug tracking, revisit schedule
**Added in v1.2**: Main branch reorganization section, .cursorrules vs CLAUDE.md clarification, official vs custom skills comparison, experiments/ folder reference, updated file paths
**Next Review**: After Phase 1 validation experiments OR quarterly bug check
**Status**: Ready for implementation and validation
