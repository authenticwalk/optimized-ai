# Roo Code Research

## What They Built

Roo Code is a VSCode extension that provides "Your AI-Powered Dev Team, Right in Your Editor." It's built as a sophisticated multi-modal AI coding assistant with:

- **Modes System**: Different AI personas for different tasks (Code, Architect, Ask, Debug, Custom)
- **MCP Server Support**: Full integration with Model Context Protocol servers
- **Checkpoints**: Save/restore conversation state
- **Todo Lists**: Built-in task tracking
- **Custom Modes**: Users can create specialized modes for their workflow
- **Roomote Control**: Remote control of local VS Code tasks

## Why We're Researching This

We're building a self-learning AI system (optimized-ai) that enables PM-mode workflow. Roo Code's sophisticated mode system, workflow orchestration, and MCP integration offer valuable patterns that could inform our:

1. **Agent/Subagent Design** (SPEC.md section 9) - Their modes system shows how to specialize AI behavior
2. **Tool Groups & Permissions** - How they control what each mode can access
3. **MCP Server Patterns** - Real-world MCP implementation insights
4. **Workflow Orchestration** - How they coordinate complex tasks
5. **Skills System** - Their custom modes align with our on-demand skills concept

## What We Explored

### Core Innovations
- [Modes System - Role-Based AI Specialization](./innovation-modes-system.md)
- [Tool Groups - Permission-Based Tool Access](./innovation-tool-groups.md)
- [File Regex Restrictions - Scoped Editing Permissions](./pattern-file-regex-editing.md)
- [Checkpoint System - Conversation State Management](./feature-checkpoints.md)

### MCP Implementation
- [MCP Hub Architecture](./mcp-hub-architecture.md)
- [MCP Server Types - stdio, SSE, streamable-http](./mcp-server-types.md)
- [MCP Tool Integration](./mcp-tools-integration.md)

### Workflow & Patterns
- [Orchestrator Pattern for Code Indexing](./pattern-orchestrator.md)
- [Atomic File Writes - safeWriteJson](./pattern-atomic-writes.md)
- [Mode-Specific Rules Organization](./pattern-mode-rules.md)

### Git Log Insights
- [Mistakes & Fixes](./git-mistakes-analysis.md)
- [Feature Evolution](./git-improvements-analysis.md)

## What We Skipped

- **UI Implementation** - React/Tailwind webview (not relevant to our CLI focus)
- **VSCode Extension API** - We're focusing on Claude Code, not VSCode
- **Deployment/Packaging** - Different distribution model
- **Localization System** - i18n (not a priority for us)
- **Telemetry/Analytics** - Different privacy approach
- **Cloud Features** - "Roo Code Cloud" integration
- **Browser Automation** - Puppeteer-based browser tools

## Key Insights Summary

### 1. Modes as Specialized Agents
**Learning**: Instead of one general-purpose AI, Roo Code uses specialized "modes" (personas) for different tasks.

**Application to Our Project**:
- Our `planner`, `implementer`, `tester`, `reviewer` agents (SPEC.md section 9) could be implemented as modes
- Each mode has specific tool access and file editing permissions
- Mode switching is explicit via `switch_mode` tool

### 2. Tool Groups for Permission Control
**Learning**: Tools are organized into groups (`read`, `edit`, `command`, `browser`, `mcp`), and modes specify which groups they can access.

**Application to Our Project**:
- Could limit our agents to specific operations (planner can't edit, tester can read/command)
- Prevents agents from doing unintended actions
- Clean separation of concerns

### 3. File Regex Patterns for Scoped Editing
**Learning**: Modes can restrict editing to specific file patterns (e.g., Test mode only edits `__tests__/**` files).

**Application to Our Project**:
- Could prevent our agents from modifying critical config files
- Ensure learning/documentation agents don't touch production code
- Safety mechanism for our `.ai-knowledge/` folder

### 4. MCP Hub with Multiple Transport Types
**Learning**: Comprehensive MCP implementation supporting stdio, SSE, and streamable-http transports with file watching and auto-restart.

**Application to Our Project**:
- We decided on "Direct CLI over MCP" (ADR-003), but their implementation shows when MCP is valuable
- Their watchPaths feature auto-restarts servers when config changes
- Good model if we need MCP for IDE integration (ADR-010)

### 5. Orchestrator Pattern for Complex Workflows
**Learning**: `CodeIndexOrchestrator` coordinates between managers (config, state, cache, vector store, scanner, file watcher).

**Application to Our Project**:
- Could apply to our agent coordination (section 9)
- Clean separation: orchestrator coordinates, managers execute
- State management pattern useful for our `.plan/` workflow

## Unique Offerings vs Claude Code/Cursor

### Missing from Claude Code that Roo Code Has:

1. **Modes System** ⭐ (BIGGEST DIFFERENTIATOR)
   - Explicit role switching with different tool permissions
   - File-scoped editing permissions per mode
   - Custom mode creation with YAML config

2. **Checkpoints**
   - Save/restore conversation state
   - Rollback to previous states

3. **Built-in Todo Lists**
   - Task tracking within the agent
   - Progress monitoring

4. **Mode-Specific Rules/Instructions**
   - Each mode has its own `.roo/rules-{mode}/` directory
   - Modular instruction organization

5. **File Regex Editing Restrictions**
   - Prevent modes from editing files outside their scope
   - Safety mechanism

6. **Tool Groups**
   - Granular permission control
   - read/edit/command/browser/mcp groups

7. **Atomic File Writes**
   - `safeWriteJson` with locking and streaming
   - Prevents corruption during concurrent writes

8. **Code Indexing with Vector Store**
   - Orchestrator pattern for codebase indexing
   - File watching and incremental updates

9. **MCP File Watching**
   - Auto-restart MCP servers when watched files change
   - `watchPaths` configuration

10. **Custom Mode Creation UI**
    - YAML-based mode definition
    - Mode Writer mode to help create modes

## Comparison to Our Project Goals

### Alignment with Our Principles

✅ **SEPARATE** (Principle 2): Their modes system enforces separation through tool groups and file restrictions

✅ **VALIDATE** (Principle 3): Their atomic writes and error handling show defensive programming

❌ **MINIMIZE** (Principle 1): Their system is comprehensive, not minimal - loads all modes at once

❌ **LEARN** (Principle 4): No visible self-learning/knowledge base like our `.ai-knowledge/` approach

### Architectural Differences

| Feature | Roo Code | Our Design (SPEC.md) |
|---------|----------|---------------------|
| **Agent Model** | Modes (role switching) | Separate agents with coordination |
| **Tool Access** | Permission groups per mode | Implicit based on agent role |
| **Context** | Single conversation with mode switches | Fresh context per subagent |
| **Learning** | Not visible | Explicit `.ai-knowledge/` folder |
| **Spin Detection** | Not visible | Auto-detect and switch approaches |
| **File Safety** | Regex restrictions | Clean workspace principles |
| **MCP** | Full integration, multiple transports | "Direct CLI first" (ADR-003) |
| **Workflow** | Mode-based | PM-mode with PR workflow |

## Recommendations for Our Project

### 1. Consider Mode-Inspired Agent Restrictions ⭐
**Why**: Their file regex and tool group patterns could enhance our agent safety.

**How**:
```typescript
// In our agent definitions
agents: {
  planner: {
    toolGroups: ['read'],  // Can't edit or run commands
    filePatterns: ['.plan/**/*']  // Only touches planning files
  },
  implementer: {
    toolGroups: ['read', 'edit', 'command'],
    filePatterns: ['src/**/*', 'tests/**/*'],  // Can't touch .ai-knowledge or .plan
    excludePatterns: ['.ai-knowledge/**/*', '.plan/**/*']
  },
  learner: {
    toolGroups: ['read', 'edit'],
    filePatterns: ['.ai-knowledge/**/*']  // Only updates knowledge
  }
}
```

### 2. Adopt Atomic File Writes for .ai-knowledge/
**Why**: Our knowledge base JSON files are critical. Corruption would be catastrophic.

**Recommendation**: Implement `safeWriteJson` pattern for all `.ai-knowledge/` writes.

### 3. Consider Checkpoint-Like State Management
**Why**: Our `.plan/` folder tracks state, but checkpoints could help with rollback.

**How**: Save snapshots of `.plan/` and `.ai-knowledge/` at key milestones.

### 4. Tool Groups for Agent Permissions
**Why**: Explicit permissions are clearer than implicit agent roles.

**Recommendation**: Define tool groups in `optimized-ai.config.json`:
```json
{
  "toolGroups": {
    "read": ["read_file", "grep", "glob"],
    "edit": ["edit_file", "write_file"],
    "command": ["bash", "npm", "git"],
    "ide": ["ide.renameSymbol", "ide.formatDocument"]
  }
}
```

### 5. MCP Hub Pattern (If Needed Later)
**Why**: If we need MCP for IDE integration (ADR-010), their hub architecture is solid.

**When**: Phase 4 (IDE integration) or Phase 8 (Infrastructure MCPs)

**What to Adopt**:
- Multi-transport support
- watchPaths for auto-restart
- Disabled tools tracking
- Timeout configuration

### 6. Mode-Specific Instructions Organization
**Why**: Our skills could benefit from organized, mode-specific rules.

**How**:
```
.claude/skills/
├── planner/
│   ├── skill.yaml
│   └── rules/
│       ├── no-implementation.md
│       └── focus-on-approach.md
├── implementer/
│   ├── skill.yaml
│   └── rules/
│       ├── check-knowledge.md
│       └── follow-patterns.md
```

### 7. Orchestrator Pattern for Agent Coordination
**Why**: Their orchestrator pattern fits our agent coordination needs (SPEC.md section 9).

**How**: Create `AgentOrchestrator` to coordinate planner → implementer → tester → reviewer flow.

## What NOT to Adopt

1. ❌ **Mode Switching** - We prefer separate subagents with fresh context (ADR Principle 2)
2. ❌ **Comprehensive MCP** - We're doing "Direct CLI first" (ADR-003)
3. ❌ **VSCode-Specific Integration** - We're focused on Claude Code
4. ❌ **Cloud Features** - Not our focus
5. ❌ **Browser Automation** - Out of scope
6. ❌ **Built-in Todo UI** - We have TodoWrite tool

## Implementation Priority

### High Priority (Phase 1-2)
1. ✅ Atomic file writes for `.ai-knowledge/`
2. ✅ Tool groups and permissions
3. ✅ File pattern restrictions for agents

### Medium Priority (Phase 3-4)
4. Orchestrator pattern for agent coordination
5. Checkpoint-like state snapshots
6. Mode-specific rules organization

### Low Priority (Phase 5+)
7. MCP Hub (only if needed for IDE integration)
8. Advanced file watching patterns

## Files for Deep-Dive Reference

Key files to study if implementing these patterns:

- `src/shared/modes.ts` - Mode system core
- `src/shared/tools.ts` - Tool groups definition
- `src/services/mcp/McpHub.ts` - MCP implementation
- `src/services/code-index/orchestrator.ts` - Orchestrator pattern
- `src/utils/safeWriteJson.ts` - Atomic writes
- `.roomodes` - Mode configuration format
- `.roo/rules-*/**/*.md` - Mode-specific rules

## Conclusion

Roo Code's **modes system** is their most innovative feature - it shows how to create specialized AI personas with controlled permissions. While we're taking a different approach (separate agents vs mode switching), their patterns for:

1. **Tool permission groups**
2. **File regex restrictions**
3. **Atomic file writes**
4. **Orchestrator coordination**
5. **MCP multi-transport support**

...are all valuable patterns we should selectively adopt. The key is adapting their safety and organization patterns while maintaining our principles of fresh context, self-learning, and minimal overhead.
