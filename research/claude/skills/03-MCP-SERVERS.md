# Model Context Protocol (MCP) Servers

**Status**: ✅ Validated by official documentation and 2025 ecosystem

---

## Table of Contents
1. [What is MCP?](#what-is-mcp)
2. [MCP vs Skills](#mcp-vs-skills)
3. [Essential MCP Servers for 2025](#essential-mcp-servers-for-2025)
4. [Creating Custom MCP Servers](#creating-custom-mcp-servers)
5. [Configuration](#configuration)
6. [Best Practices](#best-practices)
7. [Integration Patterns](#integration-patterns)

---

## What is MCP?

### Definition
**Model Context Protocol (MCP) is a standardized protocol developed by Anthropic for communication between AI models and external systems.**

### Architecture

```
┌─────────────┐
│ Claude Code │ ◄── MCP Client
└──────┬──────┘
       │
       │ MCP Protocol
       │
       ▼
┌─────────────┐
│ MCP Server  │ ◄── Exposes tools/resources
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ External    │
│ System/API  │
└─────────────┘
```

### What MCP Provides

**Tools**: Functions Claude can call
```typescript
// Example: GitHub MCP Server
tools:
  - create_issue
  - list_pull_requests
  - create_pr
  - get_commit
```

**Resources**: Data Claude can access
```typescript
// Example: Filesystem MCP Server
resources:
  - file://path/to/file.txt
  - directory://path/to/dir/
```

**Prompts**: Templates for common operations
```typescript
// Example: Documentation MCP Server
prompts:
  - explain_api
  - generate_docs
```

---

## MCP vs Skills

### Comparison Matrix

| Aspect | MCP Servers | Skills |
|--------|-------------|---------|
| **Complexity** | Full protocol specification | Markdown + YAML |
| **Purpose** | External system integration | Internal guidance/expertise |
| **Setup** | Requires server implementation | Just create SKILL.md |
| **Learning curve** | Steeper | Gentle |
| **Capabilities** | Tools, resources, prompts | Instructions, patterns |
| **Token cost** | Adds to system prompt | 30-50 until loaded |
| **Use case** | APIs, databases, tools | Domain knowledge, workflows |
| **2025 trend** | Mature ecosystem | Growing rapidly |

### When to Use Each

**Use MCP for:**
- ✅ External API integration (GitHub, Slack, etc.)
- ✅ Database access (Postgres, Supabase, etc.)
- ✅ Tool execution (npm, Docker, etc.)
- ✅ Resource access (filesystems, APIs, etc.)

**Use Skills for:**
- ✅ Domain expertise (Firebase patterns, testing best practices)
- ✅ Workflow guidance (TDD process, code review)
- ✅ Pattern libraries (component templates, anti-patterns)
- ✅ Learning/knowledge (project-specific conventions)

**Use Both:**
- ✅ MCP for data access + Skill for usage patterns
- ✅ MCP for API calls + Skill for best practices
- ✅ Example: Supabase MCP (query data) + Supabase-RLS Skill (security patterns)

---

## Essential MCP Servers for 2025

### 1. GitHub MCP Server 📦

**Purpose**: GitHub REST API integration

**Tools**
- `create_issue` - Create GitHub issues
- `list_pull_requests` - List PRs
- `create_pr` - Create pull requests
- `get_commit` - Fetch commit details
- `search_repositories` - Search repos
- `trigger_workflow` - CI/CD automation

**Use Cases**
- Read and manage issues
- Create PRs from Claude Code
- Analyze commits
- Trigger CI/CD pipelines

**Setup**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

### 2. Filesystem MCP Server 📁

**Purpose**: Safe file system operations

**Tools**
- `read_file` - Read file contents
- `write_file` - Write to files
- `list_directory` - List directory contents
- `search_files` - Search for files

**Benefits**
- Sandboxed access
- Permission management
- Cross-platform compatibility

### 3. Database MCP Servers 🗄️

**PostgreSQL**
```json
{
  "postgres": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": {
      "DATABASE_URL": "postgresql://..."
    }
  }
}
```

**Supabase**
- Query tables
- Check RLS policies
- Test RPC functions
- Analyze data

**Firebase**
- Query Firestore
- Check auth state
- Validate security rules
- Test cloud functions

### 4. Apidog MCP Server 🔌

**Purpose**: API documentation and testing

**Tools**
- `generate_client_code` - Type-safe API clients
- `create_mock_server` - API mocking
- `validate_response` - Response validation
- `suggest_improvements` - API design feedback

**Use Cases**
- Generate API clients from OpenAPI specs
- Mock APIs for testing
- Validate API designs
- Ensure type safety

### 5. AWS MCP Servers ☁️

**Services Covered**
- CDK/CloudFormation - Infrastructure as code
- Bedrock - AI/ML services
- Rekognition - Image analysis
- S3 - Object storage
- Lambda - Serverless functions

**Benefits**
- Full AWS integration
- Infrastructure automation
- Cloud-native development

### 6. Slack MCP Server 💬

**Tools**
- `send_message` - Post to channels
- `list_channels` - Get channel list
- `read_thread` - Read conversations
- `upload_file` - Share files

**Use Cases**
- Notifications from development
- Team communication
- Status updates
- Error reporting

---

## Creating Custom MCP Servers

### For Optimized AI Project

**Learning Database MCP Server**

```typescript
// @optimized-ai/mcp-learning
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  {
    name: "optimized-ai-learning",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Tool: Get pattern
server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "get_pattern") {
    const { type } = request.params.arguments;
    const patterns = await loadPatterns(type);
    return {
      content: [{
        type: "text",
        text: JSON.stringify(patterns, null, 2)
      }]
    };
  }

  if (request.params.name === "save_pattern") {
    const { type, data } = request.params.arguments;
    await savePattern(type, data);
    return {
      content: [{
        type: "text",
        text: "Pattern saved successfully"
      }]
    };
  }

  // More tools...
});

// List available tools
server.setRequestHandler("tools/list", async () => {
  return {
    tools: [
      {
        name: "get_pattern",
        description: "Retrieve learned patterns by type",
        inputSchema: {
          type: "object",
          properties: {
            type: {
              type: "string",
              description: "Pattern type (firebase-auth, supabase-rls, testing, etc.)"
            }
          },
          required: ["type"]
        }
      },
      {
        name: "save_pattern",
        description: "Save successful pattern for future use",
        inputSchema: {
          type: "object",
          properties: {
            type: { type: "string" },
            data: { type: "object" }
          },
          required: ["type", "data"]
        }
      },
      {
        name: "check_failure",
        description: "Check if similar failure exists",
        inputSchema: {
          type: "object",
          properties: {
            scenario: { type: "string" }
          }
        }
      },
      {
        name: "record_success",
        description: "Log successful task completion",
        inputSchema: {
          type: "object",
          properties: {
            task: { type: "string" },
            approach: { type: "string" }
          }
        }
      }
    ]
  };
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

**Configuration**
```json
{
  "mcpServers": {
    "optimized-ai-learning": {
      "command": "node",
      "args": ["/path/to/dist/index.js"],
      "env": {
        "KNOWLEDGE_DB_PATH": ".ai-knowledge/"
      }
    }
  }
}
```

---

## Configuration

### Project-Level (.mcp.json)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp-server"],
      "env": {
        "SUPABASE_URL": "${SUPABASE_URL}",
        "SUPABASE_ANON_KEY": "${SUPABASE_ANON_KEY}"
      }
    }
  }
}
```

### User-Level (~/.claude/settings.json)

```json
{
  "mcpServers": {
    "personal-tools": {
      "command": "node",
      "args": ["/home/user/.claude/mcp-servers/personal/index.js"]
    }
  }
}
```

### Environment Variables

```bash
# .env
GITHUB_TOKEN=ghp_xxx
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJxxx
```

---

## Best Practices

### 1. Don't Add Too Many MCP Servers

**Problem**: Each MCP server adds tools to system prompt
```
1 server: +500 tokens
5 servers: +2500 tokens
10 servers: +5000 tokens (significant!)
```

**Solution**: Enable only what you need
```bash
# Disable unused servers
claude mcp remove unused-server

# Or configure per-project
# Only load relevant servers in .mcp.json
```

### 2. Use /context to Monitor

```bash
# Check context consumption
/context

# Output shows:
# - MCP servers loaded
# - Tokens used by each
# - Remaining context
```

### 3. Scope to Project Needs

**Bad**: Global configuration with all servers
```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "github": {...},
    "gitlab": {...},
    "bitbucket": {...},
    "aws": {...},
    "gcp": {...},
    "azure": {...}
    // 20+ servers!
  }
}
```

**Good**: Project-specific servers
```json
// .mcp.json (Firebase project)
{
  "mcpServers": {
    "github": {...},      // For PR management
    "firebase": {...}     // For Firebase operations
  }
}
```

### 4. Security First

**Validate inputs**
```typescript
server.setRequestHandler("tools/call", async (request) => {
  const { path } = request.params.arguments;

  // ✅ Validate before use
  if (!isValidPath(path)) {
    throw new Error("Invalid path");
  }

  // ✅ Sanitize
  const safePath = sanitizePath(path);

  // Proceed...
});
```

**Use environment variables for secrets**
```json
{
  "env": {
    "API_KEY": "${API_KEY}",  // ✅ From environment
    "TOKEN": "hardcoded"      // ❌ Never do this
  }
}
```

### 5. Debug Mode

```bash
# Launch with MCP debugging
claude --mcp-debug

# Shows:
# - MCP server connections
# - Tool calls
# - Errors
```

### 6. Regular Cleanup

```bash
# List enabled servers
claude mcp list

# Remove unused
claude mcp remove old-server

# Update servers
npm update -g @modelcontextprotocol/server-*
```

---

## Integration Patterns

### Pattern 1: MCP + Skill Combination

**Example: Supabase**

**MCP Server**: Data access
```typescript
// Supabase MCP provides:
- query_table(table, query)
- insert_row(table, data)
- update_row(table, id, data)
```

**Skill**: Best practices
```yaml
---
name: supabase-rls
description: Supabase Row Level Security best practices
---

# When using Supabase MCP tools:
1. Always check RLS policies first
2. Test with different user roles
3. Validate data structure
4. Use prepared statements
```

**Workflow**
```
User: "Add user data to Supabase"
Claude: [loads supabase-rls skill]
Claude: [uses Supabase MCP to query current schema]
Claude: [applies RLS best practices from skill]
Claude: [uses MCP to insert with proper policies]
```

### Pattern 2: Multi-MCP Workflow

```
Feature: "Create GitHub issue from failed test"

Step 1: Use Testing MCP
  → Run tests
  → Detect failures

Step 2: Use GitHub MCP
  → Create issue with failure details
  → Link to commit
  → Assign to team member

Step 3: Use Slack MCP
  → Notify team
  → Include issue link
```

### Pattern 3: MCP for Learning Database

**Custom MCP for Optimized AI**

```typescript
// Tools provided:
- get_pattern(type)           // Retrieve learned patterns
- save_pattern(type, data)    // Save new patterns
- check_failure(scenario)     // Check historical failures
- record_success(task)        // Log successes
- get_preferences()           // User preferences
- update_metrics(data)        // Performance tracking
```

**Integration with Skills**
```yaml
---
name: firebase-auth
---

# Before implementing:
1. Check learned patterns: get_pattern('firebase-auth')
2. Check for past failures: check_failure('firebase-auth-setup')
3. Apply known-good patterns

# After success:
1. Save pattern: save_pattern('firebase-auth', implementation)
2. Record success: record_success('firebase-auth-setup')
```

---

## Key Takeaways for Optimized AI

### 1. Use MCP for External Integration
- GitHub (PR management)
- Firebase/Supabase (data operations)
- Custom learning database

### 2. Combine with Skills
- MCP: Data access
- Skills: Best practices
- Together: Powerful combination

### 3. Build Custom MCP for Learning
- Store/retrieve patterns
- Track successes/failures
- Manage preferences
- Record metrics

### 4. Monitor Token Impact
- Use /context to check usage
- Disable unused servers
- Scope to project needs

### 5. Implement Gradually
- Phase 0: No MCP (baseline)
- Phase 1: GitHub MCP only
- Phase 2: Add learning MCP
- Phase 3: Firebase/Supabase MCPs

---

## Resources

### Official MCP
- [MCP Specification](https://modelcontextprotocol.io/)
- [MCP SDK](https://github.com/modelcontextprotocol/sdk)
- [Official Servers](https://github.com/modelcontextprotocol/)

### Server Collections
- [MCPcat.io](https://mcpcat.io/) - MCP server marketplace
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)

### Guides
- [Creating MCP Servers](https://modelcontextprotocol.io/docs/creating-servers)
- [Claude Code MCP Setup](https://docs.claude.com/en/docs/claude-code/mcp)

---

**Next**: Read [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md) for token optimization strategies
