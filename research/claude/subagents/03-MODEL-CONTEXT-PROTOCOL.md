# Model Context Protocol (MCP)

**Last Updated**: November 3, 2025

---

## Table of Contents

1. [What is MCP?](#what-is-mcp)
2. [Why MCP Matters](#why-mcp-matters)
3. [Architecture](#architecture)
4. [Core Primitives](#core-primitives)
5. [Creating MCP Servers](#creating-mcp-servers)
6. [Pre-Built MCP Servers](#pre-built-mcp-servers)
7. [Integration with Claude](#integration-with-claude)
8. [MCP for Skills Architecture](#mcp-for-skills-architecture)
9. [Best Practices](#best-practices)
10. [Implementation Examples](#implementation-examples)

---

## What is MCP?

### Official Definition

> **Model Context Protocol (MCP) is an open standard that enables developers to build secure connections between AI assistants and data sources.**

### Key Insight

MCP solves the **data integration fragmentation problem**:

```
Before MCP:
AI Assistant → Custom Integration 1 → Data Source 1
AI Assistant → Custom Integration 2 → Data Source 2
AI Assistant → Custom Integration 3 → Data Source 3
...
(N data sources = N custom integrations = N × effort)

With MCP:
AI Assistant → MCP Client → MCP Protocol
                              ↓
                         MCP Server 1 → Data Source 1
                         MCP Server 2 → Data Source 2
                         MCP Server 3 → Data Source 3
...
(N data sources = 1 client + N standardized servers)
```

### Analogy

> **"Think of MCP like a USB-C port for AI applications."**

Just as USB-C standardizes physical device connections, MCP standardizes AI-to-data connections.

---

## Why MCP Matters

### The Problem

Modern AI systems face **data isolation**:

1. Each data source requires custom integration
2. No reusable patterns across projects
3. Duplication of effort
4. Difficult to maintain
5. Security and auth complexity

### The Solution

**MCP provides**:

1. **Standardized protocol** - One way to connect to all data sources
2. **Reusable servers** - Write once, use everywhere
3. **Security built-in** - Standard auth patterns
4. **Open source** - Community-contributed servers
5. **Platform support** - Native integration in Claude

### Impact on Optimized AI

For our project, MCP enables:

✅ **Standardized skill servers** - Each skill can be an MCP server
✅ **IDE integration** - MCP server exposes IDE operations
✅ **Knowledge base access** - MCP server for `.ai-knowledge/`
✅ **Tool standardization** - Consistent tool interface
✅ **Reusability** - Skills work across projects via MCP

---

## Architecture

### High-Level Design

```
┌─────────────────────┐
│   Claude Agent      │
│   (MCP Client)      │
└──────────┬──────────┘
           │
           │ MCP Protocol
           │
           ├─────────────────────┐
           │                     │
           ▼                     ▼
┌──────────────────┐  ┌──────────────────┐
│   MCP Server 1   │  │   MCP Server 2   │
│   (Firebase)     │  │   (Supabase)     │
└────────┬─────────┘  └────────┬─────────┘
         │                     │
         ▼                     ▼
    Firebase API          Supabase API
```

### Components

#### 1. MCP Client

- **What**: AI agent that consumes MCP servers
- **Who**: Claude Code, Claude API, custom agents
- **Responsibilities**: Discover servers, invoke tools, fetch resources

#### 2. MCP Server

- **What**: Service that exposes data/functionality via MCP
- **Who**: Firebase server, Slack server, custom skill servers
- **Responsibilities**: Implement MCP protocol, provide tools/resources/prompts

#### 3. MCP Protocol

- **What**: Standardized communication format
- **How**: JSON-RPC over stdio/HTTP
- **Defines**: Tool invocation, resource fetching, prompt templates

---

## Core Primitives

MCP defines **three core primitives**:

### 1. Tools

**Purpose**: Actions the AI can perform

**Example**:

```typescript
// MCP Server exposes tool
server.tool({
  name: 'query_firestore',
  description: 'Query Firestore collection',
  parameters: {
    collection: { type: 'string', description: 'Collection name' },
    query: { type: 'object', description: 'Query conditions' }
  },
  handler: async (params) => {
    const results = await firestore
      .collection(params.collection)
      .where(/* params.query */)
      .get();
    return results.docs.map(doc => doc.data());
  }
});
```

**Usage in Claude**:

```
User: "Show me all users with email @example.com"
Claude: [Invokes query_firestore tool]
        {
          "collection": "users",
          "query": { "email": { "endsWith": "@example.com" } }
        }
Server: [Returns results]
Claude: "I found 42 users..."
```

### 2. Resources

**Purpose**: Data the AI can read

**Example**:

```typescript
// MCP Server exposes resource
server.resource({
  uri: 'firebase://users/{userId}',
  description: 'User profile data',
  mimeType: 'application/json',
  handler: async (uri) => {
    const userId = extractUserId(uri);
    const user = await firestore
      .collection('users')
      .doc(userId)
      .get();
    return user.data();
  }
});
```

**Usage in Claude**:

```
User: "What's the email for user abc123?"
Claude: [Fetches resource firebase://users/abc123]
Server: { "email": "user@example.com", "name": "John" }
Claude: "The email is user@example.com"
```

### 3. Prompts

**Purpose**: Prompt templates the AI can use

**Example**:

```typescript
// MCP Server exposes prompt template
server.prompt({
  name: 'review_security',
  description: 'Security review prompt for Firebase code',
  template: `Review the following Firebase code for security issues:

{code}

Focus on:
- Firebase Security Rules
- Authentication checks
- Data validation
- Access control

Provide specific findings with severity levels.`,
  parameters: {
    code: { type: 'string', description: 'Code to review' }
  }
});
```

**Usage in Claude**:

```
User: "Review auth.ts for security issues"
Claude: [Uses review_security prompt template]
        Fills in {code} with contents of auth.ts
        Performs security analysis as defined in template
```

---

## Creating MCP Servers

### Quick Start (TypeScript)

#### 1. Install SDK

```bash
npm install @modelcontextprotocol/sdk
```

#### 2. Create Server

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

// Create server instance
const server = new Server(
  {
    name: 'optimized-ai-firebase',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {},
    },
  }
);

// Define a tool
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'query_firestore',
      description: 'Query Firestore collection',
      inputSchema: {
        type: 'object',
        properties: {
          collection: {
            type: 'string',
            description: 'Collection name'
          },
          where: {
            type: 'array',
            description: 'Query conditions'
          }
        },
        required: ['collection']
      }
    }
  ]
}));

// Handle tool invocation
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  if (name === 'query_firestore') {
    // Implement Firestore query
    const results = await queryFirestore(args.collection, args.where);
    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(results, null, 2)
        }
      ]
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

#### 3. Configure in Claude

Add to `.claude/mcp.json`:

```json
{
  "mcpServers": {
    "optimized-ai-firebase": {
      "command": "node",
      "args": ["./mcp-servers/firebase/dist/index.js"]
    }
  }
}
```

### Server Template

Use official template:

```bash
npx @modelcontextprotocol/create-server my-server
cd my-server
npm install
npm run build
```

---

## Pre-Built MCP Servers

### Official Anthropic Servers

Anthropic provides ready-made servers for:

1. **Google Drive** - Access Drive files
2. **Slack** - Send messages, read channels
3. **GitHub** - Repo operations, PRs, issues
4. **Git** - Local Git operations
5. **Postgres** - Database queries
6. **Puppeteer** - Browser automation

### Installation Example

```bash
# Install GitHub MCP server
npm install @modelcontextprotocol/server-github

# Configure in Claude
# .claude/mcp.json
{
  "mcpServers": {
    "github": {
      "command": "node",
      "args": ["./node_modules/@modelcontextprotocol/server-github/dist/index.js"],
      "env": {
        "GITHUB_TOKEN": "your_token_here"
      }
    }
  }
}
```

### Community Servers

Browse at [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)

---

## Integration with Claude

### Claude Desktop

**Automatic MCP support**:

1. Install MCP server (npm package or custom)
2. Configure in `~/.claude/mcp.json`
3. Restart Claude Desktop
4. Tools/resources automatically available

### Claude Code

**Same as Desktop**:

```json
// .claude/mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./path/to/server.js"],
      "env": {
        "API_KEY": "..."
      }
    }
  }
}
```

### Claude Agent SDK

**Programmatic integration**:

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const result = await query({
  prompt: "Query users from Firestore",
  options: {
    mcp_servers: ['optimized-ai-firebase']
  }
});
```

---

## MCP for Skills Architecture

### Our Use Case

**Skills as MCP Servers**:

```
Concept: Each skill = MCP server

Benefits:
✅ Standardized interface
✅ Tools + Resources + Prompts in one package
✅ Easy to install/enable/disable
✅ Version controlled
✅ Shareable across projects
```

### Example: Firebase Skill as MCP Server

```typescript
// @optimized-ai/skill-firebase

// Tools
- queryFirestore(collection, query)
- updateDocument(collection, id, data)
- validateSecurityRules(rulesFile)

// Resources
- firebase://patterns/auth
- firebase://patterns/firestore
- firebase://examples/queries

// Prompts
- review_firebase_security(code)
- implement_firebase_auth(requirements)
```

### Installation

```bash
npm install @optimized-ai/skill-firebase

# Auto-configures in .claude/mcp.json
```

### Usage

```
User: "Implement Firebase login"

Claude:
1. Detects "Firebase" keyword
2. Finds @optimized-ai/skill-firebase MCP server
3. Fetches resource: firebase://patterns/auth
4. Uses implement_firebase_auth prompt
5. Invokes tools as needed
6. Implements login feature
```

---

## Best Practices

### 1. Design for Discoverability

**Good tool descriptions**:

```typescript
// ❌ BAD
{
  name: 'query',
  description: 'Queries data'
}

// ✅ GOOD
{
  name: 'query_firestore_collection',
  description: 'Queries a Firestore collection with optional where clauses, ordering, and limits. Returns array of documents matching criteria.'
}
```

### 2. Minimize Tool Overlap

**Avoid similar tools**:

```typescript
// ❌ BAD: Too many similar tools
- getUserById(id)
- getUserByEmail(email)
- getUserByUsername(username)
- getUserByPhone(phone)

// ✅ GOOD: One flexible tool
- getUser({ id?, email?, username?, phone? })
```

### 3. Return Token-Efficient Data

**Summarize, don't dump**:

```typescript
// ❌ BAD: Returns everything
async function getUsers() {
  const users = await firestore.collection('users').get();
  return users.docs.map(doc => doc.data());
  // Returns 50k tokens of user data!
}

// ✅ GOOD: Returns summary
async function getUsers({ limit = 10, fields = ['id', 'email'] }) {
  const users = await firestore
    .collection('users')
    .limit(limit)
    .get();
  return users.docs.map(doc => {
    const data = doc.data();
    return fields.reduce((obj, field) => {
      obj[field] = data[field];
      return obj;
    }, {});
  });
  // Returns ~1k tokens of relevant data
}
```

### 4. Use Resources for Static Content

**Tools for actions, Resources for data**:

```typescript
// ✅ Resources for documentation
server.resource({
  uri: 'firebase://docs/auth',
  handler: () => fibaseAuthDocs
});

// ✅ Tools for operations
server.tool({
  name: 'create_user',
  handler: async (params) => {
    return await createUser(params);
  }
});
```

### 5. Implement Error Handling

**Return useful errors**:

```typescript
server.setRequestHandler('tools/call', async (request) => {
  try {
    // Perform operation
    return { success: true, data: result };
  } catch (error) {
    // Return structured error
    return {
      isError: true,
      content: [
        {
          type: 'text',
          text: `Error: ${error.message}\n\nSuggestions:\n- Check credentials\n- Verify collection exists\n- Ensure user has permissions`
        }
      ]
    };
  }
});
```

### 6. Security First

**Validate all inputs**:

```typescript
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  // Validate tool name
  if (!ALLOWED_TOOLS.includes(name)) {
    throw new Error(`Tool not allowed: ${name}`);
  }

  // Validate arguments
  if (name === 'query_firestore') {
    if (!args.collection || typeof args.collection !== 'string') {
      throw new Error('Invalid collection parameter');
    }
    // Prevent injection
    if (args.collection.includes('/') || args.collection.includes('..')) {
      throw new Error('Invalid collection name');
    }
  }

  // Proceed with validated inputs
});
```

---

## Implementation Examples

### Example 1: Simple Knowledge Base MCP Server

```typescript
// @optimized-ai/mcp-knowledge-base

import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { readFileSync } from 'fs';
import { join } from 'path';

const server = new Server({
  name: 'optimized-ai-knowledge',
  version: '1.0.0'
});

// List available patterns
server.setRequestHandler('resources/list', async () => ({
  resources: [
    {
      uri: 'knowledge://patterns/firebase-auth',
      name: 'Firebase Authentication Pattern',
      mimeType: 'text/markdown'
    },
    {
      uri: 'knowledge://patterns/testing',
      name: 'Testing Pattern',
      mimeType: 'text/markdown'
    }
  ]
}));

// Fetch specific pattern
server.setRequestHandler('resources/read', async (request) => {
  const { uri } = request.params;

  if (uri.startsWith('knowledge://patterns/')) {
    const patternName = uri.replace('knowledge://patterns/', '');
    const filePath = join('.ai-knowledge/patterns', `${patternName}.json`);

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
  }

  throw new Error(`Unknown resource: ${uri}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Example 2: IDE Operations MCP Server

```typescript
// @optimized-ai/mcp-ide

import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import * as vscode from 'vscode';

const server = new Server({
  name: 'optimized-ai-ide',
  version: '1.0.0'
});

// Define IDE operation tools
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'rename_symbol',
      description: 'Rename a symbol across the codebase using IDE refactoring',
      inputSchema: {
        type: 'object',
        properties: {
          oldName: { type: 'string' },
          newName: { type: 'string' }
        },
        required: ['oldName', 'newName']
      }
    },
    {
      name: 'format_document',
      description: 'Format document using project formatting rules',
      inputSchema: {
        type: 'object',
        properties: {
          filePath: { type: 'string' }
        },
        required: ['filePath']
      }
    },
    {
      name: 'run_tests',
      description: 'Run test suite',
      inputSchema: {
        type: 'object',
        properties: {
          pattern: { type: 'string', description: 'Test pattern to match' }
        }
      }
    }
  ]
}));

// Handle tool invocations
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'rename_symbol':
      await vscode.commands.executeCommand('editor.action.rename', {
        oldName: args.oldName,
        newName: args.newName
      });
      return {
        content: [{ type: 'text', text: `Renamed ${args.oldName} to ${args.newName}` }]
      };

    case 'format_document':
      const doc = await vscode.workspace.openTextDocument(args.filePath);
      await vscode.commands.executeCommand('editor.action.formatDocument', doc.uri);
      return {
        content: [{ type: 'text', text: `Formatted ${args.filePath}` }]
      };

    case 'run_tests':
      const results = await runTests(args.pattern);
      return {
        content: [{ type: 'text', text: JSON.stringify(results, null, 2) }]
      };

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});
```

---

## Key Takeaways

1. **MCP standardizes AI-to-data connections** (like USB-C for AI)

2. **Three core primitives**: Tools (actions), Resources (data), Prompts (templates)

3. **Skills can be MCP servers** - natural fit for our architecture

4. **Pre-built servers available** for common services (GitHub, Slack, etc.)

5. **Token efficiency matters** - Return summaries, not dumps

6. **Security is critical** - Validate inputs, limit access

7. **Discovery is key** - Good descriptions help Claude choose right tools

8. **Minimize overlap** - One flexible tool > many specific tools

9. **Native Claude support** - Works in Desktop, Code, and SDK

10. **Community-driven** - Growing ecosystem of servers

---

## Further Reading

- [Official MCP Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)
- [MCP GitHub](https://github.com/modelcontextprotocol)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive guidelines
- [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md) - More code samples
