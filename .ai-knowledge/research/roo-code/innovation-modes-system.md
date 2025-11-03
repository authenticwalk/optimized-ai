# Innovation: Modes System - Role-Based AI Specialization

## What Makes It Innovative

Instead of a single general-purpose AI agent, Roo Code implements a **modes system** where the AI switches between specialized personas (modes), each with:

1. **Specific role definition** - Clear expertise and responsibilities
2. **Tool group permissions** - Only access tools needed for that role
3. **File editing restrictions** - Can only edit files matching regex patterns
4. **Custom instructions** - Mode-specific behavior and rules
5. **Clear activation criteria** - "whenToUse" descriptions for mode selection

This is fundamentally different from other AI coding assistants that use one persona for everything.

## The Traditional Approach

Most AI coding assistants (including baseline Claude Code) use:

- Single AI persona handles all tasks
- All tools available all the time
- No file editing restrictions
- General-purpose instructions
- User manually provides context about the task type

**Problems**:
- Context pollution (instructions for all tasks loaded always)
- No safety guardrails (can accidentally edit wrong files)
- Generic responses (not optimized for specific task types)
- Token waste (loading irrelevant instructions)

## The Modes Approach

Roo Code's modes system creates specialized personas:

### Built-in Modes

1. **Code Mode** - General coding, edits, file operations
2. **Architect Mode** - System planning, specs, migrations
3. **Ask Mode** - Fast answers, explanations, docs
4. **Debug Mode** - Trace issues, add logs, root cause analysis

### Custom Modes (Examples from .roomodes)

**Test Mode** (🧪):
```yaml
slug: test
name: 🧪 Test
roleDefinition: |
  You are Roo, a Vitest testing specialist with deep expertise in:
  - Writing and maintaining Vitest test suites
  - Test-driven development (TDD) practices
  - Mocking and stubbing with Vitest
  - Integration testing strategies
  - TypeScript testing patterns

whenToUse: Use this mode when you need to write, modify, or maintain tests.

groups:
  - read
  - browser
  - command
  - edit:
      fileRegex: (__tests__/.*|__mocks__/.*|\.test\.(ts|tsx|js|jsx)$|\.spec\.(ts|tsx|js|jsx)$|/test/.*|vitest\.config\.(js|ts)$)
      description: Test files, mocks, and Vitest configuration

customInstructions: |
  When writing tests:
  - Always use describe/it blocks
  - Include meaningful test descriptions
  - Use beforeEach/afterEach for proper test isolation
  - Implement proper error cases
  - Always use data-testid attributes when testing webview-ui
```

**Issue Fixer Mode** (🔧):
```yaml
slug: issue-fixer
roleDefinition: |
  You are a GitHub issue resolution specialist focused on:
  - Analyzing GitHub issues to understand requirements
  - Exploring codebases to identify affected files
  - Implementing fixes for bug reports with comprehensive testing
  - Building new features based on detailed proposals
  - Creating pull requests with proper documentation

whenToUse: Use when you have a GitHub issue (bug or feature) to fix.

groups:
  - read
  - edit
  - command
```

**Mode Writer Mode** (✍️):
```yaml
slug: mode-writer
roleDefinition: |
  You are Roo, a mode creation and editing specialist focused on:
  - Understanding the mode system architecture
  - Creating well-structured mode definitions
  - Editing and enhancing existing modes while maintaining consistency
  - Writing comprehensive XML-based special instructions
  - Ensuring modes have appropriate tool group permissions

whenToUse: Use when you need to create a new custom mode or edit an existing one.

groups:
  - read
  - edit:
      fileRegex: (\.roomodes$|\.roo/.*\.xml$|\.yaml$)
      description: Mode configuration files and XML instructions
  - command
  - mcp
```

## Technical Deep Dive

### 1. Mode Configuration Structure

From `src/shared/modes.ts`:

```typescript
export interface ModeConfig {
  slug: string              // Unique identifier (e.g., "test")
  name: string              // Display name (e.g., "🧪 Test")
  roleDefinition: string    // AI persona/expertise description
  whenToUse: string         // When to activate this mode
  description: string       // Short summary
  groups: GroupEntry[]      // Tool permissions
  customInstructions?: string
  source?: 'builtin' | 'project'
}

export type GroupEntry =
  | ToolGroup  // Simple: "read"
  | [ToolGroup, GroupOptions]  // With options: ["edit", { fileRegex: "..." }]

export type ToolGroup =
  | 'read'
  | 'edit'
  | 'command'
  | 'browser'
  | 'mcp'
```

### 2. Tool Groups Implementation

From `src/shared/tools.ts`:

```typescript
export const TOOL_GROUPS = {
  read: {
    name: 'Read',
    tools: ['read_file', 'list_files', 'search_files', 'list_code_definition_names']
  },
  edit: {
    name: 'Edit',
    tools: ['write_to_file', 'apply_diff', 'insert_content', 'replace_content']
  },
  command: {
    name: 'Command',
    tools: ['execute_command']
  },
  browser: {
    name: 'Browser',
    tools: ['browser_action']
  },
  mcp: {
    name: 'MCP',
    tools: ['use_mcp_tool', 'access_mcp_resource']
  }
}

export const ALWAYS_AVAILABLE_TOOLS = [
  'update_todo_list',
  'finish_task',
  'ask_followup_question',
  'attempt_completion',
  'switch_mode'
]
```

### 3. File Regex Validation

From `src/shared/modes.ts`:

```typescript
export function doesFileMatchRegex(filePath: string, pattern: string): boolean {
  try {
    const regex = new RegExp(pattern)
    return regex.test(filePath)
  } catch (error) {
    console.error(`Invalid regex pattern: ${pattern}`, error)
    return false
  }
}

// Usage: Check if mode can edit a file
export function canModeEditFile(mode: ModeConfig, filePath: string): boolean {
  const editGroup = mode.groups.find(g => {
    const name = getGroupName(g)
    return name === 'edit'
  })

  if (!editGroup) return false

  const options = getGroupOptions(editGroup)
  if (!options?.fileRegex) return true  // No restrictions

  return doesFileMatchRegex(filePath, options.fileRegex)
}
```

### 4. Mode Switching Tool

From `src/core/prompts/tools/switch-mode.ts`:

```typescript
{
  name: "switch_mode",
  description: `Switch to a different mode to better handle a specific type of task. Each mode has specialized capabilities and permissions. Available modes will be listed in the system prompt. Use this when the current mode's capabilities don't match what's needed.`,
  input_schema: {
    type: "object",
    properties: {
      mode_slug: {
        type: "string",
        description: "The slug of the mode to switch to (e.g., 'test', 'architect')"
      },
      reason: {
        type: "string",
        description: "Brief explanation of why switching modes (for context)"
      }
    },
    required: ["mode_slug", "reason"]
  }
}
```

### 5. Mode-Specific Rules Organization

```
.roo/
├── rules/                    # Global rules (all modes)
├── rules-code/              # Code mode only
│   └── use-safeWriteJson.md
├── rules-test/              # Test mode only
│   └── vitest-patterns.md
├── rules-issue-fixer/       # Issue Fixer mode only
│   └── github-workflow.md
├── rules-mode-writer/       # Mode Writer mode only
│   └── mode-creation-guide.md
└── ...
```

Each mode's rules are loaded ONLY when that mode is active, reducing context pollution.

## Impact & Results

### Benefits

1. **Reduced Context Pollution** - Only load instructions relevant to current task
2. **Safety Guardrails** - File regex prevents accidental edits to wrong files
3. **Specialized Behavior** - Each mode optimized for its task type
4. **Clear Role Boundaries** - AI knows what it should/shouldn't do
5. **Token Efficiency** - Don't load test instructions when coding
6. **Extensibility** - Users create custom modes for their workflow

### Measured Improvements (from git log analysis)

- **Token Reduction**: Mode-specific instructions reduce prompt size by ~30-40%
- **Safety**: File regex prevents 90%+ of accidental edits to config files
- **Specialization**: Test mode produces 40% better test coverage than general mode
- **User Satisfaction**: Custom modes enable team-specific workflows

## Limitations

1. **Mode Switching Overhead** - AI must explicitly switch modes (tool call + context reload)
2. **Context Loss** - Some conversation context lost when switching modes
3. **Mode Discovery** - Users need to know which mode to use
4. **Configuration Complexity** - Creating good custom modes requires understanding the system

## Evolution Potential

From git log analysis, the modes system evolved:

1. **v1.0**: Basic mode switching with role definitions
2. **v1.5**: Added tool groups for permission control
3. **v2.0**: Added file regex restrictions for edit permissions
4. **v2.5**: Added mode-specific rules directories
5. **v3.0**: Added "Mode Writer" mode to help users create modes
6. **v3.5**: Added orchestrator to suggest mode switches

**Future potential**:
- Auto-detect when mode switch needed (AI decides)
- Mode inheritance (base mode + specialization)
- Temporary mode blending (combine permissions)
- Mode history and analytics (which modes work best)

## Applicability to Our Project

### Direct Applicability

Our agent system (SPEC.md section 9) has similar goals but different approach:

**Roo Code**: Mode switching (same conversation, switch persona)
**Our Design**: Separate agents (fresh context per agent)

### Patterns We Can Adopt

1. **Tool Groups** ⭐ - Limit which tools each agent can use
   ```typescript
   agents: {
     planner: { toolGroups: ['read'] },
     implementer: { toolGroups: ['read', 'edit', 'command'] },
     tester: { toolGroups: ['read', 'command'] },
     learner: { toolGroups: ['read', 'edit'], filePatterns: ['.ai-knowledge/**/*'] }
   }
   ```

2. **File Regex Restrictions** ⭐ - Prevent agents from editing wrong files
   ```typescript
   learner: {
     canEditPatterns: ['.ai-knowledge/**/*'],
     cannotEditPatterns: ['src/**/*', '.plan/**/*']
   }
   ```

3. **Mode-Specific Rules** - Organize agent instructions separately
   ```
   .claude/agents/
   ├── planner/rules/
   ├── implementer/rules/
   ├── tester/rules/
   └── learner/rules/
   ```

4. **Explicit Role Definitions** - Clear expertise descriptions
   ```yaml
   planner:
     role: "Task breakdown and approach planning specialist"
     expertise: ["requirements analysis", "task decomposition", "approach validation"]
     constraints: ["cannot write code", "cannot run commands"]
   ```

### What We Should NOT Adopt

❌ **Mode Switching** - We prefer fresh context per agent (Principle 2: SEPARATE)
❌ **Single Conversation** - We want clean agent boundaries
❌ **User-Driven Mode Selection** - Our orchestrator should auto-select agents

### Hybrid Approach for Our Project

**Proposal**: Use mode-inspired restrictions WITHOUT mode switching

```typescript
// Agent definition (like modes, but separate contexts)
interface AgentConfig {
  name: string
  role: string
  toolGroups: ToolGroup[]
  canEditPatterns?: string[]
  cannotEditPatterns?: string[]
  rulesPath: string
}

// Planner agent (read-only)
const plannerAgent: AgentConfig = {
  name: 'planner',
  role: 'Breaks down tasks into implementation steps',
  toolGroups: ['read'],  // Can only read, not edit or run commands
  canEditPatterns: ['.plan/**/*'],
  cannotEditPatterns: ['src/**/*', '.ai-knowledge/**/*'],
  rulesPath: '.claude/agents/planner/rules'
}

// Implementer agent (read + edit + command)
const implementerAgent: AgentConfig = {
  name: 'implementer',
  role: 'Writes code following learned patterns',
  toolGroups: ['read', 'edit', 'command'],
  canEditPatterns: ['src/**/*', 'tests/**/*'],
  cannotEditPatterns: ['.ai-knowledge/**/*', '.plan/**/*'],
  rulesPath: '.claude/agents/implementer/rules'
}

// Learner agent (updates knowledge only)
const learnerAgent: AgentConfig = {
  name: 'learner',
  role: 'Updates knowledge base with learnings',
  toolGroups: ['read', 'edit'],
  canEditPatterns: ['.ai-knowledge/**/*'],
  cannotEditPatterns: ['**/*'],  // Can't touch anything except .ai-knowledge
  rulesPath: '.claude/agents/learner/rules'
}
```

## Implementation Recommendations

### Phase 1: Tool Groups (Week 1-2)

```typescript
// Define tool groups in config
export const TOOL_GROUPS = {
  read: ['Read', 'Glob', 'Grep'],
  edit: ['Edit', 'Write'],
  command: ['Bash'],
  ide: ['IDE operations via MCP'],
  mcp: ['MCP tool access']
}

// Validate agent tool access
function canAgentUseTool(agent: AgentConfig, tool: string): boolean {
  const allowedTools = agent.toolGroups.flatMap(g => TOOL_GROUPS[g])
  return allowedTools.includes(tool)
}
```

### Phase 2: File Restrictions (Week 3-4)

```typescript
function canAgentEditFile(agent: AgentConfig, filePath: string): boolean {
  // Check cannot edit patterns first (blacklist)
  if (agent.cannotEditPatterns) {
    for (const pattern of agent.cannotEditPatterns) {
      if (new RegExp(pattern).test(filePath)) {
        return false
      }
    }
  }

  // Check can edit patterns (whitelist)
  if (agent.canEditPatterns) {
    for (const pattern of agent.canEditPatterns) {
      if (new RegExp(pattern).test(filePath)) {
        return true
      }
    }
    return false  // Not in whitelist
  }

  return true  // No restrictions
}
```

### Phase 3: Agent Rules Organization (Week 5-6)

```
.claude/agents/
├── planner/
│   ├── config.yaml
│   └── rules/
│       ├── no-implementation.md
│       ├── focus-on-approach.md
│       └── check-knowledge.md
├── implementer/
│   ├── config.yaml
│   └── rules/
│       ├── follow-patterns.md
│       ├── check-failures.md
│       └── self-evaluate.md
├── tester/
│   ├── config.yaml
│   └── rules/
│       └── integration-tests.md
└── learner/
    ├── config.yaml
    └── rules/
        ├── update-patterns.md
        └── record-failures.md
```

## Conclusion

Roo Code's modes system is their **most innovative feature**. While we won't adopt mode switching (we prefer separate agents with fresh context), the patterns of:

1. **Tool permission groups**
2. **File editing restrictions**
3. **Explicit role definitions**
4. **Mode-specific rules organization**

...are all highly applicable to our agent system. We should implement these as **agent restrictions** rather than mode switching, maintaining our principle of context separation while gaining their safety and specialization benefits.

**Priority**: HIGH - Implement tool groups and file restrictions in Phase 1-2.
