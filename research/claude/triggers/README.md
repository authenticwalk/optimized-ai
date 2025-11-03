# Claude Triggers & Automation Research

**Research Date**: 2025-11-03
**Purpose**: Comprehensive research on Claude Code triggers, hooks, automation patterns, and best practices to support the Optimized AI system design.

## Overview

This directory contains extensive research on Claude Code's trigger and automation systems, including hooks, subagents, MCP integration, skills architecture, and optimization patterns. This research directly supports the goals outlined in `.plan/initial-design`:

- **MINIMIZE**: Token efficiency through smart context management
- **SEPARATE**: Context isolation via skills and subagents
- **VALIDATE**: Data-driven optimization through experiments
- **LEARN**: Self-improving systems that adapt over time

## Research Documents

### Core Systems

1. **[hooks-reference.md](./hooks-reference.md)** - Complete Claude Code hooks system
   - All 8 hook lifecycle events (UserPromptSubmit, PreToolUse, PostToolUse, etc.)
   - Configuration patterns and JSON control structures
   - Security implementations and blocking patterns
   - Real-world examples and repositories

2. **[subagent-patterns.md](./subagent-patterns.md)** - Subagent coordination and orchestration
   - Orchestrator-worker patterns
   - Sequential pipelines and parallel processing
   - Task delegation strategies
   - Context isolation benefits

3. **[mcp-integration.md](./mcp-integration.md)** - Model Context Protocol triggers and patterns
   - MCP server architecture
   - Tools, Resources, and Prompts primitives
   - Knowledge graph persistence patterns
   - Event-driven integration

4. **[skills-architecture.md](./skills-architecture.md)** - On-demand skill loading systems
   - Progressive disclosure patterns (3-level loading)
   - Context isolation techniques
   - LLM-based routing mechanisms
   - Skill composition strategies

### Optimization & Quality

5. **[spin-detection.md](./spin-detection.md)** - Detecting and preventing AI loops
   - Loop detection algorithms
   - State-based spin detection
   - Automatic retry and recovery patterns
   - Checkpoint systems

6. **[context-optimization.md](./context-optimization.md)** - Token and context management
   - Intelligent chunking strategies
   - Strategic context selection
   - RAG-based retrieval patterns
   - Prompt caching techniques

7. **[best-practices.md](./best-practices.md)** - General optimization principles
   - Official Anthropic recommendations
   - Community-proven patterns
   - Multi-Claude workflows
   - Visual iteration techniques

### Implementation

8. **[automation-workflows.md](./automation-workflows.md)** - Common automation patterns
   - Pre-commit validation workflows
   - Auto-formatting and linting
   - Test automation triggers
   - PR creation and review automation

9. **[examples.md](./examples.md)** - Practical code examples and snippets
   - Hook implementations (Python, Ruby, Bash)
   - Configuration templates
   - Real repository examples
   - Production-ready patterns

10. **[repositories.md](./repositories.md)** - Notable repositories and resources
    - Open source examples
    - Community tools and libraries
    - Official references
    - Integration examples

## Key Findings Summary

### 1. Hooks Enable Deterministic Control

Claude Code hooks provide 8 lifecycle events that allow deterministic control over AI behavior without relying on LLM reasoning:

- **PreToolUse**: Validate/modify/block operations before execution
- **PostToolUse**: Validate results and provide feedback
- **UserPromptSubmit**: Inject context, validate prompts, block dangerous requests
- **Stop/SubagentStop**: Control completion criteria
- **SessionStart/SessionEnd**: Initialize/cleanup workflows
- **Notification**: Handle permission requests
- **PreCompact**: Backup before context compression

### 2. Subagents Provide Context Isolation

Subagents operate with isolated context windows, enabling:
- Fresh context for specialized tasks
- Parallel execution with independent agents
- Task-specific expertise without polluting main context
- Reusable agent configurations across projects

### 3. Skills Support Progressive Disclosure

Skills use 3-level loading to minimize tokens:
1. **Startup**: Load only skill names/descriptions (30-50 tokens each)
2. **Activation**: Load full skill markdown when needed
3. **Resources**: Load supporting files only when accessed

### 4. MCP Enables Extensibility

The Model Context Protocol provides standardized integration for:
- Custom tools (AI-controlled actions)
- Resources (context injection)
- Prompts (user-invoked templates)
- Knowledge graphs (persistent memory)

### 5. Spin Detection Requires State Tracking

Effective spin detection monitors:
- Repetitive file edits (3+ times in 2 minutes)
- Repeated errors (same error 3+ times)
- Token usage spikes (>50k in one response)
- Repetitive tool calls (same call 3+ times)
- Activity without progress (5+ minutes)

## Alignment with Project Goals

### MINIMIZE Principle

**Findings that support token minimization:**
- Skills load progressively (30-50 tokens vs full content)
- Subagents isolate context (prevents bloat)
- Context management best practices (strategic selection, chunking)
- Prompt caching reduces repeated content costs

**Recommended Patterns:**
- Use skills for domain-specific knowledge (firebase, testing, etc.)
- Deploy subagents for complex multi-step tasks
- Implement aggressive context pruning via `/clear`
- Use CLAUDE.md for persistent project knowledge

### SEPARATE Principle

**Findings that support separation of concerns:**
- Subagents provide complete context isolation
- Skills load on-demand based on task requirements
- Hooks can be matcher-filtered to specific tool types
- MCP servers isolate capabilities by domain

**Recommended Patterns:**
- One skill per domain (firebase-auth, supabase-rls, etc.)
- Dedicated subagents for: planner, implementer, reviewer, tester
- Hook matchers filter to relevant tools (Edit|Write vs Bash vs Read)
- Separate MCP servers for distinct capabilities

### VALIDATE Principle

**Findings that support experimental validation:**
- Hooks enable automatic test execution (PostToolUse)
- Pre-commit validation via PreToolUse hooks
- Quality gates before PR creation (Stop hook)
- Metrics collection via hook logging

**Recommended Patterns:**
- PostToolUse hook runs tests after code changes
- Stop hook validates all requirements met before completion
- Hooks log JSON metrics for A/B testing framework
- PreCompact hook preserves conversation transcripts

### LEARN Principle

**Findings that support self-learning systems:**
- MCP knowledge graph persistence patterns
- Hook-based pattern capture (successful approaches)
- Transcript logging for post-analysis
- Configuration evolution through metrics

**Recommended Patterns:**
- MCP memory server for knowledge graph storage
- PostToolUse hook captures successful patterns
- UserPromptSubmit hook injects learned patterns as context
- Periodic analysis of hook logs to refine rules

## Critical Patterns for Implementation

### 1. Spin Detection Hook (Priority: HIGH)

Implement PreToolUse hook that detects spin patterns:

```python
# Check for repetitive edits
if same_file_edited_count >= 3 and time_window <= 120:
    return {"decision": "block", "reason": "Spin detected: same file edited 3+ times in 2 minutes"}

# Check for repetitive errors
if same_error_count >= 3:
    return {"decision": "block", "reason": "Spin detected: same error repeated 3+ times"}
```

### 2. Skills Auto-Loading (Priority: HIGH)

Configure skills that load based on task keywords:

```yaml
name: firebase-auth
description: Use PROACTIVELY when implementing Firebase authentication, signup, login, or auth flows
```

### 3. Quality Gate Hook (Priority: MEDIUM)

Implement Stop hook that ensures quality before completion:

```python
def validate_before_stop():
    if not all_tests_passing():
        return {"decision": "block", "reason": "Tests are failing"}
    if linter_has_errors():
        return {"decision": "block", "reason": "Linter errors present"}
    if requirements_not_met():
        return {"decision": "block", "reason": "Requirements not satisfied"}
    return {"continue": True}
```

### 4. Context Injection Hook (Priority: MEDIUM)

Implement SessionStart hook for loading relevant context:

```python
def inject_context():
    git_status = run_command("git status")
    recent_patterns = load_from_knowledge_base()
    project_prefs = load_preferences()

    return {
        "hookSpecificOutput": {
            "additionalContext": f"Git Status:\n{git_status}\n\nRecent Patterns:\n{recent_patterns}"
        }
    }
```

### 5. Pattern Capture Hook (Priority: MEDIUM)

Implement PostToolUse hook for capturing successful patterns:

```python
def capture_success_pattern():
    if tool_name in ["Edit", "Write"] and validation_passed():
        pattern = extract_pattern(tool_input, tool_response)
        save_to_knowledge_base(pattern)
```

## Additional Considerations Beyond Initial Plan

While researching, I identified several capabilities not explicitly in the initial design that could significantly enhance the system:

### 1. **Checkpoint System for Experimentation**

Claude Code 2.0 introduced automatic checkpointing that could support the experimental framework:
- Checkpoints created per user prompt
- Fast rewind to previous states
- Enables A/B testing with state restoration

**Recommendation**: Integrate checkpointing into experimental runner for fast rollback.

### 2. **Multi-Claude Parallel Workflows**

Pattern of running multiple Claude instances simultaneously:
- One writes code, another reviews
- Parallel worktrees for independent tasks
- Fan-out pattern for large migrations

**Recommendation**: Experimental framework could run control/treatment in parallel Claude instances.

### 3. **Stream-JSON Output Format**

Claude Code supports `--output-format stream-json` for programmatic use:
- Enables chaining Claude tasks
- Allows parsing of structured output
- Supports headless automation

**Recommendation**: Experimental runner should use stream-json format for parsing results.

### 4. **Plugin System for Distribution**

Claude Code plugins can bundle hooks, agents, and skills:
- Distributed as NPM packages
- Merged into user configuration
- Uses `${CLAUDE_PLUGIN_ROOT}` for file references

**Recommendation**: Package optimized-ai as a Claude Code plugin for easy installation.

### 5. **Extended Thinking Modes**

Claude Code supports "think", "think harder", and "ultrathink" for complex problems:
- Triggers deeper reasoning
- Improves solution quality
- Useful for planning phase

**Recommendation**: Planner subagent should use extended thinking by default.

### 6. **Visual Iteration Support**

Claude Code accepts screenshots and images directly:
- Paste screenshots (cmd+ctrl+shift+4, ctrl+v on macOS)
- Drag-and-drop images
- Provide image file paths

**Recommendation**: Could support visual validation in experimental framework.

### 7. **Git Worktree Pattern**

Lightweight alternative to multiple clones:
```bash
git worktree add ../project-feature-a feature-a
git worktree remove ../project-feature-a
```

**Recommendation**: Experimental runner could use worktrees for parallel experiment execution.

## Next Steps

### Phase 0 Implementation Priorities

Based on this research, here are recommended priorities for Phase 0 (Experimental Framework):

1. **Configure basic hooks** for experimental runner:
   - PostToolUse: Capture metrics (time, tokens, tool calls)
   - Stop: Validate experiment completion
   - SessionStart: Initialize experiment environment

2. **Design experiment scenarios** that test:
   - With/without skills loading
   - With/without subagents
   - Different context management strategies
   - Different hook configurations

3. **Use stream-json format** for experimental runner to parse Claude Code output programmatically

4. **Leverage checkpoints** for A/B testing with state restoration

5. **Consider git worktrees** for parallel experiment execution

### Future Phase Implementations

1. **Phase 1 (Minimal Core)**:
   - Implement core hooks (spin detection, quality gates)
   - Test minimal .cursorrules sizes
   - Validate each rule's contribution

2. **Phase 2 (Skills Architecture)**:
   - Implement progressive disclosure pattern
   - Create domain-specific skills
   - Test skill loading overhead

3. **Phase 3 (Subagent System)**:
   - Implement specialized subagents
   - Test context isolation benefits
   - Measure orchestration overhead

4. **Phase 4+ (Advanced Features)**:
   - MCP knowledge graph integration
   - Pattern learning from hooks
   - Self-optimization based on metrics

## References

### Official Documentation
- [Claude Code Hooks Reference](https://docs.claude.com/en/docs/claude-code/hooks)
- [Claude Code Subagents](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [MCP Examples](https://modelcontextprotocol.io/examples)

### Key Repositories
- [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery) - Comprehensive hook examples
- [claude-flow](https://github.com/ruvnet/claude-flow) - Agent orchestration platform
- [Claude Command Suite](https://github.com/qdhenry/Claude-Command-Suite) - 148+ slash commands
