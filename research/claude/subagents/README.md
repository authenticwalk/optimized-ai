# Claude Subagents Research

Comprehensive research on Claude Code subagents, their architecture, and best practices for the Optimized AI project.

**Last Updated**: 2025-11-03

---

## What Are Subagents?

**Subagents** are isolated Claude instances that can be spawned by the main agent to handle specialized tasks with fresh context windows.

### Key Characteristics

- **Fresh Context**: Each subagent starts with a clean context window
- **Specialized Tools**: Can be configured with specific tool subsets
- **Parallel Execution**: Multiple subagents can run simultaneously
- **Isolated Results**: Return focused summaries to main agent

### ⚠️ Critical Limitation: No Nested Subagents

**Subagents CANNOT call other subagents** - This is a hard constraint:

```
❌ NOT POSSIBLE:
Main Agent → Subagent A → Subagent B (BLOCKED)

✅ ONLY VALID:
Main Agent → Subagent A
Main Agent → Subagent B
```

**Implication**: All designs must use flat hierarchy (max 2 levels)

---

## Core Architecture

### Orchestrator-Worker Pattern

The proven pattern for multi-agent systems:

**Orchestrator (Main Agent)**:
- Coordinates workflow
- Distributes tasks
- Synthesizes results
- Never does heavy lifting

**Workers (Subagents)**:
- Execute specific tasks
- Work in isolation
- Return concise summaries
- Focus on single responsibility

### Configuration

Subagents are defined using YAML frontmatter:

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
tools: Read, Grep
model: claude-3-5-sonnet-20241022
---

Review the code and provide:
1. Security concerns
2. Code quality issues
3. Best practice violations
4. Specific recommendations
```

---

## When to Use Subagents

### ✅ Use Subagents For

- **Isolated exploration**: Research codebases without polluting main context
- **Parallel tasks**: Independent operations that can run simultaneously
- **Specialized analysis**: Security reviews, code quality checks
- **Large-scale operations**: Processing multiple files/repos
- **Token-heavy tasks**: Operations that would bloat main context

### ❌ Don't Use Subagents For

- **Simple operations**: Single file edits, basic searches
- **Sequential dependencies**: Tasks that need shared state
- **Frequent coordination**: High back-and-forth with main agent
- **Trivial tasks**: Anything under 1-2 minute execution time

**Cost**: Subagents add 15× token overhead - use only for high-value tasks

---

## Performance Metrics

From Anthropic's research:

- **90.2% improvement**: Multi-agent vs single agent (complex tasks)
- **15× token overhead**: Cost of multi-agent approach
- **90% time reduction**: Benefit of parallel execution
- **Optimal use**: Complex, parallelizable, high-value tasks

---

## Common Patterns

### 1. Map-Reduce Pattern

```
Main Agent: "Analyze all API endpoints"
  → Subagent A: Analyze endpoints 1-10
  → Subagent B: Analyze endpoints 11-20
  → Subagent C: Analyze endpoints 21-30
Main Agent: Synthesize all findings
```

### 2. Specialized Review Pattern

```
Main Agent: Implements feature
  → Security Subagent: Reviews for security issues
  → Quality Subagent: Reviews for code quality
  → Test Subagent: Reviews test coverage
Main Agent: Incorporates feedback
```

### 3. Research Pattern

```
Main Agent: Needs to understand codebase
  → Explore Subagent: Maps codebase structure
Main Agent: Uses structure to implement feature
```

---

## Best Practices

### 1. Keep Summaries Lightweight

Return 1-2k tokens, not 50k:

```
❌ Bad: Full code listing with analysis
✅ Good: Bullet points of findings + file:line refs
```

### 2. Filter Tools Aggressively

Only provide tools the subagent needs:

```yaml
# ❌ Too many tools
tools: Read, Write, Edit, Grep, Glob, Bash, WebFetch

# ✅ Minimal set
tools: Read, Grep
```

### 3. Use Descriptive Names

Help Claude choose the right subagent:

```yaml
# ❌ Generic
name: analyzer

# ✅ Specific
name: security-vulnerability-scanner
description: Scans code for security vulnerabilities (SQL injection, XSS, auth bypasses)
```

### 4. Avoid Context Handoff

Don't pass large context between agents:

```
❌ Pass full codebase → subagent → process
✅ Main knows structure → subagent reads specific files
```

### 5. Parallelize When Possible

Run independent subagents simultaneously:

```typescript
// Single response with multiple Task calls
[
  Task(code-reviewer),
  Task(test-generator),
  Task(documentation-writer)
]
```

---

## Token Optimization

### Reduce Subagent Token Usage

1. **Minimal prompts**: Keep agent descriptions concise
2. **Tool filtering**: Only essential tools in `tools:` list
3. **Focused tasks**: Single responsibility per agent
4. **Brief responses**: Request summaries, not full details

### Selective MCP Access

Subagents can specify which MCP servers to include:

```yaml
mcp_servers: [github, filesystem]  # Only these 2
```

This reduces system prompt size significantly.

---

## Integration with Skills

**Subagents + Skills** is powerful:

- Main agent loads appropriate skills
- Spawns specialized subagents
- Subagents inherit skills OR use filtered set
- Combine isolation + expertise

Example:
```
Main: Loads "firebase-security" skill
  → Security Subagent: Uses skill patterns to review
  → Returns findings using skill framework
Main: Applies fixes using skill patterns
```

---

## Project Alignment

For **Optimized AI**, use subagents for:

### Phase 3 Orchestration

```yaml
# Orchestrator (uses Claude Opus for coordination)
name: project-coordinator
description: Coordinates project implementation workflow
tools: Task
```

```yaml
# Workers (use Claude Sonnet for execution)
name: implementer
description: Implements features following established patterns
tools: Read, Write, Edit, Grep, Bash

name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep

name: test-engineer
description: Writes and runs tests
tools: Read, Write, Bash

name: quality-reviewer
description: Reviews code quality and best practices
tools: Read, Grep
```

### Experimental Framework

Use subagents in Phase 0 experiments to test:

- Impact of subagent vs single agent
- Optimal task granularity
- Token overhead vs quality improvement
- Parallel execution benefits

---

## Key Takeaways

1. **Subagents provide context isolation** - Fresh start for each task
2. **Flat hierarchy only** - No nesting allowed (hard constraint)
3. **High cost, high value** - 15× overhead, use selectively
4. **Orchestrator-worker pattern** - Proven architecture
5. **Lightweight summaries** - Return focused results
6. **Tool filtering** - Reduce context noise
7. **Parallel execution** - Major speed benefit
8. **Skills integration** - Combine isolation + expertise

---

## Related Research

- **[Skills](../skills/)** - Agent Skills architecture and progressive disclosure
- **[MCP](../mcp/)** - Model Context Protocol integration
- **[Triggers](../triggers/)** - Hooks and automation patterns
- **[Built-in Tools](../build-in-tools/)** - Claude Code tools reference

---

## Resources

### Official Documentation

- [Claude Subagents Docs](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Agent SDK Subagents](https://docs.claude.com/en/api/agent-sdk/subagents)
- [MCP Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)

### GitHub Resources

- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) - 100+ agents
- [wshobson/agents](https://github.com/wshobson/agents) - Production-ready collection
- [avivl/claude-007-agents](https://github.com/avivl/claude-007-agents) - Orchestration system

---

## Core Patterns Reference

**Progressive disclosure (3-tier)**:
- Tier 1: Metadata (5 tokens per skill)
- Tier 2: SKILL.md (~400 tokens when activated)
- Tier 3: Resources (~200 tokens each, as needed)

**Orchestrator-worker (flat hierarchy)**:
- Main agent = Air traffic controller
- Subagents = Specialized workers
- Communication = Lightweight summaries

**Map/reduce pattern**:
- Subagents explore independently
- Main agent synthesizes results

**File Formats**:
- Agents: YAML frontmatter + Markdown
- Skills: SKILL.md + resource files
- MCP: TypeScript/Python servers

---

**This research validates the Optimized AI architectural approach and provides concrete implementation guidance.**
