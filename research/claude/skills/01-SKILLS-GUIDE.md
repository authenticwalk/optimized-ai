# Claude Code Skills - Complete Guide

**Status**: ✅ Validated by official documentation and community practice

---

## Table of Contents
1. [What Are Skills?](#what-are-skills)
2. [How Skills Work](#how-skills-work)
3. [File Structure](#file-structure)
4. [SKILL.md Format](#skillmd-format)
5. [Skill Discovery & Activation](#skill-discovery--activation)
6. [Best Practices](#best-practices)
7. [Advanced Features](#advanced-features)
8. [Official Examples](#official-examples)
9. [Community Patterns](#community-patterns)

---

## What Are Skills?

### Definition
**Skills are modular, discoverable packages that extend Claude's capabilities through organized folders containing instructions, scripts, and resources.**

### Key Characteristics
- ✅ **Model-invoked**: Claude autonomously decides when to use them (NOT user-invoked like slash commands)
- ✅ **On-demand loading**: Only 30-50 tokens consumed until needed
- ✅ **Context-specific**: Loaded based on task requirements
- ✅ **Composable**: Multiple skills can work together automatically
- ✅ **Isolated**: Each skill is self-contained

### Skills vs Slash Commands

| Feature | Skills | Slash Commands |
|---------|--------|----------------|
| Invocation | Model decides | User explicitly calls |
| Token cost | 30-50 until needed | Always loaded |
| Activation | Automatic based on description | Manual `/command` |
| Use case | Contextual expertise | Specific workflows |
| Composability | Multiple can activate | One at a time |

---

## How Skills Work

### Lifecycle

```
┌─────────────────┐
│  User Request   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ Claude reads all skill  │
│ descriptions (30-50     │
│ tokens each)            │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Determines relevance    │
│ based on description    │
│ and current context     │
└────────┬────────────────┘
         │
    ┌────┴────┐
    │ Match?  │
    └────┬────┘
         │
    Yes  │  No
    ▼    │    ▼
┌─────────────┐  ┌──────────────┐
│ Load full   │  │ Skip skill   │
│ SKILL.md    │  │              │
│ content     │  │              │
└─────┬───────┘  └──────────────┘
      │
      ▼
┌─────────────────┐
│ Execute with    │
│ skill guidance  │
└─────────────────┘
```

### Token Efficiency Example

```
Without skills (monolithic):
- All instructions loaded upfront: 2000+ lines
- Total tokens: 50,000+
- Context pollution: High

With skills (on-demand):
- Core: 250 lines
- Skill descriptions: 10 × 50 tokens = 500 tokens
- Relevant skill loaded: 100 lines
- Total tokens: ~15,000
- Token savings: 70%
```

---

## File Structure

### Storage Locations

Skills are discovered from three sources (in priority order):

1. **Project Skills** (highest priority)
   ```
   .claude/skills/skill-name/
   ├── SKILL.md          # Required
   ├── reference.md      # Optional
   ├── scripts/          # Optional
   │   └── helper.py
   └── templates/        # Optional
       └── template.txt
   ```

2. **Personal Skills**
   ```
   ~/.claude/skills/skill-name/
   ├── SKILL.md
   └── ...
   ```

3. **Plugin Skills**
   ```
   [plugin-directory]/skills/skill-name/
   ├── SKILL.md
   └── ...
   ```

### Directory Structure Example

```
my-optimized-skill/
├── SKILL.md                    # Main skill definition (required)
├── REFERENCE.md                # Detailed documentation (optional)
├── patterns/
│   ├── success-patterns.md     # Known good approaches
│   └── anti-patterns.md        # What to avoid
├── scripts/
│   ├── validate.py             # Helper scripts
│   └── generate.sh
├── templates/
│   ├── component.template      # Code templates
│   └── test.template
└── examples/
    ├── example1.md             # Example usage
    └── example2.md
```

---

## SKILL.md Format

### Basic Template

```markdown
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
---

# Your Skill Name

## Purpose
[Why this skill exists]

## When to Use
[Specific triggers and contexts]

## Instructions
[Step-by-step guidance for Claude]

## Examples
[Concrete examples]

## Common Pitfalls
[What to avoid]
```

### YAML Frontmatter Requirements

#### Required Fields

**name**
- Lowercase letters, numbers, and hyphens only
- Maximum 64 characters
- Example: `firebase-auth-setup`

**description**
- Maximum 1024 characters (but keep it focused)
- **Critical for discovery**: Must include both WHAT and WHEN
- This is what Claude reads to determine relevance
- Example: ❌ "Helps with Firebase" (too vague)
- Example: ✅ "Set up Firebase Authentication with email/password and social providers. Use when implementing user login, registration, or auth state management in Firebase projects."

#### Optional Fields

**allowed-tools**
- Comma-separated list of tools
- Restricts what Claude can do with this skill
- Useful for read-only or security-sensitive skills
- Example: `Read, Grep, Glob` (read-only access)

**version** (community convention)
- Track skill iterations
- Example: `1.2.0`

**dependencies** (community convention)
- Required software or packages
- Example: `python>=3.8, pandas>=1.5.0`

### Complete Example

```markdown
---
name: firebase-realtime-security
description: Design and implement Firebase Realtime Database security rules with comprehensive validation. Use when setting up Firebase database permissions, user data protection, or when you need to validate data structure and access patterns.
allowed-tools: Read, Write, Edit, Grep, Bash
version: 2.1.0
dependencies: firebase-tools>=12.0.0
---

# Firebase Realtime Database Security Rules

## Purpose
Create secure, validated Firebase Realtime Database rules that prevent unauthorized access while enabling legitimate operations.

## When to Use
- Setting up new Firebase Realtime Database
- Implementing user-specific data access
- Validating data structure and types
- Debugging permission denied errors
- Migrating from open rules to secure rules

## Core Principles
1. **Deny by default**: Start with no access, grant explicitly
2. **Validate all writes**: Check data structure and types
3. **Cascade read permissions**: Parent read enables child reads
4. **Use auth variables**: Leverage `auth.uid` for user-specific rules

## Instructions

### Step 1: Analyze Data Structure
```javascript
// Identify data paths and ownership patterns
{
  "users": {
    "$uid": {
      "profile": { /* user owns this */ },
      "settings": { /* user owns this */ }
    }
  },
  "posts": {
    "$postId": {
      "authorId": "...",
      "content": "..."
    }
  }
}
```

### Step 2: Define Rules Pattern
```json
{
  "rules": {
    "users": {
      "$uid": {
        ".read": "$uid === auth.uid",
        ".write": "$uid === auth.uid",
        "profile": {
          "name": {
            ".validate": "newData.isString() && newData.val().length > 0"
          }
        }
      }
    }
  }
}
```

### Step 3: Implement Validation
- Check data types with `.isString()`, `.isNumber()`, `.isBoolean()`
- Validate lengths and ranges
- Ensure required fields exist
- Validate relationships (foreign keys)

### Step 4: Test Rules
```bash
# Use Firebase emulator for testing
firebase emulators:start

# Test with Firebase rules simulator
# or use unit tests with @firebase/rules-unit-testing
```

## Common Patterns

### User-Owned Data
```json
".read": "auth.uid === $uid",
".write": "auth.uid === $uid"
```

### Public Read, Owner Write
```json
".read": true,
".write": "auth.uid === data.child('authorId').val()"
```

### Required Fields
```json
".validate": "newData.hasChildren(['name', 'email', 'createdAt'])"
```

## Common Pitfalls
1. ❌ Cascading write permissions (don't use)
2. ❌ Not validating data types
3. ❌ Overly permissive rules for testing that stay in production
4. ❌ Not considering cascade read implications

## Validation Checklist
- [ ] No rules default to `true` without auth check
- [ ] All write operations validate data structure
- [ ] User-specific data checks `auth.uid`
- [ ] Tested with emulator or rules simulator
- [ ] Documented any admin-level access patterns

## Related Skills
- `firebase-auth-setup`: For authentication configuration
- `firebase-functions-security`: For additional server-side validation

## Version History
- v2.1.0 (2025-10-15): Added cascade read warnings
- v2.0.0 (2025-09-01): Restructured for clarity
- v1.0.0 (2025-06-01): Initial version
```

---

## Skill Discovery & Activation

### How Claude Discovers Skills

1. **At conversation start**: Claude reads descriptions of all available skills (30-50 tokens each)
2. **During conversation**: Continuously evaluates relevance based on:
   - User's request keywords
   - Current task context
   - Skill descriptions

### Writing Effective Descriptions

#### The Formula: WHAT + WHEN + KEYWORDS

**Bad Description**
```yaml
description: Helps with documents
```
❌ Too vague, no triggers, no context

**Good Description**
```yaml
description: Extract text and tables from PDFs, fill forms, merge documents. Use when working with PDF files or document extraction.
```
✅ Clear capabilities, specific use case, keywords (PDF, forms, extract)

**Great Description**
```yaml
description: Set up Firebase Authentication with email/password, Google, and phone auth. Implements auth state management, protected routes, and user session handling. Use when adding login functionality, user registration, social authentication, or when you see 'Firebase Auth' in requirements. Keywords: authentication, login, signup, Firebase, auth providers.
```
✅ Specific capabilities, clear triggers, explicit keywords, when statement

### Activation Triggers

#### Automatic Activation
Include phrases in description to encourage proactive use:
- "Use PROACTIVELY when..."
- "MUST BE USED for..."
- "Automatically apply when..."

#### Explicit Activation
Users or main Claude can explicitly request:
- "Use the firebase-auth skill to implement login"
- "Apply the testing-patterns skill here"

---

## Best Practices

### 1. Single Responsibility
✅ **Do**: One focused capability per skill
```
✅ firebase-auth-setup
✅ firebase-firestore-queries
✅ firebase-security-rules
```

❌ **Don't**: Combine multiple concerns
```
❌ firebase-everything
❌ backend-helper
```

### 2. Description Best Practices

**Template**
```
[What it does]. [When to use it]. [Keywords for discovery].
```

**Examples**
```yaml
# Good
description: Create Jest unit tests with >90% coverage, including edge cases and mocks. Use when writing tests for TypeScript/JavaScript code. Keywords: testing, jest, unit test, coverage, TDD.

# Better
description: Design and implement Supabase Row Level Security (RLS) policies with role-based access control. Validates policy logic and tests with different user roles. Use when securing Supabase tables, implementing permissions, or debugging RLS policy errors. Keywords: Supabase, RLS, security, permissions, policies, auth.
```

### 3. Progressive Disclosure

**SKILL.md** (always loaded when skill activates)
- Quick reference
- Common patterns
- Essential instructions
- 200-400 lines max

**reference.md** or **advanced.md** (loaded only if needed)
- Detailed explanations
- Edge cases
- Complex scenarios
- Can be much longer

**Example reference in SKILL.md**
```markdown
## Advanced Usage
For complex scenarios like multi-tenant RLS or recursive queries,
see [ADVANCED.md](ADVANCED.md).
```

### 4. Tool Restrictions

**Read-only skill** (safe, no approval needed)
```yaml
allowed-tools: Read, Grep, Glob
```

**Implementation skill** (needs write access)
```yaml
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
```

**Full access** (default, all tools)
```yaml
# Omit allowed-tools field
```

### 5. Include Examples

Always include:
- ✅ Code examples with comments
- ✅ Before/after comparisons
- ✅ Common patterns
- ✅ Anti-patterns (what NOT to do)
- ✅ Validation checklists

### 6. Version Your Skills

```markdown
## Version History
- v2.1.0 (2025-11-01): Added support for OAuth providers
- v2.0.0 (2025-10-01): Breaking: Changed config format
- v1.2.0 (2025-09-15): Added phone auth support
- v1.1.0 (2025-09-01): Improved error handling
- v1.0.0 (2025-08-01): Initial release
```

### 7. Test Your Skills

**Before deployment**
- ✅ Test with various prompts
- ✅ Verify skill activates correctly
- ✅ Check that examples work
- ✅ Validate tool restrictions

**After deployment**
- ✅ Monitor activation rate
- ✅ Gather user feedback
- ✅ Refine description if needed
- ✅ Update examples based on usage

---

## Advanced Features

### 1. Skill Composition

**Claude can use multiple skills together automatically**

Example scenario: "Add authenticated Firebase query"
```
Skills loaded:
- firebase-auth-setup (for auth context)
- firebase-firestore-queries (for querying)
- firebase-security-rules (for validation)
```

**Design for composition**
```markdown
## Related Skills
This skill works well with:
- `firebase-auth-setup`: For authentication setup
- `testing-patterns`: For testing the queries
- `error-handling`: For graceful error management
```

### 2. Scripts and Helpers

**Include executable scripts**
```
skill-name/
├── SKILL.md
└── scripts/
    ├── validate-config.py
    └── generate-template.sh
```

**Reference in SKILL.md**
```markdown
## Validation
Run the validation script to check your configuration:
```bash
python scripts/validate-config.py firebase.json
```
```

### 3. Templates and Patterns

**Include reusable templates**
```
skill-name/
├── SKILL.md
└── templates/
    ├── component.template.tsx
    ├── test.template.ts
    └── config.template.json
```

**Usage in skill**
```markdown
## Quick Start
Use this template as a starting point:
[see templates/component.template.tsx](templates/component.template.tsx)
```

### 4. Skill Chaining

**Explicitly document dependencies**
```markdown
## Prerequisites
Before using this skill:
1. Run `firebase-init` skill to initialize project
2. Configure authentication with `firebase-auth-setup`
3. Then use this skill for database setup
```

---

## Official Examples

### From anthropics/skills Repository

**algorithmic-art**
- Purpose: Generate p5.js art with seeded randomness
- Size: ~300 lines
- Features: Particle systems, color palettes, canvas management

**mcp-builder**
- Purpose: Guide for creating MCP servers
- Size: ~500 lines
- Features: TypeScript templates, testing patterns, deployment

**webapp-testing**
- Purpose: Playwright-based web app testing
- Size: ~400 lines
- Features: Test patterns, selectors, assertions

**skill-creator** (meta-skill)
- Purpose: Interactive skill creation tool
- Size: ~350 lines
- Features: Guided prompts, validation, templates

### Document Skills (Production-Grade)

**docx**
- Create and edit Word documents
- Preserve formatting
- Table and image support
- ~600 lines with examples

**pdf**
- PDF manipulation and form filling
- Text extraction
- Merge/split operations
- ~550 lines

**xlsx**
- Excel spreadsheet creation
- Formula support
- Formatting and charts
- ~650 lines

---

## Community Patterns

### From obra/superpowers

**Mandatory Workflows Pattern**
```markdown
---
name: test-driven-development
description: MANDATORY for all feature implementation. Enforces TDD cycle: Red → Green → Refactor. Use PROACTIVELY before writing any new feature code.
---

# Test-Driven Development

## When to Use
- ✅ REQUIRED for all new features
- ✅ REQUIRED for bug fixes
- ✅ Use before writing implementation

## Process
1. Write failing test first (Red)
2. Write minimum code to pass (Green)
3. Refactor for quality
4. Repeat
```

**Debugging Skill Pattern**
```markdown
---
name: systematic-debugging
description: Systematic root cause analysis for errors and unexpected behavior. Use PROACTIVELY when encountering test failures, runtime errors, or bugs. Prevents guess-and-check debugging.
---

# Systematic Debugging

## Activation
- Automatically use when: test fails, error occurs, unexpected behavior
- DO NOT guess at solutions
- DO follow systematic process

## Method
1. Reproduce reliably
2. Isolate the failure
3. Identify root cause
4. Validate fix
5. Add regression test
```

### From jayampathiw/claude-code-toolkit

**Progressive Disclosure Pattern**
```
skill-name/
├── SKILL.md (400 lines - quick reference)
├── detailed-guides/
│   ├── advanced-patterns.md
│   ├── troubleshooting.md
│   └── best-practices.md
└── examples/
    ├── basic-example/
    └── advanced-example/
```

**Validation Checklist Pattern**
```markdown
## Implementation Checklist
Before marking complete:
- [ ] All required fields present
- [ ] Types validated
- [ ] Edge cases handled
- [ ] Tests passing (>90% coverage)
- [ ] Error handling implemented
- [ ] Documentation updated
```

---

## Key Takeaways

### For Optimized AI Project

1. **Start with 3-5 focused skills**
   - `firebase-auth`
   - `supabase-rls`
   - `testing-patterns`
   - `code-review`
   - `error-handling`

2. **Follow official format exactly**
   - YAML frontmatter with name + description
   - Description = WHAT + WHEN + KEYWORDS
   - Progressive disclosure
   - Include examples

3. **Measure token savings**
   - Baseline: Monolithic approach
   - Treatment: On-demand skills
   - Validate 60%+ token reduction

4. **Use "PROACTIVELY" for mandatory skills**
   - Testing skill: "Use PROACTIVELY before implementation"
   - Security skill: "MUST BE USED for auth setup"

5. **Version and iterate**
   - Start simple
   - Gather usage data
   - Refine descriptions
   - Add patterns based on success

---

## Resources

### Official
- [Claude Code Skills Docs](https://docs.claude.com/en/docs/claude-code/skills)
- [Anthropic Skills Repository](https://github.com/anthropics/skills)

### Community
- [Awesome Claude Skills](https://github.com/travisvn/awesome-claude-skills)
- [Superpowers Core Library](https://github.com/obra/superpowers)
- [Claude Code Toolkit](https://github.com/jayampathiw/claude-code-toolkit)

---

**Next**: Read [02-SUBAGENTS-ARCHITECTURE.md](02-SUBAGENTS-ARCHITECTURE.md) for subagent patterns
