# MCP Hub Architecture

## Overview

Roo Code implements a comprehensive MCP (Model Context Protocol) integration through the `McpHub` class that manages multiple MCP servers with different transport types, file watching, and auto-restart capabilities.

## The Problem It Solves

**Challenge**: Connect AI to external tools and resources (databases, APIs, file systems, etc.) in a standardized way.

**Solutions before MCP**:
- Custom integrations per tool (messy, non-standard)
- Direct API calls from AI (security risks, hard to maintain)
- Tool-specific SDKs (context pollution, bloated)

**MCP Solution**: Standardized protocol for connecting AI to tools via servers.

## Roo Code's MCP Implementation

### Supported Transport Types

From `src/services/mcp/McpHub.ts`:

```typescript
export type McpConnection = ConnectedMcpConnection | DisconnectedMcpConnection

export type ConnectedMcpConnection = {
  type: "connected"
  server: McpServer
  client: Client
  transport: StdioClientTransport | SSEClientTransport | StreamableHTTPClientTransport
}
```

**Three transport types**:

1. **stdio** - Standard input/output (local processes)
   ```json
   {
     "type": "stdio",
     "command": "npx",
     "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"],
     "env": { "CUSTOM_VAR": "value" }
   }
   ```

2. **sse** - Server-Sent Events (HTTP streaming)
   ```json
   {
     "type": "sse",
     "url": "http://localhost:3000/sse",
     "headers": { "Authorization": "Bearer token" }
   }
   ```

3. **streamable-http** - Streamable HTTP transport
   ```json
   {
     "type": "streamable-http",
     "url": "http://localhost:3000/mcp",
     "headers": { "X-API-Key": "key" }
   }
   ```

### Configuration Schema

```typescript
const BaseConfigSchema = z.object({
  disabled: z.boolean().optional(),
  timeout: z.number().min(1).max(3600).optional().default(60),
  alwaysAllow: z.array(z.string()).default([]),
  watchPaths: z.array(z.string()).optional(),  // ⭐ File watching
  disabledTools: z.array(z.string()).default([]),
})

// Example server config
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"],
      "timeout": 30,
      "watchPaths": [".env", "config.json"],  // Auto-restart on changes
      "alwaysAllow": ["read_file"],           // Pre-approved tools
      "disabledTools": ["delete_file"]        // Disabled tools
    }
  }
}
```

### Key Features

#### 1. File Watching for Auto-Restart

**Problem**: MCP servers cache configuration. Changes to `.env` or config files don't apply until restart.

**Solution**: Watch specified files and auto-restart server when they change.

```typescript
private fileWatchers: Map<string, FSWatcher[]> = new Map()

// Setup file watchers for server
if (serverConfig.watchPaths) {
  const watchers = serverConfig.watchPaths.map(watchPath => {
    const watcher = chokidar.watch(watchPath, {
      ignoreInitial: true
    })

    watcher.on('change', async (path) => {
      console.log(`[McpHub] File ${path} changed, restarting server...`)
      await this.restartServer(serverName)
    })

    return watcher
  })

  this.fileWatchers.set(serverName, watchers)
}
```

**Use case**:
- `.env` file updated with new API key → Server auto-restarts with new config
- `database.json` connection string changed → Reconnect to new database

#### 2. Tool Permission Management

**Pre-approved tools** (`alwaysAllow`):
```json
{
  "alwaysAllow": ["read_file", "list_files"]
}
```
- These tools execute without user approval
- Useful for read-only operations

**Disabled tools** (`disabledTools`):
```json
{
  "disabledTools": ["delete_file", "execute_command"]
}
```
- These tools are hidden from AI
- Safety mechanism to prevent dangerous operations

#### 3. Connection Management

**Connection states**:
```typescript
export type McpConnection =
  | { type: "connected", server, client, transport }
  | { type: "disconnected", server, client: null, transport: null }

// Disable reasons
export enum DisableReason {
  MCP_DISABLED = "mcpDisabled",        // Global MCP disabled
  SERVER_DISABLED = "serverDisabled",  // Specific server disabled
}
```

**Lifecycle**:
1. Load config from `~/.roo/mcp.json` (global) or `.roo/mcp.json` (project)
2. Validate config against schema
3. Create transport (stdio/SSE/streamable-http)
4. Connect client to server
5. List available tools and resources
6. Setup file watchers (if watchPaths specified)
7. Monitor for config changes
8. Auto-restart on file changes or config updates

#### 4. Multi-Level Configuration

**Global config**: `~/.roo/mcp.json`
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "~"]
    }
  }
}
```

**Project config**: `.roo/mcp.json` (overrides global)
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./workspace"],
      "watchPaths": ["./workspace/.env"]
    }
  }
}
```

**Merge strategy**: Project config overrides global for same server name.

### Tool and Resource Access

**List available tools**:
```typescript
const tools = await client.listTools()
// Returns: [{ name: "read_file", description: "...", inputSchema: {...} }]
```

**Call a tool**:
```typescript
const result = await client.callTool({
  name: "read_file",
  arguments: { path: "/workspace/file.txt" }
})
```

**List available resources**:
```typescript
const resources = await client.listResources()
// Returns: [{ uri: "file:///workspace/docs", name: "Documentation" }]
```

**Read a resource**:
```typescript
const resource = await client.readResource({
  uri: "file:///workspace/docs/api.md"
})
```

### Error Handling

**Connection errors**:
```typescript
try {
  await transport.connect()
} catch (error) {
  console.error(`Failed to connect to ${serverName}:`, error)
  // Mark as disconnected, keep trying
  return { type: "disconnected", server, client: null, transport: null }
}
```

**Tool execution errors**:
```typescript
try {
  const result = await client.callTool(request)
  return result.content
} catch (error) {
  // Show error to user, log to MCP Errors tab
  await this.logError(serverName, error)
  throw error
}
```

**Timeout handling**:
```typescript
const timeout = serverConfig.timeout || 60  // seconds

const resultPromise = client.callTool(request)
const timeoutPromise = delay(timeout * 1000).then(() => {
  throw new Error(`Tool ${tool} timed out after ${timeout}s`)
})

const result = await Promise.race([resultPromise, timeoutPromise])
```

## Comparison to Our Design

### Our Decision (ADR-003)

> **Prefer direct CLI scripts over MCP middleware.** Only use MCP when actually necessary.

**Reasoning**:
- Firebase CLI, Supabase CLI already exist
- No need to wrap in MCP servers
- Simpler, more maintainable
- Faster development

### When MCP Might Be Needed (from ADR-003)

1. **Complex state management** across calls
2. **Real-time data streaming**
3. **Cross-tool coordination**
4. **IDE integration** (VSCode extension) ← ADR-010

### What We Can Learn from Roo Code's MCP Hub

Even though we're doing "Direct CLI first", if we DO need MCP later (e.g., IDE integration in Phase 4), their architecture shows:

1. **watchPaths is brilliant** ⭐
   - Auto-restart servers when config changes
   - No manual restart needed
   - Great UX

2. **Multi-transport support**
   - Start with stdio (simplest)
   - Add SSE/HTTP when needed for remote servers
   - Same interface for all

3. **Tool permission management**
   - `alwaysAllow` for safe tools
   - `disabledTools` for dangerous ops
   - Good security model

4. **Timeout configuration**
   - Prevent hanging calls
   - User-configurable per server
   - Default 60s is reasonable

5. **Project vs global config**
   - Global defaults in `~/.roo/mcp.json`
   - Project overrides in `.roo/mcp.json`
   - Clean separation

## Applicability to Our Project

### If We Need IDE Integration (ADR-010)

```typescript
// Our potential MCP server for IDE operations
{
  "mcpServers": {
    "vscode-ide": {
      "type": "stdio",
      "command": "node",
      "args": ["./mcp-servers/ide-server.js"],
      "watchPaths": [".cursorrules", "optimized-ai.config.json"],
      "alwaysAllow": ["formatDocument", "organizeImports"],
      "disabledTools": ["deleteFile"],
      "timeout": 30
    }
  }
}
```

**Benefits**:
- Auto-restart when `.cursorrules` changes
- Pre-approve safe operations (format, organize)
- Block dangerous operations (delete)
- Timeout protection

### If We Need Firebase/Supabase MCP (Phase 8)

**Only if** CLI proves insufficient:

```typescript
{
  "mcpServers": {
    "firebase-debug": {
      "type": "stdio",
      "command": "npx",
      "args": ["@optimized-ai/mcp-firebase"],
      "env": {
        "FIREBASE_PROJECT_ID": "${FIREBASE_PROJECT_ID}"
      },
      "watchPaths": [".firebaserc", "firebase.json"],
      "timeout": 60
    }
  }
}
```

**Advantages over CLI**:
- State management (maintain auth session)
- Real-time listeners (Firestore snapshots)
- Complex queries (join multiple collections)

**When CLI is better**:
- Simple queries (`firebase firestore:get users`)
- One-off operations
- No state needed

## Implementation Recommendations

### Phase 0 (Now): Document MCP Architecture

✅ Done - This document

### Phase 4 (If IDE Integration Needed)

1. **Evaluate**: Do we actually need MCP for IDE ops?
   - Can we use VSCode API directly?
   - Is there a CLI for IDE operations?

2. **If yes**: Implement minimal MCP Hub
   ```typescript
   class SimpleMcpHub {
     private connections: Map<string, McpConnection>

     async connect(serverName: string, config: ServerConfig) {
       // stdio transport only (start simple)
     }

     async callTool(serverName: string, tool: string, args: any) {
       // Basic tool calling
     }
   }
   ```

3. **Add features incrementally**:
   - Week 1: Basic stdio connection
   - Week 2: Tool calling
   - Week 3: watchPaths
   - Week 4: alwaysAllow/disabledTools
   - Week 5: Timeout handling

### Phase 8 (If Infrastructure MCPs Needed)

1. **Validate CLI insufficiency** first
2. **If MCP needed**: Adopt Roo Code patterns
   - watchPaths for config auto-reload
   - Timeout for slow operations
   - Tool permissions for safety

## Code Reference

**Key files to study**:
- `src/services/mcp/McpHub.ts` - Main hub implementation
- `src/services/mcp/McpServerManager.ts` - Server lifecycle management
- `src/core/prompts/tools/use-mcp-tool.ts` - Tool execution
- `src/core/prompts/tools/access-mcp-resource.ts` - Resource access
- `src/core/prompts/sections/mcp-servers.ts` - Prompt integration

## Conclusion

Roo Code's MCP Hub is a **comprehensive, production-ready MCP implementation**. Key innovations:

1. ⭐ **watchPaths** - Auto-restart on config changes (brilliant!)
2. **Multi-transport** - stdio/SSE/HTTP flexibility
3. **Tool permissions** - alwaysAllow/disabledTools for safety
4. **Project config override** - Global defaults + project specifics
5. **Timeout protection** - Prevent hanging calls

**Our approach**:
- **Now**: Direct CLI first (simpler, faster)
- **Later**: If MCP needed (IDE integration, complex state), adopt their patterns

**Priority**: LOW - Only implement if CLI proves insufficient in Phase 4 or Phase 8.
