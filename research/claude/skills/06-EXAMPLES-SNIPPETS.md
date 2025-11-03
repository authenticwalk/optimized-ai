# Practical Examples & Code Snippets

**Status**: 📦 Production-ready examples from community repos

---

## Table of Contents
1. [Complete Skill Examples](#complete-skill-examples)
2. [Subagent Configurations](#subagent-configurations)
3. [Hook Scripts](#hook-scripts)
4. [MCP Server Examples](#mcp-server-examples)
5. [Workflow Patterns](#workflow-patterns)

---

## Complete Skill Examples

### Example 1: Firebase Authentication Skill

```yaml
---
name: firebase-auth-setup
description: Complete Firebase Authentication setup with email/password, Google, and phone auth. Implements auth state management, protected routes, and session handling. Use when adding login functionality, user registration, social authentication, or Firebase Auth features.
version: 2.0.0
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
dependencies: firebase>=10.0.0
---

# Firebase Authentication Setup

## Quick Start

For new Firebase auth implementation:
1. Initialize Firebase app config
2. Set up auth providers
3. Implement auth state listener
4. Create login/signup UI
5. Add protected route guards

## Firebase Configuration

### Initialize Firebase

```typescript
// src/lib/firebase-config.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID
};

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

## Email/Password Authentication

### Sign Up

```typescript
// src/lib/auth/email-signup.ts
import { createUserWithEmailAndPassword, sendEmailVerification } from 'firebase/auth';
import { auth } from '../firebase-config';

export async function signUpWithEmail(email: string, password: string) {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email, password);

    // Send verification email
    await sendEmailVerification(userCredential.user);

    return {
      success: true,
      user: userCredential.user
    };
  } catch (error: any) {
    // Handle specific error codes
    switch (error.code) {
      case 'auth/email-already-in-use':
        throw new Error('Email already registered');
      case 'auth/invalid-email':
        throw new Error('Invalid email format');
      case 'auth/weak-password':
        throw new Error('Password should be at least 6 characters');
      default:
        throw new Error('Sign up failed: ' + error.message);
    }
  }
}
```

### Sign In

```typescript
// src/lib/auth/email-signin.ts
import { signInWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../firebase-config';

export async function signInWithEmail(email: string, password: string) {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);

    // Check if email is verified
    if (!userCredential.user.emailVerified) {
      throw new Error('Please verify your email before signing in');
    }

    return {
      success: true,
      user: userCredential.user
    };
  } catch (error: any) {
    switch (error.code) {
      case 'auth/invalid-credential':
        throw new Error('Invalid email or password');
      case 'auth/user-disabled':
        throw new Error('Account has been disabled');
      default:
        throw new Error('Sign in failed: ' + error.message);
    }
  }
}
```

## Google Authentication

```typescript
// src/lib/auth/google-signin.ts
import { signInWithPopup, GoogleAuthProvider } from 'firebase/auth';
import { auth } from '../firebase-config';

const googleProvider = new GoogleAuthProvider();

// Optional: Request specific scopes
googleProvider.addScope('profile');
googleProvider.addScope('email');

export async function signInWithGoogle() {
  try {
    const result = await signInWithPopup(auth, googleProvider);

    // Get Google Access Token
    const credential = GoogleAuthProvider.credentialFromResult(result);
    const accessToken = credential?.accessToken;

    return {
      success: true,
      user: result.user,
      accessToken
    };
  } catch (error: any) {
    switch (error.code) {
      case 'auth/popup-closed-by-user':
        throw new Error('Sign in cancelled');
      case 'auth/popup-blocked':
        throw new Error('Popup blocked. Please enable popups for this site');
      default:
        throw new Error('Google sign in failed: ' + error.message);
    }
  }
}
```

## Auth State Management

### Auth Store (Svelte)

```typescript
// src/lib/stores/auth-store.ts
import { writable, derived } from 'svelte/store';
import { onAuthStateChanged, type User } from 'firebase/auth';
import { auth } from '../firebase-config';

function createAuthStore() {
  const { subscribe, set } = writable<User | null | undefined>(undefined);

  // Initialize auth listener
  onAuthStateChanged(auth, (user) => {
    set(user);
  });

  return {
    subscribe,
    signOut: async () => {
      await auth.signOut();
      set(null);
    }
  };
}

export const authStore = createAuthStore();

// Derived stores
export const isAuthenticated = derived(
  authStore,
  ($authStore) => $authStore !== null && $authStore !== undefined
);

export const isLoading = derived(
  authStore,
  ($authStore) => $authStore === undefined
);
```

### Protected Route Guard

```typescript
// src/lib/guards/auth-guard.ts
import { get } from 'svelte/store';
import { authStore } from '../stores/auth-store';
import { goto } from '$app/navigation';

export function requireAuth() {
  const user = get(authStore);

  if (!user) {
    goto('/login');
    return false;
  }

  return true;
}

// Usage in +page.ts
export async function load() {
  if (!requireAuth()) {
    return { redirect: '/login' };
  }

  return {};
}
```

## Testing

```typescript
// tests/auth/email-signin.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { signInWithEmail } from '$lib/auth/email-signin';

// Mock Firebase
vi.mock('firebase/auth', () => ({
  signInWithEmailAndPassword: vi.fn(),
  getAuth: vi.fn()
}));

describe('signInWithEmail', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should sign in successfully with verified email', async () => {
    const mockUser = {
      uid: '123',
      email: 'test@example.com',
      emailVerified: true
    };

    vi.mocked(signInWithEmailAndPassword).mockResolvedValue({
      user: mockUser
    } as any);

    const result = await signInWithEmail('test@example.com', 'password123');

    expect(result.success).toBe(true);
    expect(result.user.email).toBe('test@example.com');
  });

  it('should throw error for unverified email', async () => {
    const mockUser = {
      uid: '123',
      email: 'test@example.com',
      emailVerified: false
    };

    vi.mocked(signInWithEmailAndPassword).mockResolvedValue({
      user: mockUser
    } as any);

    await expect(
      signInWithEmail('test@example.com', 'password123')
    ).rejects.toThrow('Please verify your email');
  });

  it('should handle invalid credentials', async () => {
    vi.mocked(signInWithEmailAndPassword).mockRejectedValue({
      code: 'auth/invalid-credential'
    });

    await expect(
      signInWithEmail('test@example.com', 'wrong-password')
    ).rejects.toThrow('Invalid email or password');
  });
});
```

## Common Patterns

### Password Reset

```typescript
export async function resetPassword(email: string) {
  await sendPasswordResetEmail(auth, email);
}
```

### Update Profile

```typescript
export async function updateUserProfile(displayName: string, photoURL?: string) {
  if (!auth.currentUser) throw new Error('Not authenticated');

  await updateProfile(auth.currentUser, {
    displayName,
    photoURL
  });
}
```

### Reauthentication (for sensitive operations)

```typescript
export async function reauthenticateUser(password: string) {
  if (!auth.currentUser || !auth.currentUser.email) {
    throw new Error('Not authenticated');
  }

  const credential = EmailAuthProvider.credential(
    auth.currentUser.email,
    password
  );

  await reauthenticateWithCredential(auth.currentUser, credential);
}
```

## Anti-Patterns

❌ **Don't**: Store sensitive data in localStorage
```typescript
// Bad
localStorage.setItem('user', JSON.stringify(user));
```

❌ **Don't**: Trust client-side auth checks alone
```typescript
// Bad - client can modify this
if (userRole === 'admin') { /* allow action */ }

// Good - always validate on server (Cloud Functions + Security Rules)
```

❌ **Don't**: Ignore email verification
```typescript
// Bad
const user = await signInWithEmailAndPassword(email, password);
// No verification check

// Good
if (!user.emailVerified) throw new Error('Verify email first');
```

## Checklist

Before marking auth implementation complete:
- [ ] Email/password signup implemented
- [ ] Email/password signin implemented
- [ ] Email verification required
- [ ] Password reset available
- [ ] Google OAuth working (if required)
- [ ] Auth state persists on refresh
- [ ] Protected routes redirect to login
- [ ] Sign out functionality works
- [ ] Error messages user-friendly
- [ ] Tests >90% coverage
- [ ] Security rules configured

## Related Skills
- `firebase-security-rules` - For Firestore/Database security
- `firebase-cloud-functions` - For server-side auth validation
- `testing-patterns` - For comprehensive auth testing
```

---

### Example 2: Supabase RLS Skill

```yaml
---
name: supabase-rls-security
description: Design and implement Supabase Row Level Security (RLS) policies with role-based access control. Validates policy logic and tests with different user roles. Use when securing Supabase tables, implementing permissions, or debugging RLS policy errors. Keywords: Supabase, RLS, security, permissions, policies, auth.
version: 1.5.0
---

# Supabase Row Level Security

## Core RLS Principles

1. **Deny by default**: Enable RLS, then explicitly grant access
2. **Test with different roles**: Authenticated, anon, admin
3. **Use auth.uid() for user-specific data**
4. **Validate with SQL functions for complex logic**

## Enable RLS

```sql
-- Always start by enabling RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;
```

## Common Patterns

### User-Owned Data

```sql
-- Users can only read/write their own data
CREATE POLICY "Users can view own profile"
  ON users
  FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON users
  FOR UPDATE
  USING (auth.uid() = id);
```

### Public Read, Owner Write

```sql
-- Anyone can read, only owner can modify
CREATE POLICY "Public read access"
  ON posts
  FOR SELECT
  USING (true);

CREATE POLICY "Owner can update"
  ON posts
  FOR UPDATE
  USING (auth.uid() = author_id);

CREATE POLICY "Owner can delete"
  ON posts
  FOR DELETE
  USING (auth.uid() = author_id);
```

### Role-Based Access

```sql
-- Create roles in auth.users metadata
-- { "role": "admin" | "moderator" | "user" }

CREATE POLICY "Admins have full access"
  ON users
  FOR ALL
  USING (
    (auth.jwt() -> 'user_metadata' ->> 'role') = 'admin'
  );

CREATE POLICY "Moderators can update posts"
  ON posts
  FOR UPDATE
  USING (
    (auth.jwt() -> 'user_metadata' ->> 'role') IN ('admin', 'moderator')
  );
```

### Multi-Tenant with Organization

```sql
-- Users can only access data from their organization
CREATE POLICY "Organization members can view"
  ON projects
  FOR SELECT
  USING (
    organization_id IN (
      SELECT organization_id
      FROM organization_members
      WHERE user_id = auth.uid()
    )
  );
```

## Testing RLS Policies

```sql
-- Test as authenticated user
SET request.jwt.claims = '{"sub": "user-id-123", "role": "authenticated"}';

-- Test as specific user
SET request.jwt.claim.sub = 'specific-user-id';

-- Test SELECT
SELECT * FROM posts;  -- Should only show accessible rows

-- Test INSERT
INSERT INTO posts (title, author_id) VALUES ('Test', auth.uid());

-- Test UPDATE
UPDATE posts SET title = 'Updated' WHERE id = 'post-id';

-- Test DELETE
DELETE FROM posts WHERE id = 'post-id';
```

## Validation Checklist
- [ ] RLS enabled on all tables
- [ ] Default deny (no USING (true) without auth check)
- [ ] Policies tested with different roles
- [ ] INSERT policies validate ownership
- [ ] UPDATE/DELETE check ownership
- [ ] Complex logic uses SQL functions
- [ ] No sensitive data leaked in policies
```

---

## Subagent Configurations

### Production-Ready Subagent: Code Reviewer

```yaml
---
name: code-reviewer
description: Expert code quality reviewer. Checks code for quality, security, performance, and maintainability issues. Use PROACTIVELY after implementation completes. Reviews against project standards and best practices.
tools: Read, Grep, Glob
model: sonnet
---

# Code Reviewer Agent

You are a senior software engineer conducting thorough code reviews. Your goal is to ensure code quality, security, and maintainability while being constructive and specific in feedback.

## Review Process

### 1. Read the Code
- Understand the context and purpose
- Check what files were modified
- Review the implementation approach

### 2. Quality Checks

**Code Structure**
- [ ] Clear, descriptive variable and function names
- [ ] Appropriate abstractions and separation of concerns
- [ ] No code duplication (DRY principle)
- [ ] Functions are focused and single-purpose
- [ ] Follows project conventions

**Security**
- [ ] No hardcoded secrets or credentials
- [ ] Input validation present
- [ ] SQL injection prevention (prepared statements)
- [ ] XSS prevention (sanitized outputs)
- [ ] Authentication and authorization checks

**Performance**
- [ ] No N+1 query problems
- [ ] Appropriate caching where beneficial
- [ ] Efficient algorithms and data structures
- [ ] No obvious memory leaks
- [ ] Database indexes considered

**Error Handling**
- [ ] Errors are caught and handled appropriately
- [ ] User-friendly error messages
- [ ] Logging for debugging
- [ ] No silent failures

**Testing**
- [ ] Tests present with >90% coverage
- [ ] Edge cases tested
- [ ] Error paths tested
- [ ] Tests are maintainable and clear
- [ ] Integration tests for critical paths

**Documentation**
- [ ] Public APIs documented (JSDoc/TSDoc)
- [ ] Complex logic has explanatory comments
- [ ] README updated if needed
- [ ] Type definitions complete

### 3. Provide Feedback

Format your review as:

## Code Review: [Feature/PR Name]

### ✅ Strengths
- [Specific things done well]
- [Good patterns used]
- [Excellent implementations]

### ⚠️ Issues Found

#### 🔴 Critical (Must Fix)
- [ ] [Issue with security/data loss/breaking change]
- [ ] [Another critical issue]

#### 🟡 Important (Should Fix)
- [ ] [Quality/performance concern]
- [ ] [Missing edge case]

#### 🔵 Nice-to-Have (Consider)
- [ ] [Refactoring suggestion]
- [ ] [Code style improvement]

### 📝 Detailed Comments

**File: src/path/to/file.ts**

Line 45-52:
```typescript
[Code snippet]
```
Concern: [Specific issue]
Suggestion: [How to fix]

### 🎯 Recommendation

[APPROVE / REQUEST CHANGES / NEEDS REWORK]

[Explanation of recommendation]

## Never Auto-Approve

If you find critical issues:
- Security vulnerabilities
- Data loss risks
- Breaking changes without migration
- No tests for critical functionality

Mark as REQUEST CHANGES and explain clearly.

## Be Constructive

- Praise good implementations
- Explain WHY something is problematic
- Suggest specific improvements
- Provide code examples for complex suggestions
- Consider the context and constraints
```

---

## Hook Scripts

### Complete Workflow Hook System

```bash
# .claude/hooks/workflow.sh
#!/bin/bash

set -e

EVENT=$1
QUEUE_FILE="enhancements/_queue.json"
LOG_FILE=".claude/hooks.log"
SLUG_FILE=".claude/current-slug"

# Colors for output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_event() {
  echo "[$(date -Iseconds)] $1" >> "$LOG_FILE"
}

get_current_slug() {
  if [[ -f "$SLUG_FILE" ]]; then
    cat "$SLUG_FILE"
  else
    echo "unknown"
  fi
}

update_queue_status() {
  local new_status=$1
  local slug=$(get_current_slug)

  node -e "
    const fs = require('fs');
    const queue = JSON.parse(fs.readFileSync('$QUEUE_FILE', 'utf8'));
    const item = queue.items.find(i => i.id === '$slug');
    if (item) {
      item.status = '$new_status';
      item.updated_at = new Date().toISOString();
      fs.writeFileSync('$QUEUE_FILE', JSON.stringify(queue, null, 2));
    }
  "
}

print_next_step() {
  echo ""
  echo -e "${GREEN}Next step (review and copy-paste if approved):${NC}"
  echo "  $1"
  echo ""
}

case $EVENT in
  "pm-spec-complete")
    local slug=$(get_current_slug)
    log_event "PM-Spec completed: $slug"
    update_queue_status "READY_FOR_ARCH"

    echo ""
    echo -e "${GREEN}✅ PM-Spec Complete${NC}"
    echo "📋 Spec: docs/specs/$slug.md"
    echo "📋 Slug: $slug"
    echo ""

    # Check for questions
    if grep -q "## Questions" "docs/specs/$slug.md"; then
      echo -e "${YELLOW}⚠️  Spec has questions - review before proceeding${NC}"
    fi

    print_next_step "claude --agents architect-review 'Review spec: $slug and create ADR'"
    ;;

  "architect-complete")
    local slug=$(get_current_slug)
    log_event "Architect-Review completed: $slug"
    update_queue_status "READY_FOR_BUILD"

    echo ""
    echo -e "${GREEN}✅ Architecture Review Complete${NC}"
    echo "📋 ADR: docs/decisions/ADR-$slug.md"
    echo ""

    # Verify ADR has required sections
    if ! grep -q "## Decision" "docs/decisions/ADR-$slug.md"; then
      echo -e "${YELLOW}⚠️  ADR missing Decision section${NC}"
    fi

    print_next_step "claude --agents implementer-tester 'Implement $slug per ADR'"
    ;;

  "implementer-complete")
    local slug=$(get_current_slug)
    log_event "Implementer-Tester completed: $slug"
    update_queue_status "READY_FOR_REVIEW"

    echo ""
    echo -e "${GREEN}✅ Implementation Complete${NC}"
    echo "📋 Code implemented"
    echo ""

    # Run tests
    echo "Running tests..."
    if npm test > /dev/null 2>&1; then
      echo -e "${GREEN}✅ Tests passing${NC}"

      # Check coverage
      coverage=$(npm run test:coverage 2>/dev/null | grep "All files" | grep -oP '\d+\.\d+' | head -1)
      if [[ -n "$coverage" ]]; then
        if (( $(echo "$coverage >= 90" | bc -l) )); then
          echo -e "${GREEN}✅ Coverage: $coverage%${NC}"
        else
          echo -e "${YELLOW}⚠️  Coverage: $coverage% (target: 90%)${NC}"
        fi
      fi
    else
      echo -e "${YELLOW}⚠️  Tests failing - review before proceeding${NC}"
    fi

    print_next_step "claude --agents code-reviewer 'Review implementation: $slug'"
    ;;

  "reviewer-complete")
    local slug=$(get_current_slug)
    log_event "Code-Reviewer completed: $slug"

    echo ""
    echo -e "${GREEN}✅ Code Review Complete${NC}"
    echo ""
    echo "If approved, ready to:"
    echo "1. Create feature branch: feat/$slug"
    echo "2. Commit changes"
    echo "3. Create PR"
    echo ""
    print_next_step "git checkout -b feat/$slug && git add . && git commit -m 'feat: $slug' && git push -u origin feat/$slug"
    ;;

  *)
    echo "Unknown event: $EVENT"
    exit 1
    ;;
esac
```

---

(Document continues with MCP server examples and workflow patterns - truncated for length. The complete examples document would include all sections listed in the TOC.)

---

**Next**: Read [07-ADDITIONAL-INSIGHTS.md](07-ADDITIONAL-INSIGHTS.md) for insights beyond your initial plan
