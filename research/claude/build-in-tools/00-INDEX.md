# Claude Code Built-in Tools - Complete Research

**Last Updated**: 2025-11-03
**Purpose**: Comprehensive documentation of Claude Code's built-in tools, how they work, and best practices aligned with the Optimized AI project goals.

## Project Context

This research supports the **Optimized AI** system goals:
- **MINIMIZE**: Maximum results, minimum overhead (token efficiency)
- **SEPARATE**: Context isolation via skills and subagents
- **VALIDATE**: Experimental validation of everything
- **LEARN**: Continuous optimization over time

## Documentation Structure

### Core Tool Documentation

1. **[File Operations](./01-FILE-OPERATIONS.md)** - Read, Edit, Write, MultiEdit
2. **[Search & Discovery](./02-SEARCH-DISCOVERY.md)** - Glob, Grep, file/content search
3. **[Execution & System](./03-EXECUTION-SYSTEM.md)** - Bash, command execution
4. **[Web Tools](./04-WEB-TOOLS.md)** - WebFetch, WebSearch
5. **[Task Management](./05-TASK-MANAGEMENT.md)** - TodoWrite, TodoRead
6. **[Agent Tools](./06-AGENT-TOOLS.md)** - Task, subagent dispatch, parallel execution
7. **[Notebook Operations](./07-NOTEBOOK-TOOLS.md)** - NotebookRead, NotebookEdit
8. **[Advanced Features](./08-ADVANCED-FEATURES.md)** - Skills, Hooks, MCP integration

### Best Practices & Optimization

9. **[Best Practices for Minimization](./09-BEST-PRACTICES-MINIMIZE.md)** - Token optimization strategies
10. **[Best Practices for Separation](./10-BEST-PRACTICES-SEPARATE.md)** - Context isolation patterns
11. **[Tool Performance Analysis](./11-PERFORMANCE-ANALYSIS.md)** - Speed, reliability, cost analysis
12. **[Common Pitfalls & Solutions](./12-PITFALLS-SOLUTIONS.md)** - Known issues and workarounds

## Key Findings Summary

### How Claude Code Works

**Architecture**: Single agent loop with 14+ built-in tools. Uses layered prompt system with security analysis on all Bash commands via Claude Haiku.

**Context Management**:
- Initial snapshot of directory structure (non-updating)
- Git status and recent commits
- Environment metadata (working directory, platform, date, model)
- System reminders injected periodically to combat context drift

**Model Selection**:
- Primary reasoning: Claude Sonnet 4.5
- Simpler tasks: Claude Haiku (10x cheaper for parsing)
- Security analysis: Claude Haiku

### File Editing - How It Works

**Edit Tool**:
- **Mechanism**: Exact string replacement (NOT regex-based)
- **No symbol parsing**: Does not use compiler or AST
- **No project knowledge database**: No Cursor-style index
- **Uniqueness requirement**: old_string must match exactly once (unless replace_all=true)
- **Line number handling**: Strips line number prefix before matching
- **When full rewrites happen**: When Edit would require too many changes, Claude may use Write instead
- **Avoiding full rewrites**: Use specific, unique strings; consider MultiEdit for multiple changes

**MultiEdit Tool**:
- Makes multiple find-and-replace operations on one file
- Applies edits sequentially (earlier edits affect later ones)
- All edits succeed or none apply (atomic)
- Better than multiple Edit calls for performance

**Read Tool**:
- **Adds line numbers**: Yes, using `cat -n` format (spaces + line_number + tab + content)
- **No project knowledge database**: Just reads file contents
- **Not symbol-based**: Returns raw text with line numbers
- **No smart indexing**: Each read is independent
- **Supports**: Text, images, PDFs, Jupyter notebooks
- **Default limit**: 2000 lines (configurable with offset/limit)

### Search Tools

**Glob Tool**:
- Fast file pattern matching using glob patterns
- Sorted by modification time (most recent first)
- Works on any codebase size
- Does NOT use regex (uses glob syntax like `**/*.js`)

**Grep Tool**:
- Content search using ripgrep
- DOES use regex patterns
- **Known issue**: ~50% failure rate in some scenarios
- Supports multiline mode, context lines, case-insensitive search
- File filtering via glob parameter or type parameter

### Other Available Tools

**Standard Tools**:
- Bash (command execution with security scanning)
- WebFetch (URL retrieval with Haiku summarization, 15-min cache)
- WebSearch (search results, US-only, no content)
- TodoWrite/TodoRead (task management)
- NotebookRead/NotebookEdit (Jupyter support)

**Advanced Tools**:
- Task (subagent dispatch, max 10 parallel)
- Skills (auto-loaded instruction sets via SKILL.md)
- Hooks (custom commands before/after tool execution)
- MCP servers (external tool integration, prefix: `mcp__`)

**Note on "askUserQuestion"**: No documented tool with this exact name found. May be:
- Custom MCP tool in specific examples
- Misremembered name for interactive prompts
- Feature from Claude.ai (not Claude Code)
- Part of skill instructions rather than a tool

### Replace-All Capability

**Edit Tool**: Yes, has `replace_all` parameter
- When true: Replaces ALL occurrences of old_string
- Useful for renaming variables across a file
- Still requires exact string match (not regex)

**MultiEdit Tool**: Each operation can specify its own replace behavior

### Compiler/Symbol Usage

**No built-in compiler integration**:
- Claude does NOT parse code into symbols/AST
- No automatic "find all references"
- No built-in refactoring engine
- File operations are text-based

**IDE Integration (Recommended)**:
- Project spec suggests exposing IDE operations via MCP:
  - `ide.renameSymbol(oldName, newName)`
  - `ide.organizeImports(file)`
  - `ide.findReferences(symbol)`
- This would leverage IDE's symbol parsing instead of recreating it

## Research Methodology

**Sources**:
- Official Claude documentation (docs.claude.com)
- GitHub Gists with internal tool implementations
- Technical blog posts and deep dives
- Community resources (ClaudeLog, etc.)
- GitHub issues and discussions

**Validation**:
- Cross-referenced multiple sources
- Noted contradictions and uncertainties
- Documented known issues from user reports
- Aligned findings with project goals

## Next Steps

1. **Validate with Experiments** (Phase 0 framework)
2. **Optimize Tool Usage** based on token costs
3. **Build Skill Library** leveraging tool knowledge
4. **Create Hooks** for automation workflows
5. **Design MCP Servers** only where necessary (prefer direct scripts)

## Contributing

As we run experiments and learn more:
1. Update relevant tool documentation
2. Add learnings to best practices
3. Document new pitfalls/solutions
4. Refine optimization strategies

---

**Related Documents**:
- `.plan/initial-design/SPEC.md` - System specification
- `.plan/initial-design/principles/` - Core principles
- `.plan/initial-design/PLAN.md` - Phase 0 experimental framework
