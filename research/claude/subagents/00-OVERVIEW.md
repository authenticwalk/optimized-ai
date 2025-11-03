# Claude Agents & Subagents - Research Overview

**Research Date**: November 3, 2025
**Purpose**: Inform the design and implementation of the Optimized AI system
**Focus**: Agents, subagents, skills, MCP, token optimization, and multi-agent orchestration

---

## Table of Contents

1. [00-OVERVIEW.md](00-OVERVIEW.md) - This file
2. [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Comprehensive subagent documentation
3. [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md) - Skills system and on-demand loading
4. [03-MODEL-CONTEXT-PROTOCOL.md](03-MODEL-CONTEXT-PROTOCOL.md) - MCP specification and implementation
5. [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md) - Token efficiency techniques
6. [05-ORCHESTRATION-PATTERNS.md](05-ORCHESTRATION-PATTERNS.md) - Multi-agent coordination patterns
7. [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Comprehensive best practices
8. [07-GITHUB-RESOURCES.md](07-GITHUB-RESOURCES.md) - Repository links and examples
9. [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md) - Code examples and snippets
10. [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md) - How this applies to Optimized AI

---

## Executive Summary

### What We Learned

**Claude Subagents** are specialized AI instances with isolated contexts that enable:
- **Parallelization**: Multiple agents working simultaneously on different tasks
- **Context Isolation**: Fresh context windows prevent pollution
- **Tool Restriction**: Limited tool access for security and focus
- **Specialization**: Domain-specific expertise and instructions

**Agent Skills** are on-demand loadable instruction packages that:
- Use **progressive disclosure** (30-50 tokens until activated)
- Load only when relevant to current task
- Include instructions, scripts, and resources
- Enable massive context savings (skills vs monolithic prompts)

**Model Context Protocol (MCP)** standardizes:
- Connection between AI assistants and data sources
- Tool and resource exposure to agents
- Integration with external systems (Slack, GitHub, etc.)
- Reusable server implementations

### Key Performance Metrics

From Anthropic's research system:
- **Multi-agent systems outperformed single-agent by 90.2%**
- Token usage explains **80% of performance variance**
- Agents use **~4× more tokens** than chat
- Multi-agent systems use **~15× more tokens** than chat
- **Parallel tool calling** reduces research time by up to 90%

### Core Principles Validated

Our project's MINIMIZE and SEPARATE principles are strongly supported:

✅ **MINIMIZE**: On-demand loading proves essential
- Skills: 30-50 tokens vs thousands for monolithic prompts
- Progressive disclosure: Load only what's needed when needed
- Token efficiency directly correlates with performance

✅ **SEPARATE**: Context isolation delivers measurable benefits
- Subagents prevent context pollution
- Fresh contexts enable better focus
- Parallel execution accelerates complex tasks
- Clear boundaries reduce confusion

---

## Critical Insights for Optimized AI

### 1. Skills Architecture is Proven

Anthropic's Skills system validates our planned architecture:
- **Small core** (SKILL.md with metadata) + **on-demand resources**
- **Progressive disclosure** in 3 tiers: metadata → full skill → referenced files
- **Context efficiency**: Only load what's needed for current task
- **Composable**: Multiple skills work together automatically

**Direct Application**: Our planned skills/ directory structure is exactly right.

### 2. Subagents for Complex Tasks Only

Anthropic's guidance:
- Use for **breadth-first queries** requiring parallel exploration
- Use when **high-value tasks** justify 15× token overhead
- Use for **heavily parallelizable work**
- **Avoid** for tasks requiring shared context or heavy coordination

**Direct Application**: Our Phase 3 plan to add subagents after validating core/skills is correct sequencing.

### 3. Orchestrator-Worker Pattern is Standard

Every successful multi-agent system uses:
- **Lead/Orchestrator**: Coordinates, maintains global context, synthesizes results
- **Specialized Workers**: Execute specific subtasks with fresh contexts
- **Lightweight Communication**: Workers return summaries, not full context
- **Clear Task Decomposition**: Precise objectives, formats, boundaries

**Direct Application**: Our planner/implementer/reviewer agent structure aligns with proven patterns.

### 4. Token Optimization is Paramount

Key techniques we must implement:
- **Prompt caching**: Move unchanging content to top (75% cheaper)
- **Tool filtering**: Only include relevant tools in context
- **Output token management**: Each output token costs ~4× input
- **Context compaction**: Summarize history, maintain lightweight state
- **Just-in-time context**: Retrieve on-demand vs pre-loading

**Direct Application**: Phase 0 experiments must measure token usage rigorously.

### 5. Experimental Validation Confirmed

Anthropic's approach mirrors our Phase 0 plan:
- Start with small test samples (20 queries) early
- Use statistical analysis and LLM judges
- Measure token usage, tool calls, quality
- Iterate based on data
- Track what works and doesn't

**Direct Application**: Our experimental framework design is industry-standard.

---

## Architecture Alignment

### Our Planned System → Anthropic's Patterns

| Our Design | Anthropic Pattern | Status |
|------------|------------------|--------|
| Core .cursorrules (50 lines) | Minimal system prompt | ✅ Validated |
| Skills (load on-demand) | Agent Skills progressive disclosure | ✅ Validated |
| Subagents (planner/implementer/reviewer) | Orchestrator-worker pattern | ✅ Validated |
| MCP skills servers | MCP standard | ✅ Validated |
| Knowledge base (.ai-knowledge/) | Not directly addressed | ⚠️ Novel approach |
| Spin detection | Not directly addressed | ⚠️ Novel approach |
| Experimental framework | Industry standard | ✅ Validated |

### What We're Doing Right

1. **Minimal core + on-demand loading**: Exactly matches Skills architecture
2. **Subagent structure**: Aligns with proven orchestrator-worker pattern
3. **MCP integration**: Standard approach for tools and resources
4. **Token obsession**: Confirmed as primary performance driver
5. **Experimental validation**: Matches Anthropic's development approach

### What We Need to Add/Refine

1. **Progressive disclosure**: Implement 3-tier loading for skills
2. **Prompt caching**: Design prompts with caching in mind
3. **Tool filtering**: Dynamically filter tools based on context
4. **Context compaction**: Add summarization for long tasks
5. **Parallel tool calling**: Enable where beneficial
6. **Statistical significance**: Use proper A/B testing methodology

### Novel Contributions

These aspects aren't well-covered in existing documentation:

1. **Self-learning knowledge base**: Our .ai-knowledge/ approach
2. **Spin detection system**: Automated detection and recovery
3. **Workspace management**: .plan/ folder workflow
4. **Continuous optimization**: Learning from usage patterns
5. **IDE integration via MCP**: Using MCP for IDE operations

---

## Research Sources

### Official Documentation
- [Claude Subagents Docs](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Agent SDK Subagents](https://docs.claude.com/en/api/agent-sdk/subagents)
- [Agent Skills Docs](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/overview)
- [MCP Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)

### Anthropic Engineering Blog
- [Building Agents with Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)
- [Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Equipping Agents with Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

### GitHub Repositories
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) - 100+ specialized agents
- [wshobson/agents](https://github.com/wshobson/agents) - Production-ready orchestration
- [avivl/claude-007-agents](https://github.com/avivl/claude-007-agents) - Unified orchestration system
- [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) - Official MCP SDK

### Community Resources
- Multiple Medium articles on implementation patterns
- DEV.to tutorials on multi-agent orchestration
- InfoQ, WinBuzzer coverage of Claude Code updates
- Various blog posts with real-world implementations

---

## Next Steps

1. **Read detailed documents** in this research folder (01-09)
2. **Review implementation examples** (08-IMPLEMENTATION-EXAMPLES.md)
3. **Check project alignment** (09-PROJECT-ALIGNMENT.md)
4. **Update Phase 0 plan** based on findings
5. **Refine skill definitions** using progressive disclosure
6. **Enhance subagent specs** with orchestration patterns
7. **Add token optimization** experiments to Phase 0

---

## Key Takeaways

### For Phase 0 (Experimental Framework)
- ✅ Our approach is validated by industry leaders
- ✅ Add token usage as primary metric
- ✅ Include A/B testing for statistical significance
- ✅ Measure context pollution and skill loading efficiency

### For Phase 1 (Minimal Core)
- ✅ Keep .cursorrules < 50 lines
- ✅ Design for prompt caching (static content at top)
- ✅ Implement tool filtering
- ✅ Progressive disclosure architecture

### For Phase 2 (Skills Architecture)
- ✅ Use SKILL.md + YAML frontmatter
- ✅ Implement 3-tier progressive disclosure
- ✅ Load skills on-demand (30-50 token metadata)
- ✅ Split large skills into referenced files
- ✅ Make skills composable

### For Phase 3 (Subagents)
- ✅ Use orchestrator-worker pattern
- ✅ Implement for complex, parallelizable tasks only
- ✅ Return lightweight summaries (not full context)
- ✅ Clear task decomposition with precise boundaries
- ✅ Monitor 15× token overhead

---

**This research comprehensively validates our architectural approach while providing concrete implementation guidance from Anthropic and the community.**
