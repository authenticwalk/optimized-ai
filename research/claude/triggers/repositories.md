# Notable Claude Code Repositories & Resources

A curated collection of repositories, tools, and resources for Claude Code triggers, hooks, and automation.

## Official Resources

### 1. **Anthropic Documentation**
- **Hooks Reference**: https://docs.claude.com/en/docs/claude-code/hooks
- **Subagents Guide**: https://docs.claude.com/en/docs/claude-code/sub-agents
- **Best Practices**: https://www.anthropic.com/engineering/claude-code-best-practices
- **Common Workflows**: https://docs.claude.com/en/docs/claude-code/common-workflows

### 2. **Model Context Protocol (MCP)**
- **Official Site**: https://modelcontextprotocol.io/
- **Specification**: https://spec.modelcontextprotocol.io/
- **Introduction**: https://modelcontextprotocol.io/introduction
- **Examples**: https://modelcontextprotocol.io/examples
- **GitHub**: https://github.com/modelcontextprotocol/servers

## Hooks & Automation

### 1. **claude-code-hooks-mastery** ⭐⭐⭐⭐⭐
**Repository**: https://github.com/disler/claude-code-hooks-mastery

**What it includes**:
- Complete examples of all 8 hook types
- UV single-file script patterns
- TTS notification system
- Security command blocking
- Automatic logging to JSON
- Chat transcript extraction
- Well-documented code

**Why it's valuable**:
- Production-ready patterns
- Comprehensive coverage
- Best practices demonstrated
- Easy to adapt for your project

**Key files to study**:
- `user_prompt_submit.py` - Context injection
- `pre_tool_use.py` - Security validation
- `post_tool_use.py` - Transcript extraction
- `stop.py` - Completion control
- `session_start.py` - Initialization

### 2. **claude-code-hooks-multi-agent-observability**
**Repository**: https://github.com/disler/claude-code-hooks-multi-agent-observability

**What it includes**:
- Real-time monitoring for Claude Code agents
- Hook event tracking
- Multi-agent orchestration patterns

**Use case**: Monitoring and debugging complex multi-agent workflows

### 3. **claude-code-guide**
**Repository**: https://github.com/zebbern/claude-code-guide

**What it includes**:
- Full guide on Claude tips and tricks
- Optimization techniques
- Hidden commands
- Best practices compilation

## Agent Orchestration

### 1. **claude-flow** ⭐⭐⭐⭐⭐
**Repository**: https://github.com/ruvnet/claude-flow

**What it is**:
The leading agent orchestration platform for Claude. Enterprise-grade architecture for deploying intelligent multi-agent swarms.

**Features**:
- Distributed swarm intelligence
- RAG integration
- Native Claude Code support via MCP
- 87 specialized MCP tools
- Multiple topology patterns (mesh, hierarchical, ring, star)
- Map-reduce patterns
- Stream-JSON chaining

**Key Documentation**:
- [Development Patterns](https://github.com/ruvnet/claude-flow/wiki/Development-Patterns)
- [Workflow Orchestration](https://github.com/ruvnet/claude-flow/wiki/Workflow-Orchestration)
- [Agent System Overview](https://github.com/ruvnet/claude-flow/wiki/Agent-System-Overview)
- [Agent Usage Guide](https://github.com/ruvnet/claude-flow/wiki/Agent-Usage-Guide)
- [MCP Tools](https://github.com/ruvnet/claude-flow/wiki/MCP-Tools)

**Best for**: Complex multi-agent coordination, enterprise projects

### 2. **agents** (wshobson)
**Repository**: https://github.com/wshobson/agents

**What it includes**:
- Intelligent automation
- Multi-agent orchestration
- Production-ready patterns

### 3. **claude-code-sub-agents** (lst97)
**Repository**: https://github.com/lst97/claude-code-sub-agents

**What it includes**:
- Specialized AI subagents for full-stack development
- Agent organizer patterns
- Personal development workflows

**Notable agent**: [Agent Organizer](https://github.com/lst97/claude-code-sub-agents/blob/main/agents/agent-organizer.md)

## Slash Commands & Workflows

### 1. **Claude-Command-Suite** ⭐⭐⭐⭐⭐
**Repository**: https://github.com/qdhenry/Claude-Command-Suite

**What it includes**:
- 148+ slash commands
- 54 AI agents
- Claude Code Skills
- Automated workflows

**Categories**:
- Code review workflows
- Testing automation
- Deployment pipelines
- Business scenario modeling
- GitHub-Linear synchronization

**Best for**: Comprehensive workflow automation

### 2. **commands** (wshobson)
**Repository**: https://github.com/wshobson/commands

**What it includes**:
Production-ready slash command collections:
- claude-code-essentials
- full-stack-development
- security-hardening
- data-ml-pipeline
- infrastructure-devops

**Best for**: Enterprise development workflows

### 3. **awesome-claude-code** ⭐⭐⭐⭐
**Repository**: https://github.com/hesreallyhim/awesome-claude-code

**What it is**:
Curated list of commands, files, and workflows for Claude Code

**Includes**:
- Slash commands
- CLAUDE.md examples
- CLI tools
- Workflow patterns
- GitHub Actions integration
- Community resources

**Notable workflows**:
- Design review automation
- Laravel TALL stack development
- Project bootstrapping

### 4. **claude-code** (PaulDuvall)
**Repository**: https://github.com/PaulDuvall/claude-code/

**What it includes**:
- Systematized AI development patterns
- Concrete Claude Code configurations
- Custom slash commands
- Hook implementations
- Documentation-driven development

## Skills & Infrastructure

### 1. **claude-code-infrastructure-showcase**
**Repository**: https://github.com/diet103/claude-code-infrastructure-showcase

**What it includes**:
- Examples of Claude Code infrastructure
- Skill auto-activation patterns
- Hook implementations
- Agent coordination

**Best for**: Understanding skill loading and infrastructure setup

## MCP Server Implementations

### 1. **Official MCP Servers** ⭐⭐⭐⭐⭐
**Repository**: https://github.com/modelcontextprotocol/servers

**What it includes**:
Reference implementations for MCP, including:

**Official Servers**:
- Everything - Reference/test server with prompts, resources, tools
- Fetch - Web content fetching
- Filesystem - Secure file operations
- Git - Repository management
- Memory - Knowledge graph-based persistent memory
- Sequential Thinking - Problem-solving through thought sequences

**Community Servers** (1000+):
- Slack, GitHub, Google Drive
- Postgres, MongoDB, Redis
- Puppeteer, Playwright
- Stripe, AWS, Azure
- Email, calendar integrations
- And many more...

**Best for**: Learning MCP server patterns, reference implementations

### 2. **mcp-knowledge-graph**
**Repository**: https://github.com/shaneholloman/mcp-knowledge-graph

**What it includes**:
- Persistent memory for Claude
- Local knowledge graph implementation
- Entity-relation-observation model
- JSONL storage patterns

**Best for**: Implementing persistent memory systems

### 3. **MemoryMesh**
**Repository**: https://github.com/CheMiguel23/MemoryMesh

**What it is**:
Knowledge graph server using MCP for structured memory persistence

**Version**: v0.2.8
**Features**: SQLite-based knowledge graphs, entity management

### 4. **mcp-prompts**
**Repository**: https://github.com/sparesparrow/mcp-prompts

**What it includes**:
- MCP server for managing prompts
- Prompt template storage
- LLM interaction templates

### 5. **model-context-protocol-resources**
**Repository**: https://github.com/cyanheads/model-context-protocol-resources

**What it includes**:
- [MCP Server Development Guide](https://github.com/cyanheads/model-context-protocol-resources/blob/main/guides/mcp-server-development-guide.md)
- Best practices
- Implementation patterns

## Testing & Automation

### 1. **Claude-Autopilot**
**Repository**: https://github.com/benbasha/Claude-Autopilot

**What it is**:
VS Code/Cursor extension for automating Claude Code tasks

**Features**:
- Intelligent queuing
- Batch processing
- Auto-resume on limits
- Comprehensive error handling
- Health monitoring
- Automatic retry with exponential backoff

**Best for**: Large-scale automation, batch processing

### 2. **Claude-SPARC Automated Development System**
**Gist**: https://gist.github.com/ruvnet/e8bb444c6149e6e060a785d1a693a194

**What it includes**:
- Agentic workflow for automated software development
- SPARC methodology implementation
- Claude Code CLI integration

## Guides & Tutorials

### 1. **ClaudeLog** ⭐⭐⭐⭐
**Website**: https://claudelog.com/

**Sections**:
- [Hooks Mechanics](https://claudelog.com/mechanics/hooks/)
- [Subagent Delegation](https://claudelog.com/faqs/what-is-sub-agent-delegation-in-claude-code/)
- Best practices
- FAQs

**Best for**: Conceptual understanding, quick reference

### 2. **eesel AI Blog**
**Articles**:
- [Hooks in Claude Code Guide](https://www.eesel.ai/blog/hooks-in-claude-code)
- [Claude Code Slash Commands](https://www.eesel.ai/blog/claude-code-slash-commands)
- [Workflow Automation](https://www.eesel.ai/blog/claude-code-workflow-automation)

**Best for**: Practical guides with examples

### 3. **GitButler Blog**
**Articles**:
- [Automate Your AI Workflows with Claude Code Hooks](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)

**Includes**:
- Desktop notification pattern
- Auto-commit workflow
- Multi-session isolation
- Ruby script examples

### 4. **Medium Articles**

**Skills Deep Dive**:
- [Claude Skills: A Technical Deep-Dive](https://medium.com/data-science-collective/claude-skills-a-technical-deep-dive-into-context-injection-architecture-ee6bf30cf514)
- [Claude Agent Skills: First Principles](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)

**Workflow Guides**:
- [I Mastered the Claude Code Workflow](https://medium.com/@ashleyha/i-mastered-the-claude-code-workflow-145d25e502cf)
- [How I Escaped the AI Loop of Death](https://medium.com/coding-nexus/how-i-escaped-the-ai-loop-of-death-a-4-rule-system-to-supercharge-your-coding-with-cursor-and-493b7f35a038)

### 5. **Complete Guides**

**Tyler Folkman's Substack**:
- [Complete Guide to Claude Skills](https://tylerfolkman.substack.com/p/the-complete-guide-to-claude-skills)

**Sid Bharath's Blog**:
- [Cooking with Claude Code: Complete Guide](https://www.siddharthbharath.com/claude-code-the-complete-guide/)

**alexop.dev**:
- [Claude Code Slash Commands: Boost Productivity](https://alexop.dev/tils/claude-code-slash-commands-boost-productivity/)

## Specialized Use Cases

### 1. **Firebase/Supabase**
**Search**: "claude code firebase examples repository"
**Find**: Community examples of Firebase/Supabase integration with Claude Code

### 2. **Security**

**Superagent + Claude Code Hooks**:
- [Documentation](https://docs.superagent.sh/examples/claude-code-userprompt)
- Security guardrails via UserPromptSubmit hooks

### 3. **Multi-Model Integration**

**Gemini Context Hook**:
- [GreenFlux Blog](https://blog.greenflux.us/claude-code-hook-to-ask-gemini-for-help/)
- Using hooks to integrate Gemini for additional context

## Learning Resources

### Official Docs
- **Anthropic Engineering Blog**: High-level best practices
- **Claude Docs**: Complete reference documentation
- **MCP Docs**: Protocol specification and guides

### Community Blogs
- **ClaudeLog**: Mechanics and FAQs
- **eesel AI**: Practical guides
- **GitButler**: Integration patterns
- **Medium**: Deep dives and case studies

### Video/Interactive
- **DEV Community**: Tutorial series
- **GitHub Discussions**: Community Q&A
- **Discord/Slack**: Real-time community help

## Tool Comparison

### Hooks vs Subagents vs Skills

| Feature | Hooks | Subagents | Skills |
|---------|-------|-----------|--------|
| **Purpose** | Automation, validation | Task delegation | Knowledge injection |
| **When** | Lifecycle events | Explicit/auto delegation | Auto-loaded on need |
| **Context** | Main session | Isolated context | Injected into context |
| **Language** | Any (shell scripts) | Natural language | Markdown |
| **Best for** | Deterministic control | Complex subtasks | Domain expertise |

### Use Together

**Example workflow**:
1. **SessionStart hook**: Load project context
2. **Skills**: Auto-load firebase-auth skill
3. **UserPromptSubmit hook**: Validate prompt, add context
4. **Subagent**: Planner creates implementation plan
5. **PreToolUse hook**: Block dangerous commands
6. **PostToolUse hook**: Run tests after code changes
7. **Subagent**: Code reviewer validates quality
8. **Stop hook**: Ensure all requirements met
9. **SessionEnd hook**: Archive transcript, cleanup

## Recommended Starting Points

### For Beginners
1. Read [Official Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
2. Study [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery) examples
3. Try simple hooks (notification, logging)
4. Experiment with basic subagents

### For Intermediate
1. Implement security hooks (PreToolUse blocking)
2. Create custom subagents for your domain
3. Build skill library for your tech stack
4. Set up MCP memory server
5. Automate testing with PostToolUse hooks

### For Advanced
1. Multi-agent orchestration with [claude-flow](https://github.com/ruvnet/claude-flow)
2. Custom MCP servers for your tools
3. Complex hook coordination patterns
4. Advanced skill progressive loading
5. Full CI/CD automation with hooks

## Community

### Where to Find Help
- **GitHub Discussions**: On official repos
- **Discord**: Claude/Anthropic community servers
- **Reddit**: r/ClaudeCode (check if exists)
- **Stack Overflow**: Tag: claude-code

### Contributing Back
- Share your hooks on GitHub
- Document your patterns
- Write blog posts about learnings
- Contribute to awesome-claude-code
- Help others in discussions

## Quick Reference Links

### Must-Read
- [Hooks Reference](https://docs.claude.com/en/docs/claude-code/hooks)
- [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- [Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)

### Inspiration
- [Claude Command Suite](https://github.com/qdhenry/Claude-Command-Suite)
- [claude-flow](https://github.com/ruvnet/claude-flow)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)

### Deep Dives
- [Skills Technical Deep-Dive](https://medium.com/data-science-collective/claude-skills-a-technical-deep-dive-into-context-injection-architecture-ee6bf30cf514)
- [MCP Server Development Guide](https://github.com/cyanheads/model-context-protocol-resources/blob/main/guides/mcp-server-development-guide.md)

---

**Last Updated**: 2025-11-03
**Compiled by**: Claude (Sonnet 4.5)
