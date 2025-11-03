# Agent Skills Architecture

**Last Updated**: November 3, 2025

---

## Table of Contents

1. [What Are Agent Skills?](#what-are-agent-skills)
2. [Progressive Disclosure Architecture](#progressive-disclosure-architecture)
3. [Skills vs Subagents](#skills-vs-subagents)
4. [Skill Structure](#skill-structure)
5. [On-Demand Loading Mechanism](#on-demand-loading-mechanism)
6. [Creating Skills](#creating-skills)
7. [Best Practices](#best-practices)
8. [Token Efficiency](#token-efficiency)
9. [Real-World Examples](#real-world-examples)

---

## What Are Agent Skills?

### Official Definition

According to Anthropic:

> **"Skills are folders that include instructions, scripts, and resources that Claude can load when needed."**

More technically:

> **"A skill is a directory containing a SKILL.md file that contains organized folders of instructions, scripts, and resources that give agents additional capabilities."**

### Core Characteristics

Skills are:
1. **Composable** - Multiple skills coordinate automatically when needed
2. **Portable** - Consistent format across all Claude platforms
3. **Efficient** - Minimal loading of context and resources (30-50 tokens until activated)
4. **Powerful** - Can include executable code for reliable task execution

### Key Insight

**Skills enable massive context savings through progressive disclosure:**

```
Without Skills (Monolithic):
.cursorrules: 500 lines (always loaded)
All instructions for all scenarios
= 500 lines × ~4 tokens = ~2,000 tokens ALWAYS in context

With Skills:
Core .cursorrules: 50 lines = ~200 tokens
Skill metadata: 10 skills × 5 tokens = 50 tokens
Total baseline: 250 tokens

When Firebase task triggered:
+ firebase-auth.skill: 100 lines = ~400 tokens
Total during task: 650 tokens

Savings: 2,000 → 650 tokens = 67.5% reduction!
```

---

## Progressive Disclosure Architecture

### Three-Tier Loading Strategy

Anthropic's Skills implement a sophisticated loading mechanism:

#### **Tier 1: Discovery (Startup)**

```
Loaded: Skill metadata only
Cost: 30-50 tokens total for all skills
Content: Name + Description

Example metadata:
- name: firebase-auth
- description: Firebase authentication patterns and best practices
- Cost: ~5 tokens
```

**Benefit**: Can have 10-20 skills installed with minimal context penalty

#### **Tier 2: Invocation (Task Match)**

```
Trigger: Claude detects task matches skill description
Loaded: SKILL.md full content
Cost: ~400 tokens (100 lines)

Example:
User: "Implement Firebase login"
Claude: Matches "firebase-auth" description
Action: Loads SKILL.md file
Context: Now includes Firebase-specific instructions
```

**Benefit**: Only relevant instructions loaded for current task

#### **Tier 3: Progressive Resources (As Needed)**

```
Trigger: Specific scenario within skill
Loaded: Referenced resource files
Cost: Variable, only specific files needed

Example:
SKILL.md references:
- forms.md (form handling patterns)
- validation.md (input validation rules)
- security.md (security best practices)

Claude only loads security.md if working on security scenario
```

**Benefit**: Even within a skill, only load what's immediately relevant

### Analogy from Anthropic

> **"Like a well-organized manual that starts with a table of contents, then specific chapters, and finally a detailed appendix."**

You don't read the entire manual upfront—you navigate to relevant sections as needed.

---

## Skills vs Subagents

### Comparison Matrix

| Aspect | Skills | Subagents |
|--------|--------|-----------|
| **Purpose** | Add instructions/knowledge | Execute specialized tasks |
| **Context** | Loaded into main agent | Separate isolated context |
| **Activation** | Automatic when relevant | Delegated by orchestrator |
| **Cost** | 30-50 tokens until used | 15× token overhead |
| **Parallelization** | No (instructions) | Yes (execution) |
| **Tool Access** | Uses main agent's tools | Restricted tool set |
| **Best For** | Domain knowledge | Task execution |

### When to Use Each

#### Use **Skills** for:

- ✅ Domain-specific instructions (Firebase, Supabase patterns)
- ✅ Project conventions (coding style, architecture)
- ✅ Best practices (testing, security)
- ✅ Reference information (API docs, schemas)
- ✅ Code templates and examples

#### Use **Subagents** for:

- ✅ Complex task execution (implementation, review)
- ✅ Parallel processing (multiple files)
- ✅ Context isolation (research, exploration)
- ✅ Specialized tool access (read-only reviewer)

### Combined Usage

**Skills and Subagents work together**:

```typescript
// Subagent definition references skills
agents: {
  'firebase-implementer': {
    description: 'Implements Firebase features',
    prompt: `You implement Firebase features.

When working on authentication, load the 'firebase-auth' skill.
When working on Firestore, load the 'firebase-firestore' skill.

Follow the patterns and best practices from loaded skills.`,
    tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob'],
    model: 'sonnet'
  }
}
```

**Result**: Subagent gets specialized instructions via skills as it works

---

## Skill Structure

### Directory Layout

```
skills/
├── firebase-auth/
│   ├── SKILL.md              # Core file (required)
│   ├── patterns/
│   │   ├── login.md          # Login patterns
│   │   ├── signup.md         # Signup patterns
│   │   └── session.md        # Session management
│   ├── examples/
│   │   ├── basic-auth.ts     # Code example
│   │   └── protected-route.ts
│   ├── scripts/
│   │   ├── init-firebase.py  # Executable script
│   │   └── test-auth.sh
│   └── reference/
│       └── firebase-api.md   # API documentation
│
├── testing/
│   ├── SKILL.md
│   ├── patterns/
│   │   ├── unit-tests.md
│   │   ├── integration-tests.md
│   │   └── edge-cases.md
│   └── examples/
│       └── test-templates.ts
│
└── refactoring/
    ├── SKILL.md
    └── patterns/
        ├── extraction.md
        ├── simplification.md
        └── duplication.md
```

### SKILL.md Format

```markdown
---
name: firebase-auth
description: Firebase authentication patterns and best practices for signup, login, and session management
version: "1.0.0"
author: Your Team
tags: [firebase, authentication, security]
---

# Firebase Authentication Skill

This skill provides patterns and best practices for implementing Firebase Authentication.

## When to Use This Skill

Load this skill when:
- Implementing user signup/login
- Managing authentication state
- Working with Firebase Auth API
- Handling auth errors and edge cases

## Core Patterns

### Basic Authentication Flow

1. Initialize Firebase (see [scripts/init-firebase.py](scripts/init-firebase.py))
2. Implement signup (see [patterns/signup.md](patterns/signup.md))
3. Implement login (see [patterns/login.md](patterns/login.md))
4. Manage sessions (see [patterns/session.md](patterns/session.md))

### Quick Reference

Common operations:
- `createUserWithEmailAndPassword()` - Signup
- `signInWithEmailAndPassword()` - Login
- `signOut()` - Logout
- `onAuthStateChanged()` - Monitor auth state

## Security Considerations

⚠️ Always:
- Validate email format
- Enforce strong passwords
- Handle auth errors gracefully
- Use HTTPS only
- Secure Firebase configuration

For detailed security patterns, see [reference/security.md](reference/security.md)

## Code Examples

Basic signup implementation:
```typescript
// See examples/basic-auth.ts for complete example
import { auth } from './firebase-config';
import { createUserWithEmailAndPassword } from 'firebase/auth';

async function signup(email: string, password: string) {
  try {
    const userCredential = await createUserWithEmailAndPassword(
      auth,
      email,
      password
    );
    return userCredential.user;
  } catch (error) {
    // Handle errors (see patterns/error-handling.md)
    throw error;
  }
}
```

## Progressive Loading

- **Tier 1**: This SKILL.md file (loaded when skill invoked)
- **Tier 2**: Pattern files (loaded as specific scenarios arise)
- **Tier 3**: Examples and scripts (executed but not fully loaded into context)

## Common Issues

### Issue: "Firebase not initialized"
**Solution**: Ensure `initializeApp()` called before any auth operations
**Reference**: [scripts/init-firebase.py](scripts/init-firebase.py)

### Issue: "Invalid email format"
**Solution**: Add email validation before `createUserWithEmailAndPassword()`
**Reference**: [patterns/validation.md](patterns/validation.md)

---

For advanced patterns and edge cases, see the [patterns/](patterns/) directory.
```

### Resource File Example

`patterns/login.md`:

```markdown
# Login Patterns

## Standard Email/Password Login

```typescript
import { signInWithEmailAndPassword } from 'firebase/auth';

export async function login(email: string, password: string) {
  // 1. Validate inputs
  if (!email || !password) {
    throw new Error('Email and password required');
  }

  // 2. Attempt login
  try {
    const credential = await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

    // 3. Update local state
    localStorage.setItem('user', JSON.stringify(credential.user));

    return credential.user;
  } catch (error) {
    // 4. Handle specific error codes
    if (error.code === 'auth/user-not-found') {
      throw new Error('No account found with this email');
    }
    if (error.code === 'auth/wrong-password') {
      throw new Error('Incorrect password');
    }
    throw error;
  }
}
```

## Error Handling

Common Firebase Auth error codes:
- `auth/invalid-email` - Malformed email
- `auth/user-disabled` - Account disabled
- `auth/user-not-found` - No account exists
- `auth/wrong-password` - Incorrect password

## Best Practices

✅ **Do:**
- Validate inputs before API call
- Handle all error codes explicitly
- Update UI state on success/failure
- Clear sensitive data on logout

❌ **Don't:**
- Store passwords in local storage
- Ignore error handling
- Make assumptions about user state
```

---

## On-Demand Loading Mechanism

### How Claude Loads Skills

#### Step 1: Skill Discovery (Startup)

```
Claude starts session:

1. Scans skill directories:
   - Project: .claude/skills/
   - User: ~/.claude/skills/

2. Loads metadata only:
   - name: "firebase-auth"
   - description: "Firebase authentication patterns..."
   - (Does NOT load SKILL.md content yet)

3. Adds to system prompt:
   "Available skills:
   - firebase-auth: Firebase authentication patterns and best practices
   - supabase-rls: Supabase RLS policy patterns
   - testing: Testing patterns and edge cases
   ..."

Token cost at startup: ~50 tokens for 10 skills
```

#### Step 2: Task Analysis (User Request)

```
User: "Implement Firebase email login"

Claude analyzes:
- Keywords: "Firebase", "email login"
- Intent: Authentication implementation
- Match: "firebase-auth" skill description matches!

Decision: Load firebase-auth skill
```

#### Step 3: Skill Loading (Invocation)

```
Claude invokes Bash tool:
$ cat .claude/skills/firebase-auth/SKILL.md

Result: SKILL.md content loaded into context

Token cost: ~400 tokens (100 lines)
```

#### Step 4: Progressive Resource Loading (As Needed)

```
Claude reads SKILL.md, sees:
"For login patterns, see patterns/login.md"

Claude's internal reasoning:
"User wants login implementation. Need login patterns."

Claude invokes Bash tool:
$ cat .claude/skills/firebase-auth/patterns/login.md

Token cost: ~200 tokens (50 lines)

Total skill cost: 400 + 200 = 600 tokens
Still way less than 2,000 token monolithic approach!
```

### Context Window Management

```
Initial Context (before skills):
- System prompt: 1,000 tokens
- Core .cursorrules: 200 tokens
- User message: 100 tokens
Total: 1,300 tokens

After skill loading (when needed):
- System prompt: 1,000 tokens
- Core .cursorrules: 200 tokens
- User message: 100 tokens
- Skill metadata (all skills): 50 tokens
- firebase-auth SKILL.md: 400 tokens
- patterns/login.md: 200 tokens
Total: 1,950 tokens

Still under 2,000 tokens!
Compare to monolithic: 3,000+ tokens always loaded
```

---

## Creating Skills

### Method 1: Interactive (Recommended for Beginners)

Use Claude's built-in skill creator:

```
User: "Create a skill for Firebase authentication"

Claude (with skill-creator skill):
1. Asks about workflow and requirements
2. Generates folder structure
3. Formats SKILL.md file
4. Bundles resources you provide
5. Saves to .claude/skills/firebase-auth/
```

### Method 2: Manual (Recommended for Experienced)

1. **Create directory structure**:

```bash
mkdir -p .claude/skills/firebase-auth/{patterns,examples,scripts,reference}
```

2. **Write SKILL.md**:

```bash
cat > .claude/skills/firebase-auth/SKILL.md << 'EOF'
---
name: firebase-auth
description: Firebase authentication patterns and best practices
---

# Firebase Authentication Skill

[Content as shown in structure section]
EOF
```

3. **Add resource files**:

```bash
cat > .claude/skills/firebase-auth/patterns/login.md << 'EOF'
# Login Patterns
[Content as shown earlier]
EOF
```

4. **Test the skill**:

```
User: "Implement Firebase login using email and password"
Claude: [Should automatically load firebase-auth skill]
```

### Method 3: Programmatic (Advanced)

For dynamically generated skills:

```typescript
import { createSkill } from '@claude-agent-sdk/skills';

await createSkill({
  name: 'firebase-auth',
  description: 'Firebase authentication patterns and best practices',
  content: `
# Firebase Authentication Skill
[SKILL.md content]
  `,
  resources: {
    'patterns/login.md': loginPatternsContent,
    'examples/basic-auth.ts': basicAuthExample,
  },
  location: 'project' // or 'user'
});
```

---

## Best Practices

### 1. Keep SKILL.md Lean

**Problem**: Large SKILL.md defeats progressive disclosure

```markdown
❌ BAD: Everything in SKILL.md (1,000 lines)
---
name: firebase
description: All Firebase patterns
---
# Firebase Skill
[500 lines of authentication patterns]
[300 lines of Firestore patterns]
[200 lines of Cloud Functions patterns]

Token cost when loaded: 4,000 tokens!
```

```markdown
✅ GOOD: Split into resources (100 lines)
---
name: firebase
description: All Firebase patterns
---
# Firebase Skill

## Authentication
See [patterns/auth.md](patterns/auth.md)

## Firestore
See [patterns/firestore.md](patterns/firestore.md)

## Cloud Functions
See [patterns/functions.md](patterns/functions.md)

Token cost when loaded: 400 tokens
+ Load specific pattern file as needed: 200 tokens
Total: 600 tokens (85% savings!)
```

### 2. Split Mutually Exclusive Contexts

**If contexts are rarely used together, keep them separate**:

```
❌ BAD: Single monolithic skill
skills/
└── backend/
    ├── SKILL.md (contains auth + database + API + deployment)
    └── [rarely need all at once]

✅ GOOD: Separate focused skills
skills/
├── auth/
│   └── SKILL.md (only authentication)
├── database/
│   └── SKILL.md (only database)
├── api/
│   └── SKILL.md (only API patterns)
└── deployment/
    └── SKILL.md (only deployment)
```

**Benefit**: Load only relevant skill for current task

### 3. Use Code as Documentation AND Execution

**Scripts serve dual purpose**:

```python
# scripts/init-firebase.py
"""
Firebase Initialization Script

This script initializes Firebase with proper configuration.
Can be executed directly or referenced as documentation.

Usage:
    python scripts/init-firebase.py

Configuration:
    Set FIREBASE_API_KEY, FIREBASE_PROJECT_ID, etc. in .env
"""

import os
from firebase_admin import initialize_app, credentials

def init_firebase():
    """Initialize Firebase with environment configuration."""
    cred = credentials.Certificate({
        "type": "service_account",
        "project_id": os.getenv("FIREBASE_PROJECT_ID"),
        "private_key": os.getenv("FIREBASE_PRIVATE_KEY"),
        # ...
    })

    initialize_app(cred)
    print("✅ Firebase initialized successfully")

if __name__ == "__main__":
    init_firebase()
```

**Benefits**:
- **Executable**: Claude can run it directly
- **Documentation**: Shows exact implementation
- **Context-efficient**: Claude doesn't need to load entire file into context to execute it

### 4. Descriptive Names and Descriptions

**Claude uses these to decide when to load the skill**:

```yaml
❌ BAD: Vague
name: auth
description: Authentication stuff

✅ GOOD: Specific and keyword-rich
name: firebase-auth
description: Firebase authentication patterns for email/password signup, login, session management, and error handling with security best practices
```

**Keywords matter**: Include terms users are likely to mention

### 5. Iterate Based on Real Usage

**Monitor and refine**:

```
After 10 tasks using firebase-auth skill:

Observations:
- patterns/login.md loaded in 9/10 tasks
- patterns/password-reset.md loaded in 1/10 tasks
- examples/oauth.ts never loaded

Actions:
1. Move login patterns into SKILL.md (always needed)
2. Keep password-reset separate (rarely needed)
3. Remove oauth examples (unused)

Result: More efficient loading pattern
```

### 6. Think from Claude's Perspective

**Ask: "What information does Claude actually need RIGHT NOW?"**

```
Task: "Implement basic Firebase login"

Claude needs:
✅ Login function signature
✅ Required imports
✅ Error handling patterns
✅ Basic validation

Claude doesn't need (yet):
❌ OAuth integration patterns
❌ Multi-factor authentication
❌ Advanced security configurations
❌ Custom claims

Only include what's immediately relevant in SKILL.md.
Put advanced topics in separate resource files.
```

---

## Token Efficiency

### Measurement

**Comparing approaches**:

```
Scenario: Implement Firebase auth + Firestore queries + Cloud Functions

Approach 1: Monolithic .cursorrules
- All patterns always loaded
- Token cost: 3,000 tokens (constant)
- 3 tasks × 3,000 = 9,000 tokens total

Approach 2: Skills (progressive disclosure)
- Core: 200 tokens (constant)
- Metadata: 50 tokens (constant)
- Task 1 (auth): +400 tokens = 650 tokens
- Task 2 (firestore): +400 tokens = 650 tokens
- Task 3 (functions): +400 tokens = 650 tokens
- Total: 1,950 tokens (60% savings!)

Approach 3: Ultra-granular skills
- Core: 200 tokens
- Metadata: 100 tokens (20 skills)
- Each task: +200 tokens (smaller skills)
- Total: 900 tokens (70% savings!)
```

### Real-World Metrics

From our research:
- **Skills use 30-50 tokens** until activated
- **Progressive disclosure saves 60-70%** vs monolithic
- **Token usage = 80% of performance variance**

**Conclusion**: Skills are essential for token efficiency

---

## Real-World Examples

### Example 1: Firebase Skill Suite

```
skills/
├── firebase-auth/
│   ├── SKILL.md (100 lines = 400 tokens)
│   ├── patterns/
│   │   ├── signup.md
│   │   ├── login.md
│   │   ├── logout.md
│   │   └── session.md
│   └── examples/
│       └── complete-auth.ts
│
├── firebase-firestore/
│   ├── SKILL.md (100 lines = 400 tokens)
│   ├── patterns/
│   │   ├── queries.md
│   │   ├── updates.md
│   │   ├── transactions.md
│   │   └── security-rules.md
│   └── examples/
│       └── crud-operations.ts
│
└── firebase-functions/
    ├── SKILL.md (100 lines = 400 tokens)
    ├── patterns/
    │   ├── http-triggers.md
    │   ├── firestore-triggers.md
    │   └── scheduled-functions.md
    └── examples/
        └── sample-functions.ts
```

**Usage**:
- Auth task: Load firebase-auth only (650 tokens)
- Firestore task: Load firebase-firestore only (650 tokens)
- Combined task: Load both (1,050 tokens)
- Still way less than 3,000 token monolithic approach

### Example 2: Testing Skill

```
skills/testing/
├── SKILL.md
├── patterns/
│   ├── unit-tests.md
│   ├── integration-tests.md
│   ├── e2e-tests.md
│   └── edge-cases.md
├── examples/
│   ├── test-templates.ts
│   └── mock-data.ts
└── scripts/
    └── run-tests.sh
```

**SKILL.md**:

```markdown
---
name: testing
description: Testing patterns, edge case identification, and test templates for unit, integration, and e2e tests
---

# Testing Skill

## When to Use
- Writing new tests
- Identifying edge cases
- Reviewing test coverage
- Debugging test failures

## Quick Guide

### Test Types
- **Unit**: Test individual functions (see [patterns/unit-tests.md](patterns/unit-tests.md))
- **Integration**: Test component interactions (see [patterns/integration-tests.md](patterns/integration-tests.md))
- **E2E**: Test full user flows (see [patterns/e2e-tests.md](patterns/e2e-tests.md))

### Edge Cases to Consider
See [patterns/edge-cases.md](patterns/edge-cases.md) for comprehensive list:
- Empty inputs
- Null/undefined
- Boundary values
- Invalid types
- Network failures
- Concurrent operations

### Templates
Use [examples/test-templates.ts](examples/test-templates.ts) for starter code.

## Running Tests
Execute [scripts/run-tests.sh](scripts/run-tests.sh) to run full test suite.
```

**Token efficiency**:
- Metadata: 5 tokens
- When activated: 400 tokens (SKILL.md)
- Specific pattern: +200 tokens as needed
- Total: 605 tokens vs 1,500 if all patterns were in core

---

## Key Takeaways

1. **Skills enable 60-70% token savings** through progressive disclosure

2. **Three-tier loading**: Metadata → SKILL.md → Resource files

3. **Keep SKILL.md lean** (100 lines), split into resources

4. **Skills ≠ Subagents**: Skills add knowledge, subagents execute tasks

5. **Use code as both documentation and executable tools**

6. **Descriptive names and descriptions** help Claude decide when to load

7. **Iterate based on real usage patterns**

8. **Think from Claude's perspective**: What's needed RIGHT NOW?

9. **Skills are essential for our MINIMIZE principle**

10. **Proven by Anthropic**: Industry-standard approach**

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Subagents vs Skills
- [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md) - Token efficiency techniques
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive guidelines
- [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md) - Applying to Optimized AI
