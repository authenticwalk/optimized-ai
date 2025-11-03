# Implementation Examples

**Last Updated**: November 3, 2025

---

## Quick Navigation

1. [Subagent Definitions](#subagent-definitions)
2. [Skill Files](#skill-files)
3. [MCP Servers](#mcp-servers)
4. [Orchestration Code](#orchestration-code)
5. [Complete Workflows](#complete-workflows)

---

## Subagent Definitions

### Example 1: Security Reviewer (YAML Frontmatter)

**File**: `.claude/agents/security-reviewer.md`

```markdown
---
name: security-reviewer
description: Expert security reviewer for web applications, focusing on OWASP Top 10 vulnerabilities including SQL injection, XSS, CSRF, and authentication bypasses
tools: Read, Grep, Glob
model: claude-3-5-sonnet-20241022
version: "1.0"
author: Optimized AI Team
---

# Security Reviewer Agent

You are a security expert with 15 years of experience in web application security.

## Focus Areas

### Primary Concerns
- SQL injection and NoSQL injection
- Cross-site scripting (XSS) - stored, reflected, DOM-based
- Cross-site request forgery (CSRF)
- Authentication and session management flaws
- Authorization and access control issues
- Security misconfiguration
- Sensitive data exposure
- Insecure deserialization
- Components with known vulnerabilities
- Insufficient logging and monitoring

### Code Review Process

For each file you review:

1. **Identify Vulnerabilities**
   - Scan for common vulnerability patterns
   - Check OWASP Top 10 issues
   - Review authentication and authorization flows
   - Examine input validation
   - Analyze output encoding

2. **Assess Severity**
   - **Critical**: Immediate exploitation possible, high impact
   - **High**: Exploitation likely with minimal effort
   - **Medium**: Exploitation requires specific conditions
   - **Low**: Minor security improvements

3. **Provide Remediation**
   - Specific code changes needed
   - Reference to security best practices
   - Example of secure implementation
   - Links to relevant OWASP articles

## Output Format

Return findings in this structure:

```json
{
  "file": "path/to/file.ts",
  "line": 142,
  "vulnerability": "SQL Injection",
  "severity": "Critical",
  "description": "User input is directly concatenated into SQL query",
  "code_snippet": "const query = `SELECT * FROM users WHERE id = ${userId}`",
  "remediation": "Use parameterized queries",
  "example": "const query = 'SELECT * FROM users WHERE id = ?'; db.query(query, [userId])",
  "reference": "https://owasp.org/www-community/attacks/SQL_Injection"
}
```

## Guidelines

- **Only report Critical and High severity findings** (unless specifically asked for all)
- Provide specific, actionable remediation steps
- Include code examples where helpful
- Reference OWASP or security standards
- Be concise but complete

## Common Patterns to Watch For

### SQL Injection
```typescript
// ❌ VULNERABLE
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;

// ✅ SECURE
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [req.params.id]);
```

### XSS
```typescript
// ❌ VULNERABLE
element.innerHTML = userInput;

// ✅ SECURE
element.textContent = userInput;
// or use proper sanitization
```

### Authentication
```typescript
// ❌ VULNERABLE
if (password === storedPassword) { /* ... */ }

// ✅ SECURE
if (await bcrypt.compare(password, storedPasswordHash)) { /* ... */ }
```
```

---

### Example 2: Implementer Agent (Programmatic)

**File**: `agents/implementer.ts`

```typescript
import { AgentDefinition } from '@anthropic-ai/claude-agent-sdk';

export const implementerAgent: AgentDefinition = {
  description: 'Implements features based on specifications, following project patterns and best practices',

  prompt: `You are an expert software implementer.

## Your Role

You implement features based on specifications provided to you. You follow project patterns, coding standards, and best practices.

## Process

1. **Understand Requirements**
   - Read the specification carefully
   - Identify key requirements
   - Note edge cases and constraints

2. **Plan Implementation**
   - Determine files to create/modify
   - Identify dependencies
   - Plan code structure

3. **Implement**
   - Write clean, maintainable code
   - Follow project conventions
   - Add appropriate error handling
   - Include inline comments for complex logic

4. **Self-Review**
   - Check for bugs
   - Verify all requirements met
   - Ensure code quality

## Code Quality Standards

- Use TypeScript strict mode
- Follow single responsibility principle
- Keep functions small and focused
- Use descriptive names
- Add error handling
- No hardcoded values
- Use project's existing patterns

## Project Context

Load relevant skills for the technology stack being used (e.g., firebase-auth skill for authentication work).

## Output

Return:
- List of files created/modified
- Summary of changes
- Any assumptions made
- Suggestions for testing
`,

  tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob', 'Bash'],

  model: 'sonnet'
};
```

---

### Example 3: Orchestrator Agent

**File**: `agents/project-coordinator.ts`

```typescript
export const projectCoordinatorAgent: AgentDefinition = {
  description: 'Coordinates complex multi-step projects by delegating to specialized agents',

  prompt: `You are a project coordinator for software development.

## Your Role

You coordinate complex projects by:
1. Breaking down tasks into subtasks
2. Delegating to appropriate specialized agents
3. Tracking progress
4. Synthesizing results
5. Making decisions

## Available Specialized Agents

- **planner**: Creates detailed implementation plans
- **implementer**: Writes code
- **security-reviewer**: Reviews security
- **quality-reviewer**: Reviews code quality
- **test-engineer**: Creates tests
- **documenter**: Writes documentation

## Your Process

1. **Analyze Task**
   - Understand overall objective
   - Identify major components
   - Determine dependencies

2. **Create Plan**
   - Break into subtasks
   - Identify which agents needed
   - Determine sequence (sequential vs parallel)

3. **Delegate**
   - Give each agent clear, specific task
   - Provide necessary context
   - Set expectations for output

4. **Track & Synthesize**
   - Monitor progress
   - Integrate results
   - Resolve conflicts

5. **Deliver**
   - Ensure all requirements met
   - Provide summary
   - Highlight any issues

## Important

- You do NOT implement code yourself
- You delegate to appropriate agents
- You keep context focused on coordination
- You return high-level summaries, not detailed work

## Output Format

```json
{
  "status": "completed" | "in_progress" | "blocked",
  "summary": "High-level summary of what was accomplished",
  "subtasks": [
    {
      "agent": "planner",
      "status": "completed",
      "result": "Created implementation plan"
    },
    // ...
  ],
  "deliverables": ["file1.ts", "file2.ts"],
  "issues": ["Any blockers or concerns"],
  "next_steps": ["What should happen next"]
}
```
`,

  tools: ['Task', 'Read', 'Grep'],  // Can delegate and gather info, not modify code

  model: 'opus'  // Best reasoning for coordination
};
```

---

## Skill Files

### Example 1: Firebase Authentication Skill

**File**: `.claude/skills/firebase-auth/SKILL.md`

```markdown
---
name: firebase-auth
description: Firebase authentication patterns for email/password signup, login, session management, and error handling
version: "1.0.0"
---

# Firebase Authentication Skill

## Quick Start

### Initialization
```typescript
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: process.env.FIREBASE_AUTH_DOMAIN,
  projectId: process.env.FIREBASE_PROJECT_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

### Signup
```typescript
import { createUserWithEmailAndPassword } from 'firebase/auth';

async function signup(email: string, password: string) {
  try {
    const credential = await createUserWithEmailAndPassword(auth, email, password);
    return credential.user;
  } catch (error) {
    throw handleAuthError(error);
  }
}
```

### Login
```typescript
import { signInWithEmailAndPassword } from 'firebase/auth';

async function login(email: string, password: string) {
  try {
    const credential = await signInWithEmailAndPassword(auth, email, password);
    return credential.user;
  } catch (error) {
    throw handleAuthError(error);
  }
}
```

### Session Management
```typescript
import { onAuthStateChanged } from 'firebase/auth';

onAuthStateChanged(auth, (user) => {
  if (user) {
    // User is signed in
    console.log('User:', user.uid);
  } else {
    // User is signed out
    console.log('No user');
  }
});
```

## Error Handling

Common Firebase Auth error codes:

| Error Code | Meaning | User Message |
|------------|---------|--------------|
| `auth/email-already-in-use` | Email exists | "An account with this email already exists" |
| `auth/invalid-email` | Invalid format | "Please enter a valid email address" |
| `auth/weak-password` | Password too weak | "Password must be at least 6 characters" |
| `auth/user-not-found` | No account | "No account found with this email" |
| `auth/wrong-password` | Incorrect password | "Incorrect password" |
| `auth/too-many-requests` | Rate limited | "Too many attempts. Please try again later" |

### Error Handler Function
```typescript
function handleAuthError(error: any): Error {
  const errorMessages: Record<string, string> = {
    'auth/email-already-in-use': 'An account with this email already exists',
    'auth/invalid-email': 'Please enter a valid email address',
    'auth/weak-password': 'Password must be at least 6 characters',
    'auth/user-not-found': 'No account found with this email',
    'auth/wrong-password': 'Incorrect password',
    'auth/too-many-requests': 'Too many attempts. Please try again later'
  };

  const message = errorMessages[error.code] || 'An error occurred. Please try again.';
  return new Error(message);
}
```

## Security Best Practices

1. **Never store passwords in plain text**
2. **Always use HTTPS**
3. **Validate email format before calling Firebase**
4. **Enforce strong password requirements**
5. **Rate limit authentication attempts**
6. **Use secure, httpOnly cookies for tokens**
7. **Implement session timeout**
8. **Log security events**

## Testing

### Unit Test Example
```typescript
import { describe, it, expect, vi } from 'vitest';
import { signup } from './auth';

describe('signup', () => {
  it('should create user with valid credentials', async () => {
    const email = 'test@example.com';
    const password = 'securePassword123';

    const user = await signup(email, password);

    expect(user).toBeDefined();
    expect(user.email).toBe(email);
  });

  it('should throw error for invalid email', async () => {
    await expect(signup('invalid', 'password123'))
      .rejects
      .toThrow('Please enter a valid email address');
  });
});
```

## For More Details

- Advanced patterns: See [patterns/advanced.md](patterns/advanced.md)
- OAuth integration: See [patterns/oauth.md](patterns/oauth.md)
- Multi-factor auth: See [patterns/mfa.md](patterns/mfa.md)
```

---

## MCP Servers

### Example 1: Knowledge Base MCP Server

**File**: `mcp-servers/knowledge-base/index.ts`

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { readFileSync, readdirSync } from 'fs';
import { join } from 'path';

const KNOWLEDGE_BASE_PATH = '.ai-knowledge';

const server = new Server(
  {
    name: 'optimized-ai-knowledge-base',
    version: '1.0.0',
  },
  {
    capabilities: {
      resources: {},
      tools: {}
    },
  }
);

// List available patterns
server.setRequestHandler('resources/list', async () => {
  const patternsDir = join(KNOWLEDGE_BASE_PATH, 'patterns');
  const files = readdirSync(patternsDir);

  return {
    resources: files
      .filter(f => f.endsWith('.json'))
      .map(f => ({
        uri: `knowledge://patterns/${f.replace('.json', '')}`,
        name: f.replace('.json', '').replace(/-/g, ' '),
        mimeType: 'application/json',
        description: `Pattern: ${f.replace('.json', '')}`
      }))
  };
});

// Read specific pattern
server.setRequestHandler('resources/read', async (request) => {
  const { uri } = request.params;

  if (!uri.startsWith('knowledge://patterns/')) {
    throw new Error(`Invalid URI: ${uri}`);
  }

  const patternName = uri.replace('knowledge://patterns/', '');
  const filePath = join(KNOWLEDGE_BASE_PATH, 'patterns', `${patternName}.json`);

  try {
    const content = readFileSync(filePath, 'utf-8');
    return {
      contents: [
        {
          uri,
          mimeType: 'application/json',
          text: content
        }
      ]
    };
  } catch (error) {
    throw new Error(`Pattern not found: ${patternName}`);
  }
});

// Save new pattern (tool)
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'save_pattern',
      description: 'Save a successful pattern to the knowledge base',
      inputSchema: {
        type: 'object',
        properties: {
          name: {
            type: 'string',
            description: 'Pattern name (e.g., "firebase-auth")'
          },
          category: {
            type: 'string',
            description: 'Pattern category (e.g., "authentication")'
          },
          pattern: {
            type: 'object',
            description: 'Pattern details'
          }
        },
        required: ['name', 'category', 'pattern']
      }
    }
  ]
}));

server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  if (name === 'save_pattern') {
    const { name: patternName, category, pattern } = args;

    const filePath = join(KNOWLEDGE_BASE_PATH, 'patterns', `${patternName}.json`);

    const data = {
      name: patternName,
      category,
      createdAt: new Date().toISOString(),
      pattern
    };

    writeFileSync(filePath, JSON.stringify(data, null, 2));

    return {
      content: [
        {
          type: 'text',
          text: `Pattern saved: ${patternName}`
        }
      ]
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);

console.error('Knowledge Base MCP Server running');
```

**Configuration**: `.claude/mcp.json`

```json
{
  "mcpServers": {
    "knowledge-base": {
      "command": "node",
      "args": ["./mcp-servers/knowledge-base/dist/index.js"]
    }
  }
}
```

---

## Orchestration Code

### Example: Code Review Orchestration

**File**: `workflows/code-review.ts`

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

async function orchestrateCodeReview(files: string[]) {
  // Step 1: Orchestrator analyzes scope
  const orchestrator = await query({
    prompt: `Analyze these files for code review: ${files.join(', ')}

Determine:
1. Which files need security review
2. Which files need quality review
3. Which files need performance review

Return a JSON object with categorized files.`,
    options: {
      model: 'opus'
    }
  });

  const scope = JSON.parse(orchestrator.text);

  // Step 2: Parallel reviews
  const [securityResults, qualityResults, performanceResults] = await Promise.all([
    // Security review
    query({
      prompt: `Review these files for security issues: ${scope.security_files.join(', ')}

Focus on OWASP Top 10 vulnerabilities.
Return only critical and high severity findings.`,
      options: {
        agents: {
          'security-reviewer': {
            description: 'Expert security reviewer',
            prompt: securityReviewerPrompt,
            tools: ['Read', 'Grep', 'Glob'],
            model: 'sonnet'
          }
        }
      }
    }),

    // Quality review
    query({
      prompt: `Review these files for code quality: ${scope.quality_files.join(', ')}

Focus on maintainability, clean code principles, and best practices.`,
      options: {
        agents: {
          'quality-reviewer': {
            description: 'Code quality reviewer',
            prompt: qualityReviewerPrompt,
            tools: ['Read', 'Grep', 'Glob'],
            model: 'sonnet'
          }
        }
      }
    }),

    // Performance review
    query({
      prompt: `Review these files for performance: ${scope.performance_files.join(', ')}

Identify optimization opportunities and performance bottlenecks.`,
      options: {
        agents: {
          'performance-reviewer': {
            description: 'Performance reviewer',
            prompt: performanceReviewerPrompt,
            tools: ['Read', 'Grep', 'Glob', 'Bash'],
            model: 'sonnet'
          }
        }
      }
    })
  ]);

  // Step 3: Synthesize results
  const synthesis = await query({
    prompt: `Synthesize these review findings:

Security: ${securityResults.text}
Quality: ${qualityResults.text}
Performance: ${performanceResults.text}

Create a prioritized list of issues to address.
Group by severity and category.
Provide actionable recommendations.`,
    options: {
      model: 'opus'
    }
  });

  return {
    security: JSON.parse(securityResults.text),
    quality: JSON.parse(qualityResults.text),
    performance: JSON.parse(performanceResults.text),
    synthesis: synthesis.text
  };
}

// Usage
const results = await orchestrateCodeReview([
  'src/auth/login.ts',
  'src/auth/register.ts',
  'src/api/users.ts'
]);

console.log(results.synthesis);
```

---

## Complete Workflows

### Example: Feature Implementation Workflow

```typescript
async function implementFeature(requirement: string) {
  // Phase 1: Planning
  const plan = await query({
    prompt: `Create a detailed implementation plan for: ${requirement}

Include:
- Files to create/modify
- Dependencies
- Edge cases
- Testing strategy
- Security considerations`,
    options: {
      agents: {
        'planner': {
          description: 'Expert technical planner',
          prompt: plannerPrompt,
          tools: ['Read', 'Grep', 'Glob'],
          model: 'opus'
        }
      }
    }
  });

  // Phase 2: Implementation
  const implementation = await query({
    prompt: `Implement this plan:\n\n${plan.text}

Follow project conventions and best practices.
Load relevant skills for the technology stack.`,
    options: {
      agents: {
        'implementer': {
          description: 'Expert implementer',
          prompt: implementerPrompt,
          tools: ['Read', 'Edit', 'Write', 'Grep', 'Glob', 'Bash'],
          model: 'sonnet'
        }
      }
    }
  });

  // Phase 3: Testing
  const tests = await query({
    prompt: `Create comprehensive tests for this implementation:\n\n${implementation.text}

Include:
- Unit tests
- Integration tests
- Edge case tests`,
    options: {
      agents: {
        'test-engineer': {
          description: 'Expert test engineer',
          prompt: testEngineerPrompt,
          tools: ['Read', 'Write', 'Bash', 'Grep'],
          model: 'sonnet'
        }
      }
    }
  });

  // Phase 4: Review
  const review = await query({
    prompt: `Review this implementation:

Plan: ${plan.text}
Implementation: ${implementation.text}
Tests: ${tests.text}

Provide feedback on:
- Correctness
- Security
- Quality
- Completeness`,
    options: {
      agents: {
        'reviewer': {
          description: 'Expert code reviewer',
          prompt: reviewerPrompt,
          tools: ['Read', 'Grep', 'Glob', 'Bash'],
          model: 'opus'
        }
      }
    }
  });

  return {
    plan: plan.text,
    implementation: implementation.text,
    tests: tests.text,
    review: review.text
  };
}
```

---

## Key Takeaways

1. **YAML frontmatter** is standard for filesystem-based agents
2. **Programmatic definitions** offer more flexibility
3. **Clear prompts** with step-by-step processes work best
4. **Tool restrictions** enforce least privilege
5. **MCP servers** standardize data access
6. **Orchestration** coordinates multiple agents
7. **Parallel execution** significantly speeds up workflows
8. **Structured outputs** make integration easier

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Subagent fundamentals
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Best practices
- [07-GITHUB-RESOURCES.md](07-GITHUB-RESOURCES.md) - More examples
- [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md) - Applying to our project
