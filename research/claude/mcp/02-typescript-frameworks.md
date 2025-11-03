# TypeScript MCP Frameworks Comparison

## Overview

This document compares TypeScript frameworks for building MCP servers, focusing on those with decorator patterns for ease of use (matching the project requirement: "just write the function and wrap it with a decorator to expose it").

## 🏆 Framework Rankings

### 1. @nestjs-mcp/server ⭐ RECOMMENDED

**Best for**: Production applications, enterprise use cases, teams familiar with NestJS

#### Installation
```bash
npm install @nestjs-mcp/server @modelcontextprotocol/sdk zod
```

#### Key Features
- ✅ **Clean decorator pattern** - `@Tool`, `@Prompt`, `@Resource`
- ✅ **NestJS integration** - Dependency injection, modules, guards
- ✅ **TypeScript-first** - Full type safety
- ✅ **Production-ready** - Guards, interceptors, middleware
- ✅ **Compatible with official SDK** - Always kept up-to-date

#### Decorator Examples

**Tool Definition:**
```typescript
import { Tool, Resolver } from '@nestjs-mcp/server';
import { z } from 'zod';

@Resolver('user_tools')
export class UserToolsResolver {
  @Tool({
    name: 'delete_user',
    description: 'Deletes a user by ID',
    paramsSchema: { userId: z.string() },
    annotations: {
      destructiveHint: true,
      readOnlyHint: false
    },
  })
  deleteUser({ userId }: { userId: string }, extra: RequestHandlerExtra) {
    // Implementation
    return {
      content: [{ type: 'text', text: `User ${userId} deleted.` }]
    };
  }
}
```

**Resource Definition:**
```typescript
import { Resource, Resolver } from '@nestjs-mcp/server';

@Resolver('data_resources')
export class DataResourcesResolver {
  @Resource({
    uri: 'data://users/{userId}',
    name: 'user_data',
    description: 'Get user data by ID',
  })
  async getUserData({ userId }: { userId: string }) {
    const user = await this.userService.findById(userId);
    return {
      uri: `data://users/${userId}`,
      text: JSON.stringify(user),
      mimeType: 'application/json',
    };
  }
}
```

**Prompt Definition:**
```typescript
import { Prompt, Resolver } from '@nestjs-mcp/server';

@Resolver('prompts')
export class PromptResolver {
  @Prompt({
    name: 'code_review',
    description: 'Generate code review prompt',
    arguments: [
      { name: 'language', description: 'Programming language', required: true },
      { name: 'style', description: 'Review style', required: false },
    ],
  })
  generateCodeReview({ language, style }: { language: string; style?: string }) {
    return {
      messages: [
        {
          role: 'user',
          content: {
            type: 'text',
            text: `Review this ${language} code with ${style || 'standard'} style.`,
          },
        },
      ],
    };
  }
}
```

**Guards for Access Control:**
```typescript
import { UseGuards, Resolver, Tool } from '@nestjs-mcp/server';

@Resolver('protected')
export class ProtectedResolver {
  @UseGuards(AuthGuard)
  @Tool({ name: 'admin_action', description: 'Admin-only action' })
  adminAction() {
    // Only executes if AuthGuard passes
  }
}
```

#### Pros
- 🟢 Perfect decorator pattern - exactly what you asked for
- 🟢 NestJS ecosystem (DI, testing, modules)
- 🟢 Built-in guards and middleware
- 🟢 Excellent for complex applications
- 🟢 Great TypeScript support
- 🟢 Official SDK wrapper - always compatible

#### Cons
- 🔴 Requires NestJS knowledge
- 🔴 More boilerplate than FastMCP
- 🔴 Heavier dependency footprint

---

### 2. @bamada/nestjs-mcp (Alternative NestJS Implementation)

**Best for**: Teams wanting NestJS without the @nestjs-mcp/server package

#### Installation
```bash
npm install @bamada/nestjs-mcp @modelcontextprotocol/sdk
```

#### Example
```typescript
import { Injectable } from '@nestjs/common';
import { McpTool, McpPrompt, RequestHandlerExtra } from '@bamada/nestjs-mcp';
import { z } from 'zod';

@Injectable()
export class MyMcpProvider {
  @McpTool({
    name: 'add_numbers',
    description: 'Adds two numbers',
    paramsSchema: {
      a: z.number(),
      b: z.number(),
    },
  })
  addNumbers({ a, b }: { a: number; b: number }) {
    return { result: a + b };
  }

  @McpPrompt({
    name: 'greeting',
    description: 'Generates a greeting',
    arguments: [
      { name: 'userName', description: 'Name', required: true },
    ],
  })
  generateGreeting({ userName }: { userName: string }) {
    return {
      messages: [
        { role: 'user', content: { type: 'text', text: `Hello ${userName}!` } },
      ],
    };
  }
}
```

#### Pros
- 🟢 Clean decorator syntax
- 🟢 NestJS integration
- 🟢 Good documentation

#### Cons
- 🔴 Less maintained than @nestjs-mcp/server
- 🔴 Smaller community

---

### 3. FastMCP (TypeScript)

**Best for**: Quick prototypes, simple servers, developers who want minimal setup

#### Installation
```bash
npm install fastmcp
```

#### Example
```typescript
import { FastMCP } from 'fastmcp';

const mcp = new FastMCP('My Server');

// Tools
mcp.tool('calculate', {
  description: 'Calculate sum',
  parameters: { a: 'number', b: 'number' },
}, async ({ a, b }) => {
  return { result: a + b };
});

// Resources
mcp.resource('data://config', {
  description: 'App configuration',
}, async () => {
  return { config: { /* ... */ } };
});

// Prompts
mcp.prompt('generate', {
  description: 'Generate prompt',
  arguments: { topic: 'string' },
}, async ({ topic }) => {
  return `Generate content about ${topic}`;
});
```

#### Pros
- 🟢 Minimal setup
- 🟢 Express-like API
- 🟢 Quick to get started
- 🟢 Low dependency footprint

#### Cons
- 🔴 No decorator pattern (functional instead)
- 🔴 Less structured than NestJS options
- 🔴 TypeScript version less popular than Python version

---

### 4. @rekog/mcp-nest

**Best for**: Teams wanting Context injection and progress reporting

#### Installation
```bash
npm install @rekog/mcp-nest
```

#### Example with Context
```typescript
import { Injectable } from '@nestjs/common';
import { Tool, Context } from '@rekog/mcp-nest';
import { z } from 'zod';

@Injectable()
export class GreetingTool {
  @Tool({
    name: 'greeting-tool',
    description: 'Returns a greeting with progress',
    parameters: z.object({
      name: z.string().default('World'),
    }),
  })
  async sayHello({ name }, context: Context) {
    // Report progress
    await context.reportProgress({ progress: 50, total: 100 });

    // Sample from LLM if needed
    const response = await context.sample({
      messages: [{ role: 'user', content: 'Generate a greeting' }],
    });

    return `Hello, ${name}!`;
  }
}
```

#### Pros
- 🟢 Context injection for advanced features
- 🟢 Progress reporting
- 🟢 LLM sampling support
- 🟢 Clean decorator pattern

#### Cons
- 🔴 Newer/less proven
- 🔴 Smaller community

---

### 5. Official SDK (@modelcontextprotocol/sdk)

**Best for**: Maximum control, custom architectures, no framework dependencies

#### Installation
```bash
npm install @modelcontextprotocol/sdk zod
```

#### Example
```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { z } from 'zod';

const server = new McpServer({
  name: 'my-server',
  version: '1.0.0',
});

// Register tool
server.tool(
  'add',
  {
    description: 'Add two numbers',
    a: z.number(),
    b: z.number(),
  },
  async ({ a, b }) => {
    return {
      content: [
        { type: 'text', text: JSON.stringify({ result: a + b }) }
      ],
    };
  }
);

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

#### Pros
- 🟢 Official implementation
- 🟢 Most flexible
- 🟢 Direct protocol access
- 🟢 Well-documented
- 🟢 Zod schema validation

#### Cons
- 🔴 No decorator pattern
- 🔴 More verbose
- 🔴 Manual server lifecycle management
- 🔴 More boilerplate

---

## 🎯 Recommendation Matrix

| Use Case | Recommended Framework | Reason |
|----------|----------------------|---------|
| **Production app with complexity** | `@nestjs-mcp/server` | Best architecture, DI, guards |
| **Simple server, quick start** | `FastMCP` | Minimal setup |
| **Need maximum control** | Official SDK | Direct protocol access |
| **Need Context features** | `@rekog/mcp-nest` | Progress reporting, LLM sampling |
| **Team knows NestJS** | `@nestjs-mcp/server` | Familiar patterns |
| **Prototype/MVP** | `FastMCP` or Official SDK | Quick iteration |

## 🏗️ For Optimized AI Project

**Recommendation: @nestjs-mcp/server**

Reasons:
1. ✅ **Matches your requirement**: "Just write function and wrap with decorator"
2. ✅ **Separation of Concerns**: Aligns with your SEPARATE principle (resolvers, modules)
3. ✅ **Minimize**: Clean, structured code with guards and DI reduces boilerplate
4. ✅ **Production-ready**: Built-in patterns for scaling
5. ✅ **TypeScript-first**: Full type safety
6. ✅ **Ecosystem**: Can leverage NestJS plugins and patterns

### Example for Your Project

```typescript
// @optimized-ai/mcp-core server structure

// src/resolvers/patterns.resolver.ts
@Resolver('patterns')
export class PatternsResolver {
  constructor(private patternsService: PatternsService) {}

  @Tool({
    name: 'get_pattern',
    description: 'Retrieve learned pattern',
    paramsSchema: { type: z.string() },
  })
  getPattern({ type }: { type: string }) {
    return this.patternsService.getPattern(type);
  }

  @Tool({
    name: 'save_pattern',
    description: 'Save new pattern',
    paramsSchema: {
      type: z.string(),
      data: z.object({}).passthrough(),
    },
  })
  savePattern({ type, data }: { type: string; data: any }) {
    return this.patternsService.savePattern(type, data);
  }
}

// src/resolvers/firebase.resolver.ts (skill-specific)
@Resolver('firebase')
export class FirebaseResolver {
  @Tool({
    name: 'query_firestore',
    description: 'Query Firestore collection',
    paramsSchema: {
      collection: z.string(),
      query: z.object({}).passthrough(),
    },
  })
  async queryFirestore({ collection, query }: { collection: string; query: any }) {
    // Implementation
  }
}
```

## 📊 Feature Comparison Table

| Feature | @nestjs-mcp/server | @bamada/nestjs-mcp | @rekog/mcp-nest | FastMCP | Official SDK |
|---------|-------------------|-------------------|----------------|---------|--------------|
| Decorator Pattern | ✅ Excellent | ✅ Good | ✅ Good | ❌ Functional | ❌ Imperative |
| Type Safety | ✅✅ | ✅ | ✅ | ✅ | ✅✅ |
| Zod Integration | ✅ | ✅ | ✅ | ⚠️ Limited | ✅ |
| Dependency Injection | ✅ NestJS | ✅ NestJS | ✅ NestJS | ❌ | ❌ |
| Guards/Middleware | ✅ | ✅ | ✅ | ❌ | Manual |
| Context Access | ✅ | ✅ | ✅✅ Advanced | ⚠️ Basic | ✅ |
| Learning Curve | Medium | Medium | Medium | Low | Low-Medium |
| Boilerplate | Medium | Medium | Medium | Low | Medium-High |
| Community Size | Growing | Small | Small | Medium | Large |
| Maintenance | Active | Less Active | Active | Active | Official |
| Documentation | Good | Good | Good | Good | Excellent |

## 🚀 Quick Start Comparison

### @nestjs-mcp/server
```bash
npm install @nestjs/core @nestjs/common @nestjs-mcp/server
# Create module, resolver, register in app
# ~50 lines for basic setup
```

### FastMCP
```bash
npm install fastmcp
# Create server, register tools
# ~10 lines for basic setup
```

### Official SDK
```bash
npm install @modelcontextprotocol/sdk zod
# Create server, register handlers, connect transport
# ~30 lines for basic setup
```

## 📝 Code Organization Patterns

### NestJS Pattern (Recommended for Optimized AI)
```
src/
├── app.module.ts
├── resolvers/
│   ├── core.resolver.ts       # Core MCP operations
│   ├── patterns.resolver.ts   # Pattern management
│   ├── firebase.resolver.ts   # Firebase skill (loaded on-demand)
│   └── supabase.resolver.ts   # Supabase skill (loaded on-demand)
├── services/
│   ├── patterns.service.ts
│   ├── knowledge.service.ts
│   └── storage.service.ts
└── main.ts
```

### FastMCP Pattern
```
src/
├── server.ts                   # Single file or
├── tools/
│   ├── patterns.ts
│   ├── firebase.ts
│   └── supabase.ts
└── index.ts
```

## 🎓 Learning Resources

### @nestjs-mcp/server
- npm: https://www.npmjs.com/package/@nestjs-mcp/server
- GitHub: https://github.com/bamada/nestjs-mcp
- Examples: Check test files in repository

### FastMCP (TypeScript)
- GitHub: https://github.com/punkpeye/fastmcp
- Docs: Check README and examples

### Official SDK
- GitHub: https://github.com/modelcontextprotocol/typescript-sdk
- Docs: https://modelcontextprotocol.io/
- Examples: https://github.com/modelcontextprotocol/servers
