# Claude Skills Architecture

## Overview

Claude Skills are a sophisticated prompt-based meta-tool architecture that extends LLM capabilities through specialized instruction injection. Unlike traditional function calling or code execution, skills operate through **progressive disclosure** and **context modification**.

**Key Insight**: Skills load on-demand based on task requirements, keeping context lean while providing specialized expertise when needed.

## Core Concept: Progressive Disclosure

Skills use a 3-level loading strategy to minimize token usage:

### Level 1: Startup (Minimal)
- Claude scans all available skills at session start
- Loads only **name** and **description** (30-50 tokens per skill)
- Skills remain dormant until activated

### Level 2: Activation (Full Skill)
- When Claude determines a skill is needed, the system:
  - Loads the full SKILL.md markdown file
  - Expands instructions into detailed guidance
  - Injects as new user messages in conversation context
  - Modifies execution context (tools, model selection)
- Skill instructions are now active

### Level 3: Resources (On-Demand)
- Supporting files loaded only when explicitly accessed
- Subdirectory memory files load when accessing those directories
- Progressive loading prevents token waste

**Token Efficiency Example**:
```
100 skills at Level 1: 100 × 40 tokens = 4,000 tokens
1 skill activated to Level 2: 4,000 + 2,000 = 6,000 tokens
Resources loaded at Level 3: 6,000 + 1,000 = 7,000 tokens

vs. Loading all 100 skills fully: 100 × 2,000 = 200,000 tokens
Savings: 193,000 tokens (~96% reduction)
```

## Skill Structure

### File Organization

```
skills/
├── SKILL.md                    # Main skill instructions
├── scripts/                    # Optional executables
│   ├── script1.py
│   └── script2.sh
└── resources/                  # Supporting files
    ├── patterns.md
    ├── examples/
    │   └── example1.md
    └── reference/
        └── docs.md
```

### SKILL.md Format

```yaml
---
name: skill-identifier
description: When to activate this skill (be specific)
tools: optional, comma-separated list
version: 1.0.0
tags: category1, category2
---

# Skill Name

Brief overview of what this skill provides.

## When to Use

- [Use case 1]
- [Use case 2]
- [Use case 3]

## Instructions

Detailed guidance for this domain...

## Patterns

Common patterns for this skill...

## Pitfalls

What to avoid...

## Examples

Practical examples...
```

### Example Skills

#### 1. Firebase Authentication Skill

```yaml
---
name: firebase-auth
description: Use PROACTIVELY when implementing Firebase Authentication, user signup, login, password reset, or session management
tools: Read,Write,Edit,Bash
version: 1.2.0
tags: firebase, authentication, security
---

# Firebase Authentication Skill

Specialized guidance for implementing Firebase Authentication correctly and securely.

## When to Use

- Implementing user signup/login
- Adding social authentication (Google, GitHub, etc.)
- Password reset flows
- Session management
- Authentication state handling
- Security rules involving auth

## Firebase Auth Setup

### 1. Initialize Firebase

```typescript
// firebase.config.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  // ... other config
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

**Critical**: Never hardcode credentials, always use environment variables.

### 2. Email/Password Authentication

```typescript
// auth.service.ts
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut,
  sendPasswordResetEmail
} from 'firebase/auth';
import { auth } from './firebase.config';

export async function signUp(email: string, password: string) {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email, password);
    return { success: true, user: userCredential.user };
  } catch (error) {
    return { success: false, error: handleAuthError(error) };
  }
}

export async function signIn(email: string, password: string) {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    return { success: true, user: userCredential.user };
  } catch (error) {
    return { success: false, error: handleAuthError(error) };
  }
}

export async function logOut() {
  await signOut(auth);
}

export async function resetPassword(email: string) {
  try {
    await sendPasswordResetEmail(auth, email);
    return { success: true };
  } catch (error) {
    return { success: false, error: handleAuthError(error) };
  }
}

function handleAuthError(error: any): string {
  const errorCode = error.code;
  switch (errorCode) {
    case 'auth/email-already-in-use':
      return 'Email already in use';
    case 'auth/invalid-email':
      return 'Invalid email address';
    case 'auth/weak-password':
      return 'Password is too weak';
    case 'auth/user-not-found':
      return 'User not found';
    case 'auth/wrong-password':
      return 'Incorrect password';
    default:
      return 'Authentication error occurred';
  }
}
```

### 3. Authentication State Listener

```typescript
// useAuth.ts (Svelte store or React hook)
import { onAuthStateChanged, type User } from 'firebase/auth';
import { auth } from './firebase.config';

export function createAuthStore() {
  let currentUser: User | null = null;
  const subscribers: ((user: User | null) => void)[] = [];

  onAuthStateChanged(auth, (user) => {
    currentUser = user;
    subscribers.forEach(sub => sub(user));
  });

  return {
    subscribe(callback: (user: User | null) => void) {
      subscribers.push(callback);
      callback(currentUser);
      return () => {
        const index = subscribers.indexOf(callback);
        if (index >= 0) subscribers.splice(index, 1);
      };
    },
    get current() {
      return currentUser;
    }
  };
}

export const authStore = createAuthStore();
```

## Common Patterns

### Protected Routes

```typescript
// ProtectedRoute.svelte
<script lang="ts">
  import { authStore } from './useAuth';
  import { navigate } from 'svelte-routing';

  $: if (!$authStore) {
    navigate('/login');
  }
</script>

{#if $authStore}
  <slot />
{/if}
```

### Social Authentication

```typescript
import { signInWithPopup, GoogleAuthProvider } from 'firebase/auth';

export async function signInWithGoogle() {
  const provider = new GoogleAuthProvider();
  try {
    const result = await signInWithPopup(auth, provider);
    return { success: true, user: result.user };
  } catch (error) {
    return { success: false, error: handleAuthError(error) };
  }
}
```

## Security Best Practices

1. **Environment Variables**: Never commit Firebase config to git
2. **Error Handling**: Always catch and handle auth errors gracefully
3. **Password Requirements**: Enforce strong password policies
4. **Rate Limiting**: Implement on server-side (Firebase Functions)
5. **Security Rules**: Lock down Firestore based on auth.uid
6. **Token Verification**: Verify ID tokens on backend
7. **Session Persistence**: Choose appropriate persistence (LOCAL/SESSION/NONE)

## Common Pitfalls

❌ **Don't**:
- Hardcode API keys in source files
- Trust client-side auth state for security decisions
- Skip error handling
- Forget to clean up auth listeners
- Use weak password requirements

✅ **Do**:
- Use environment variables
- Verify auth on backend
- Handle all error cases
- Unsubscribe from listeners on cleanup
- Enforce password strength

## Testing

```typescript
// auth.test.ts
import { describe, it, expect, vi } from 'vitest';
import { signUp, signIn } from './auth.service';

describe('Authentication', () => {
  it('should sign up new user', async () => {
    const result = await signUp('test@example.com', 'password123');
    expect(result.success).toBe(true);
  });

  it('should handle duplicate email', async () => {
    // First signup
    await signUp('test@example.com', 'password123');

    // Duplicate signup
    const result = await signUp('test@example.com', 'password123');
    expect(result.success).toBe(false);
    expect(result.error).toContain('already in use');
  });

  it('should reject weak password', async () => {
    const result = await signUp('test@example.com', '123');
    expect(result.success).toBe(false);
    expect(result.error).toContain('weak');
  });
});
```

## Resources

- [Firebase Auth Documentation](https://firebase.google.com/docs/auth)
- [Security Rules Guide](https://firebase.google.com/docs/rules)
- [Best Practices](https://firebase.google.com/docs/auth/web/start)
```

#### 2. Testing Skill

```yaml
---
name: testing
description: Use PROACTIVELY when writing tests or ensuring test coverage for new features
tools: Read,Write,Edit,Bash
version: 1.0.0
tags: testing, quality, vitest, jest
---

# Testing Skill

Comprehensive guidance for writing effective tests.

## Testing Philosophy

- **Test behavior, not implementation**
- **Cover edge cases thoroughly**
- **Write clear, descriptive test names**
- **One assertion per test when possible**
- **Use arrange-act-assert pattern**

## Test Structure

### Naming Convention

```typescript
describe('ComponentName', () => {
  describe('methodName', () => {
    it('should [expected behavior] when [condition]', () => {
      // Test implementation
    });
  });
});
```

### AAA Pattern

```typescript
it('should calculate total price with tax', () => {
  // Arrange
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 3 }
  ];
  const taxRate = 0.1;

  // Act
  const total = calculateTotal(items, taxRate);

  // Assert
  expect(total).toBe(38.5); // (20 + 15) * 1.1
});
```

## Coverage Requirements

### 1. Happy Path
Test normal, expected usage:

```typescript
it('should add item to cart', () => {
  const cart = new Cart();
  const item = { id: 1, name: 'Product', price: 10 };

  cart.addItem(item);

  expect(cart.items).toHaveLength(1);
  expect(cart.items[0]).toEqual(item);
});
```

### 2. Edge Cases
Test boundaries and limits:

```typescript
it('should handle empty cart', () => {
  const cart = new Cart();
  expect(cart.getTotal()).toBe(0);
});

it('should handle maximum quantity', () => {
  const cart = new Cart();
  const item = { id: 1, name: 'Product', price: 10 };

  cart.addItem(item, 999);

  expect(cart.items[0].quantity).toBe(999);
});

it('should handle zero price', () => {
  const cart = new Cart();
  const item = { id: 1, name: 'Free Item', price: 0 };

  cart.addItem(item);

  expect(cart.getTotal()).toBe(0);
});
```

### 3. Error Cases
Test invalid inputs and failure scenarios:

```typescript
it('should throw error for negative quantity', () => {
  const cart = new Cart();
  const item = { id: 1, name: 'Product', price: 10 };

  expect(() => cart.addItem(item, -1)).toThrow('Quantity must be positive');
});

it('should handle network failure gracefully', async () => {
  const api = new API();
  vi.spyOn(api, 'fetch').mockRejectedValue(new Error('Network error'));

  const result = await api.getUser(1);

  expect(result.success).toBe(false);
  expect(result.error).toBe('Network error');
});
```

### 4. Integration Tests
Test component interactions:

```typescript
it('should complete checkout flow', async () => {
  const cart = new Cart();
  const payment = new PaymentProcessor();
  const checkout = new CheckoutService(cart, payment);

  cart.addItem({ id: 1, name: 'Product', price: 10 });
  const result = await checkout.process({ cardNumber: '4242...' });

  expect(result.success).toBe(true);
  expect(cart.items).toHaveLength(0); // Cart cleared
  expect(payment.lastTransaction).toBeDefined();
});
```

## Common Patterns

### Mock External Dependencies

```typescript
import { vi } from 'vitest';

it('should call API with correct parameters', async () => {
  const mockFetch = vi.fn().mockResolvedValue({ id: 1, name: 'User' });
  const api = new API(mockFetch);

  await api.getUser(1);

  expect(mockFetch).toHaveBeenCalledWith('/users/1');
});
```

### Test Async Operations

```typescript
it('should wait for async operation', async () => {
  const result = await fetchData();
  expect(result).toBeDefined();
});

it('should timeout long operations', async () => {
  await expect(longRunningTask()).rejects.toThrow('Timeout');
}, 5000); // 5 second timeout
```

### Component Testing (Svelte)

```typescript
import { render, fireEvent } from '@testing-library/svelte';
import Button from './Button.svelte';

it('should emit click event', async () => {
  const { component, getByRole } = render(Button, { label: 'Click me' });

  let clicked = false;
  component.$on('click', () => { clicked = true; });

  const button = getByRole('button');
  await fireEvent.click(button);

  expect(clicked).toBe(true);
});
```

## Best Practices

1. **Descriptive Names**: Test name should describe what's being tested
2. **Isolation**: Each test should be independent
3. **Fast**: Tests should run quickly
4. **Deterministic**: Same test should always produce same result
5. **Maintainable**: Easy to understand and update

## Anti-Patterns

❌ **Avoid**:
- Testing implementation details
- Multiple assertions testing different things
- Tests that depend on each other
- Excessive mocking (mock only what you must)
- Testing the framework/library itself

✅ **Prefer**:
- Testing public interfaces
- One concept per test
- Independent, isolated tests
- Real implementations when feasible
- Testing your own code
```

## Skill Selection Mechanism

### LLM-Based Routing (No Algorithmic Selection)

Claude Code doesn't use embeddings, classifiers, or pattern matching to decide which skill to invoke. Instead:

1. **All skills formatted into text description**
2. **Embedded in Skill tool's prompt**
3. **Claude's language model makes the decision**

This means skill descriptions must be:
- Clear and specific
- Action-oriented
- Include relevant keywords
- Mention specific use cases

### Example Selection Process

**User Prompt**: "Implement Firebase authentication"

**Claude's reasoning** (internal):
1. Scan skill descriptions
2. Identify "firebase-auth" skill mentions "Firebase Authentication"
3. Description says "use PROACTIVELY when implementing Firebase Authentication"
4. Activate firebase-auth skill
5. Load full SKILL.md instructions
6. Proceed with implementation using skill guidance

## Context Injection Mechanism

Skills inject instructions into conversation context using:

```yaml
role: "user"
isMeta: true
```

This makes the skill prompt appear as user input to Claude, keeping it:
- **Temporary**: Localized to current interaction
- **Non-persistent**: After skill completes, conversation returns to normal
- **Clean**: No residual behavioral modifications

## Best Practices for Skill Design

### 1. Write Specific Descriptions

```yaml
# ❌ BAD
description: Helps with Firebase

# ✅ GOOD
description: Use PROACTIVELY when implementing Firebase Authentication (signup, login, password reset) or managing user sessions
```

### 2. Include Action Triggers

Use phrases that trigger automatic activation:
- "Use PROACTIVELY when..."
- "MUST BE USED for..."
- "ALWAYS load when..."
- "REQUIRED for..."

### 3. Define Clear Scope

```yaml
# ❌ BAD - Too broad
name: backend
description: Backend development help

# ✅ GOOD - Focused
name: firebase-firestore-queries
description: Use when optimizing Firestore queries, compound queries, or troubleshooting query performance
```

### 4. Structure for Scanability

Use clear headings and sections:
```markdown
## When to Use
## Patterns
## Pitfalls
## Examples
## Security Considerations
```

### 5. Include Practical Examples

Don't just explain - show working code:

```markdown
## Pattern: Firestore Query Optimization

Instead of:
❌ Multiple reads

Do this:
✅ Single query with compound index

[Include actual code example]
```

### 6. Leverage Progressive Resources

```
SKILL.md (2KB) - Main instructions, always loaded when skill activates
resources/patterns.md (10KB) - Loaded only if Claude needs pattern reference
resources/examples/ (50KB) - Loaded only when specific example needed
```

### 7. Version Skills

```yaml
version: 1.2.0
changelog:
  - 1.2.0: Added security rules patterns
  - 1.1.0: Added compound query examples
  - 1.0.0: Initial release
```

### 8. Tag for Discovery

```yaml
tags: firebase, firestore, database, queries, optimization, performance
```

## Testing Skills

### Manual Testing

```
User: "Implement Firebase authentication"
Expected: firebase-auth skill should activate
Verify: Check if skill instructions appear in context
```

### Effectiveness Metrics

Track:
- **Activation rate**: How often skill activates when expected
- **Quality improvement**: Code quality with vs without skill
- **Token efficiency**: Tokens saved by focused guidance
- **Time to completion**: Faster with skill guidance?
- **Error rate**: Fewer mistakes with skill?

## Integration with Project Goals

### MINIMIZE Principle

**Skills minimize tokens**:
- Only 30-50 tokens at Level 1 (dormant)
- Full 2,000 tokens only when activated
- Resources loaded on-demand

**Example**:
```
100 skills available: 4,000 tokens (dormant)
Task needs 2 skills: 4,000 + 4,000 = 8,000 tokens

vs. Loading all 100: 200,000 tokens
Savings: 192,000 tokens (96%)
```

### SEPARATE Principle

**Skills maintain separation**:
- Domain-specific knowledge isolated in skills
- Core .cursorrules stays minimal
- Each skill has clear boundary
- Skills don't conflict with each other

### VALIDATE Principle

**Skills support validation**:
- Each skill can be A/B tested
- Measure quality with vs without skill
- Track skill activation and outcomes
- Data-driven skill improvement

**Experiment**:
```
Control: No skills loaded
Treatment: firebase-auth skill active
Measure: Code quality, time, errors
```

### LEARN Principle

**Skills enable learning**:
- Capture successful patterns in skills
- Evolve skills based on team experience
- Share skills across projects
- Version control skill improvements

## Skill Library Organization

### Recommended Structure

```
.claude/skills/
├── core/                          # Fundamental skills
│   ├── testing.skill.md
│   ├── refactoring.skill.md
│   └── debugging.skill.md
├── frameworks/                    # Framework-specific
│   ├── svelte.skill.md
│   ├── htmx.skill.md
│   └── ionic.skill.md
├── infrastructure/                # Backend services
│   ├── firebase/
│   │   ├── auth.skill.md
│   │   ├── firestore.skill.md
│   │   └── functions.skill.md
│   ├── supabase/
│   │   ├── rls.skill.md
│   │   ├── queries.skill.md
│   │   └── rpc.skill.md
│   └── postgres.skill.md
├── patterns/                      # Design patterns
│   ├── repository-pattern.skill.md
│   ├── observer-pattern.skill.md
│   └── factory-pattern.skill.md
└── security/                      # Security-focused
    ├── input-validation.skill.md
    ├── authentication.skill.md
    └── encryption.skill.md
```

## Performance Considerations

### Activation Overhead

```
Skill activation time: ~100-200ms
Benefit: Focused guidance, fewer errors, less rework
Net impact: Positive (saves time overall)
```

### Token Budget

```
Session budget: 200k tokens
Dormant skills: 4k tokens (100 skills × 40 tokens)
Budget remaining: 196k tokens
Impact: Negligible (~2%)
```

### Scaling

```
1-10 skills: No noticeable impact
10-50 skills: <5% of token budget
50-100 skills: ~5-10% of token budget
100+ skills: Consider organizing into subcategories
```

## References

- [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)
- [Claude Skills Technical Overview](https://medium.com/data-science-collective/claude-skills-a-technical-deep-dive-into-context-injection-architecture-ee6bf30cf514)
- [Complete Guide to Claude Skills](https://tylerfolkman.substack.com/p/the-complete-guide-to-claude-skills)
- [Claude Code Infrastructure Showcase](https://github.com/diet103/claude-code-infrastructure-showcase)

---

**Last Updated**: 2025-11-03
