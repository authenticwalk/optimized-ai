# Claude Agents & Subagents Research

**Research Date**: November 3, 2025
**Purpose**: Comprehensive research on Claude agents, subagents, skills, MCP, and best practices to inform the Optimized AI project

---

## Quick Start

**New to this research?** Start here:

1. **[00-OVERVIEW.md](00-OVERVIEW.md)** - Executive summary and key findings (15 min read)
2. **[09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md)** - How this applies to our project (10 min read)
3. Dive deeper into specific topics as needed

---

## Document Index

### Core Documentation

| # | Document | Description | Time | Priority |
|---|----------|-------------|------|----------|
| 00 | [OVERVIEW](00-OVERVIEW.md) | Executive summary, key findings, architecture alignment | 15 min | 🔴 READ FIRST |
| 01 | [SUBAGENTS DEEP DIVE](01-SUBAGENTS-DEEP-DIVE.md) | Comprehensive guide to Claude subagents | 45 min | 🔴 Essential |
| 02 | [SKILLS ARCHITECTURE](02-AGENT-SKILLS-ARCHITECTURE.md) | Agent Skills and progressive disclosure | 35 min | 🔴 Essential |
| 03 | [MODEL CONTEXT PROTOCOL](03-MODEL-CONTEXT-PROTOCOL.md) | MCP specification and implementation | 30 min | 🟡 Important |
| 04 | [TOKEN OPTIMIZATION](04-TOKEN-OPTIMIZATION.md) | Token efficiency techniques | 25 min | 🔴 Essential |
| 05 | [ORCHESTRATION PATTERNS](05-ORCHESTRATION-PATTERNS.md) | Multi-agent coordination patterns | 30 min | 🟡 Important |
| 06 | [BEST PRACTICES](06-BEST-PRACTICES.md) | Comprehensive best practices guide | 35 min | 🔴 Essential |
| 07 | [GITHUB RESOURCES](07-GITHUB-RESOURCES.md) | Repository links and community resources | 20 min | 🟢 Reference |
| 08 | [IMPLEMENTATION EXAMPLES](08-IMPLEMENTATION-EXAMPLES.md) | Code examples and snippets | 30 min | 🟡 Important |
| 09 | [PROJECT ALIGNMENT](09-PROJECT-ALIGNMENT.md) | Applying research to Optimized AI | 25 min | 🔴 READ SECOND |
| 10 | [AGENT GRANULARITY & LIMITATIONS](10-AGENT-GRANULARITY-AND-LIMITATIONS.md) | Agent vs Skill decisions, nesting limitations, context pollution | 40 min | 🔴 CRITICAL |

**Total reading time**: ~5 hours for complete coverage

---

## Reading Paths

### Path 1: Executive Overview (60 minutes)

For quick understanding:

1. [00-OVERVIEW.md](00-OVERVIEW.md) - Key findings
2. [10-AGENT-GRANULARITY-AND-LIMITATIONS.md](10-AGENT-GRANULARITY-AND-LIMITATIONS.md) - Critical constraints
3. [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md) - Project application
4. [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Quick reference section

### Path 2: Implementation Focus (2.5 hours)

For developers ready to build:

1. [00-OVERVIEW.md](00-OVERVIEW.md) - Context
2. [10-AGENT-GRANULARITY-AND-LIMITATIONS.md](10-AGENT-GRANULARITY-AND-LIMITATIONS.md) - Critical design decisions
3. [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - How subagents work
4. [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md) - How skills work
5. [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md) - Code examples
6. [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md) - Do's and don'ts

### Path 3: Deep Technical Understanding (5 hours)

For comprehensive knowledge:

Read all documents in order (00-10)

### Path 4: Specific Topics

Pick documents based on your needs:

- **Token optimization?** → [04-TOKEN-OPTIMIZATION.md](04-TOKEN-OPTIMIZATION.md)
- **Multi-agent systems?** → [05-ORCHESTRATION-PATTERNS.md](05-ORCHESTRATION-PATTERNS.md)
- **MCP integration?** → [03-MODEL-CONTEXT-PROTOCOL.md](03-MODEL-CONTEXT-PROTOCOL.md)
- **Code examples?** → [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md)
- **Community resources?** → [07-GITHUB-RESOURCES.md](07-GITHUB-RESOURCES.md)

---

## Key Findings Summary

### Architecture Validation

✅ **Our approach is validated by Anthropic's production implementation**

| Our Design | Anthropic Pattern | Status |
|------------|------------------|--------|
| Minimal core (50 lines) | Minimal system prompt | ✅ Validated |
| Skills (on-demand) | Progressive disclosure | ✅ Validated |
| Subagents | Orchestrator-worker | ✅ Validated |
| MCP integration | MCP standard | ✅ Validated |
| Experimental validation | A/B testing | ✅ Validated |

### Performance Metrics

From Anthropic's research:

- **90.2% improvement**: Multi-agent vs single agent (complex tasks)
- **80% variance**: Token usage explains performance
- **60-70% savings**: Skills vs monolithic prompts
- **75% discount**: Prompt caching on static content
- **15× overhead**: Multi-agent token cost (use wisely)
- **90% time reduction**: Parallel execution benefit

### Core Principles Confirmed

1. **MINIMIZE**: Progressive disclosure saves 60-70% tokens
2. **SEPARATE**: Subagents improve results by 90.2%
3. **VALIDATE**: A/B testing is industry standard
4. **ITERATE**: Skills enable continuous optimization

---

## Critical Insights

### 0. ⚠️ CRITICAL LIMITATION: No Nested Subagents

**Subagents CANNOT call other subagents** - This is a hard constraint that shapes all design decisions.

```
❌ NOT POSSIBLE:
Main Agent → Subagent A → Subagent B (BLOCKED)

✅ ONLY VALID:
Main Agent → Subagent A
Main Agent → Subagent B
```

**Implication**: All designs must use flat hierarchy (max 2 levels)

**See**: [10-AGENT-GRANULARITY-AND-LIMITATIONS.md](10-AGENT-GRANULARITY-AND-LIMITATIONS.md) for complete analysis

### 1. Skills Architecture is Proven

**Three-tier progressive disclosure**:
- Tier 1: Metadata (5 tokens per skill)
- Tier 2: SKILL.md (~400 tokens when activated)
- Tier 3: Resources (~200 tokens each, as needed)

**Result**: 60-70% token savings vs monolithic prompts

### 2. Token Usage = Performance

**80% of performance variance** explained by token usage

**Optimization priority**:
1. Output tokens (4× cost)
2. Prompt caching (75% savings)
3. Progressive loading
4. Tool filtering

### 3. Multi-Agent for High-Value Tasks Only

**Cost**: 15× token overhead
**Benefit**: 90.2% better results
**Use for**: Complex, parallelizable, high-value tasks
**Don't use for**: Simple operations, shared context tasks

### 4. Orchestrator-Worker is Standard

**Every successful multi-agent system uses**:
- Orchestrator: Coordinates (doesn't execute)
- Workers: Execute in isolated contexts
- Communication: Lightweight summaries
- Execution: Parallel where possible

### 5. Implementation is Standardized

**YAML frontmatter** for filesystem agents:
```yaml
---
name: agent-name
description: When to use this agent
tools: Read, Grep, Glob
model: claude-3-5-sonnet-20241022
---
```

**MCP servers** for tools and resources:
```typescript
server.tool({ name, description, handler });
server.resource({ uri, mimeType, handler });
server.prompt({ name, template, parameters });
```

---

## Immediate Action Items

Based on research findings:

### For Phase 0 (Experimental Framework)

1. **Add prompt caching experiments**
   - Measure cache hit rates
   - Test static vs dynamic content placement

2. **Add progressive disclosure experiments**
   - Compare monolithic vs skills
   - Measure token savings and quality

3. **Add tool filtering experiments**
   - Test full vs filtered tool sets
   - Measure context reduction and success

4. **Track enhanced metrics**
   ```typescript
   {
     cachedTokens: number,
     skillsLoaded: string[],
     skillTokens: number,
     contextWindowUsage: number
   }
   ```

### For Phase 1 (Minimal Core)

1. **Structure .cursorrules for caching**
   - Static content at top
   - Dynamic content at bottom
   - Target: 75% cache hit rate

2. **Design minimal core**
   - Core principles (50 lines)
   - Skill metadata (50 tokens)
   - Total baseline: 250 tokens

### For Phase 2 (Skills)

1. **Implement 3-tier progressive disclosure**
   - Tier 1: Metadata
   - Tier 2: SKILL.md (lean, ~100 lines)
   - Tier 3: Resources (split into files)

2. **Create initial skills**
   - firebase-auth
   - supabase-rls
   - testing
   - refactoring

### For Phase 3 (Subagents)

1. **Define orchestrator-worker hierarchy**
   ```
   Orchestrator (Opus): project-coordinator
   Workers (Sonnet):
   - planner
   - implementer
   - security-reviewer
   - quality-reviewer
   - test-engineer
   ```

2. **Implement lightweight summaries**
   - Workers return 1-2k tokens
   - Not 50k token explorations

---

## Resources

### Official Documentation

- [Claude Subagents Docs](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Agent SDK Subagents](https://docs.claude.com/en/api/agent-sdk/subagents)
- [Agent Skills Docs](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/overview)
- [MCP Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)

### GitHub Resources

**Agent Collections**:
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) - 100+ agents
- [wshobson/agents](https://github.com/wshobson/agents) - Production-ready collection
- [avivl/claude-007-agents](https://github.com/avivl/claude-007-agents) - Orchestration system

**Official SDKs**:
- [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python)
- [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)

**MCP Servers**:
- [modelcontextprotocol](https://github.com/modelcontextprotocol) - Official MCP org

### Community Resources

- [ClaudeLog](https://claudelog.com) - Docs, guides, tutorials
- [Cursor IDE Blog](https://www.cursor-ide.com/blog) - Multi-agent guides
- GitHub Topics: [claude-agents](https://github.com/topics/claude-agents)

---

## Document Maintenance

### Last Updated
November 3, 2025

### Update Frequency
- Review quarterly or when major Claude updates released
- Add new examples as discovered
- Update metrics as Anthropic publishes new research

### Contributing
When adding to this research:
1. Update this README with new document
2. Cross-reference in related documents
3. Update reading paths if needed
4. Add to appropriate sections

---

## Questions?

### Where to start?
Read [00-OVERVIEW.md](00-OVERVIEW.md) first

### How does this apply to our project?
See [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md)

### Need code examples?
See [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md)

### Want best practices?
See [06-BEST-PRACTICES.md](06-BEST-PRACTICES.md)

### Looking for repos?
See [07-GITHUB-RESOURCES.md](07-GITHUB-RESOURCES.md)

---

## Quick Reference Card

**Key Metrics to Remember**:
- Skills: 60-70% token savings
- Multi-agent: 90.2% better results, 15× cost
- Prompt caching: 75% discount
- Token usage: 80% of performance variance
- Output tokens: 4× cost of input

**Critical Constraints**:
- ⚠️ **No nested subagents** - Flat hierarchy only
- ⚠️ **Main agent = Air traffic controller** - Never does context-heavy work
- ⚠️ **Most "agents" should be skills** - Only use agents for isolated, context-heavy work

**Core Patterns**:
- Progressive disclosure (3-tier)
- Orchestrator-worker (flat hierarchy)
- Map/reduce (subagents explore, main synthesizes)
- Lightweight summaries (1-2k tokens)
- Tool filtering (minimal set)
- Prompt caching (static at top)

**File Formats**:
- Agents: YAML frontmatter + Markdown
- Skills: SKILL.md + resource files
- MCP: TypeScript/Python servers

---

**This research comprehensively validates our Optimized AI architectural approach and provides concrete implementation guidance.**
