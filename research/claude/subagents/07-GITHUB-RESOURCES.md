# GitHub Resources & Repositories

**Last Updated**: November 3, 2025

---

## Official Anthropic Resources

### 1. Claude Agent SDK (Python)
**Repository**: [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python)

**Description**: Official Python SDK for building agents with Claude

**Key Features**:
- Subagent support built-in
- Examples in `examples/` directory
- Quick start guide
- Streaming mode support

**Installation**:
```bash
pip install claude-agent-sdk
```

**Example**: `examples/quick_start.py`, `examples/streaming_mode.py`

---

### 2. Model Context Protocol (TypeScript SDK)
**Repository**: [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)

**Description**: Official TypeScript SDK for building MCP servers and clients

**Key Features**:
- Create MCP servers
- Create MCP clients
- Full protocol implementation
- Type-safe

**Installation**:
```bash
npm install @modelcontextprotocol/sdk
```

**Quick Start**:
```bash
npx @modelcontextprotocol/create-server my-server
```

---

### 3. MCP Server Template
**Repository**: [modelcontextprotocol/create-typescript-server](https://github.com/modelcontextprotocol/create-typescript-server)

**Description**: CLI tool to create new TypeScript MCP servers

**Usage**:
```bash
npx @modelcontextprotocol/create-server my-server
cd my-server
npm install
npm run build
```

---

## Official MCP Servers

Anthropic provides pre-built MCP servers:

1. **Google Drive**: `@modelcontextprotocol/server-google-drive`
2. **Slack**: `@modelcontextprotocol/server-slack`
3. **GitHub**: `@modelcontextprotocol/server-github`
4. **Git**: `@modelcontextprotocol/server-git`
5. **Postgres**: `@modelcontextprotocol/server-postgres`
6. **Puppeteer**: `@modelcontextprotocol/server-puppeteer`

**Installation Example**:
```bash
npm install @modelcontextprotocol/server-github
```

---

## Community Agent Collections

### 1. VoltAgent/awesome-claude-code-subagents ⭐ RECOMMENDED
**Repository**: [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)

**Description**: Production-ready Claude subagents collection with 100+ specialized AI agents

**Categories**:
- Development & Architecture
- Infrastructure & Operations
- Data & Analytics
- Business & Marketing
- Content & Documentation
- Security & Compliance

**Notable Agents**:
- `full-stack-architect.md` - System design
- `security-auditor.md` - Security reviews
- `performance-optimizer.md` - Performance tuning
- `test-engineer.md` - Test creation
- `api-designer.md` - API design

**Usage**:
```bash
# Download agent
curl -o .claude/agents/security-auditor.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/agents/security-auditor.md
```

---

### 2. wshobson/agents
**Repository**: [wshobson/agents](https://github.com/wshobson/agents)

**Description**: Intelligent automation and multi-agent orchestration for Claude Code

**Key Features**:
- 85 specialized AI agents
- 15 multi-agent workflow orchestrators
- 47 agent skills
- 44 development tools
- 63 focused plugins

**Categories**:
- Code agents (review, refactor, test)
- Project management agents
- DevOps agents
- Documentation agents
- Workflow orchestrators

**Highlights**:
- Production-ready collection
- Well-documented agents
- Orchestration examples
- Real-world workflows

---

### 3. avivl/claude-007-agents
**Repository**: [avivl/claude-007-agents](https://github.com/avivl/claude-007-agents)

**Description**: Unified AI agent orchestration system with advanced coordination

**Key Features**:
- 10+ specialized agents across 14 categories
- Advanced coordination intelligence
- Resilience engineering
- Structured logging

**Categories**:
- Architecture
- Development
- Testing
- DevOps
- Security
- Documentation
- Project Management

**Notable Features**:
- Hierarchical coordination
- Error recovery
- Progress tracking
- Integration examples

---

### 4. zhsama/claude-sub-agent
**Repository**: [zhsama/claude-sub-agent](https://github.com/zhsama/claude-sub-agent)

**Description**: AI-driven development workflow system

**Key Features**:
- Complete development workflow
- Phase-based implementation
- Specialized agents for each phase
- Real-world project examples

**Workflow Phases**:
1. Planning
2. Architecture
3. Implementation
4. Testing
5. Deployment

---

### 5. rahulvrane/awesome-claude-agents
**Repository**: [rahulvrane/awesome-claude-agents](https://github.com/rahulvrane/awesome-claude-agents)

**Description**: Collection of awesome Claude Code subagents

**Key Features**:
- Curated agent list
- Community contributions
- Various domains
- Regular updates

---

### 6. pjt222/claude-code-agents
**Repository**: [pjt222/claude-code-agents](https://github.com/pjt222/claude-code-agents)

**Description**: Comprehensive collection with security focus

**Key Features**:
- Standardized YAML frontmatter
- MCP integration
- Defensive security focus
- Production patterns

**Notable Agents**:
- Security-focused agents
- Compliance agents
- Audit agents

---

## Agent Skills Collections

### 1. travisvn/awesome-claude-skills
**Repository**: [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)

**Description**: Curated list of Claude Skills

**Key Features**:
- Skills for various domains
- Resources and tools
- Customization guides
- Best practices

**Categories**:
- Development
- Data Science
- DevOps
- Business
- Creative

---

## Example Implementations

### 1. Multi-Agent Orchestration Examples

**Repository**: Multiple community examples

**Key Patterns**:

#### Pattern 1: Code Review Pipeline
```
Orchestrator → Security Reviewer
             → Quality Reviewer
             → Performance Reviewer
             → Synthesize results
```

**Example Repos**:
- wshobson/agents - See `workflows/code-review.md`
- avivl/claude-007-agents - See `examples/review-pipeline`

#### Pattern 2: Feature Development
```
Orchestrator → Planner
             → Implementer
             → Tester
             → Documenter
```

**Example Repos**:
- zhsama/claude-sub-agent - Full workflow example
- wshobson/agents - See `workflows/feature-development.md`

---

## MCP Server Examples

### 1. Minimal MCP Server Template
**Repository**: [hypermodel-labs/mcp-server-template](https://github.com/jatinsandilya/mcp-server-template)

**Description**: Minimal TypeScript template for MCP servers

**Features**:
- Basic MCP server setup
- Sample tool implementation
- Ready-to-use structure

---

### 2. FastMCP Framework
**Repository**: [punkpeye/fastmcp](https://github.com/punkpeye/fastmcp)

**Description**: TypeScript framework for building MCP servers

**Features**:
- Higher-level abstraction
- Built on official SDK
- Faster development
- Good documentation

**Installation**:
```bash
npm install fastmcp
```

---

### 3. MCP Streamable HTTP Example
**Repository**: [invariantlabs-ai/mcp-streamable-http](https://github.com/invariantlabs-ai/mcp-streamable-http)

**Description**: Example MCP implementation with HTTP transport

**Features**:
- Python and TypeScript examples
- HTTP transport
- Streaming support

---

## Orchestration Frameworks

### 1. ruvnet/claude-flow
**Repository**: [ruvnet/claude-flow](https://github.com/ruvnet/claude-flow)

**Description**: Leading agent orchestration platform for Claude

**Key Features**:
- Enterprise-grade architecture
- Distributed swarm intelligence
- RAG integration
- Native Claude Code support via MCP
- Ranked #1 in agent-based frameworks

**Capabilities**:
- Multi-agent swarms
- Autonomous workflows
- Conversational AI systems
- Complex orchestration

**Use Cases**:
- Large-scale agent coordination
- Production deployments
- Enterprise applications

---

## Useful Searches

### Finding More Resources

**GitHub Topics**:
- [claude-agents](https://github.com/topics/claude-agents)
- [claude-code](https://github.com/topics/claude-code)
- [mcp-server](https://github.com/topics/mcp-server)
- [anthropic](https://github.com/topics/anthropic)

**Search Queries**:
```
"claude code" subagents site:github.com
"claude agent" implementation site:github.com
"mcp server" typescript site:github.com
anthropic claude agents site:github.com
```

---

## Community Resources

### Documentation Sites

1. **ClaudeLog**: [claudelog.com](https://claudelog.com)
   - Claude Code docs, guides, tutorials
   - Best practices
   - FAQs

2. **Cursor IDE Blog**: [cursor-ide.com/blog](https://www.cursor-ide.com/blog)
   - Multi-agent systems guides
   - Implementation patterns

---

## Example Agent Files

### Security Reviewer

From VoltAgent/awesome-claude-code-subagents:

```markdown
---
name: security-auditor
description: Expert security auditor for identifying vulnerabilities
tools: Read, Grep, Glob
model: claude-3-5-sonnet-20241022
---

# Security Auditor

You are a security expert specializing in web application security.

## Focus Areas
- SQL injection
- XSS (Cross-site scripting)
- CSRF
- Authentication bypasses
- Authorization issues
- Secrets management

## Process
1. Analyze code for security vulnerabilities
2. Assess severity (Critical/High/Medium/Low)
3. Provide specific remediation
4. Reference OWASP guidelines

## Output Format
- File and line number
- Vulnerability type
- Severity
- Remediation steps
- Code example (if applicable)
```

### Test Engineer

From wshobson/agents:

```markdown
---
name: test-engineer
description: Creates comprehensive tests for code
tools: Read, Write, Bash, Grep
model: claude-3-5-sonnet-20241022
---

# Test Engineer

You create comprehensive tests for code.

## Types of Tests
- Unit tests
- Integration tests
- Edge case tests

## Process
1. Analyze code to test
2. Identify edge cases
3. Write test cases
4. Run tests and verify

## Edge Cases to Consider
- Null/undefined inputs
- Empty arrays/objects
- Boundary values
- Invalid types
- Error conditions
```

---

## How to Use These Resources

### 1. Browse Collections
Visit awesome-claude-code-subagents for pre-built agents

### 2. Copy Agent Files
Download agent .md files to `.claude/agents/`

### 3. Customize
Modify prompts, tools, focus areas for your needs

### 4. Create Skills
Use agent patterns to create project-specific skills

### 5. Build MCP Servers
Use templates to create custom MCP servers for your skills

### 6. Study Orchestration
Review workflow examples from wshobson/agents and avivl/claude-007-agents

---

## Recommended Starting Point

**For Beginners**:
1. Start with VoltAgent/awesome-claude-code-subagents
2. Download 3-5 agents that match your needs
3. Test them in Claude Code
4. Customize as needed

**For Intermediate**:
1. Explore wshobson/agents for orchestration examples
2. Study avivl/claude-007-agents for coordination patterns
3. Try creating your own agents

**For Advanced**:
1. Build custom MCP servers using templates
2. Implement multi-agent orchestration
3. Create domain-specific agent systems
4. Contribute back to community

---

## Key Takeaways

1. **100+ pre-built agents available** - Don't start from scratch
2. **Official SDKs** provide solid foundation
3. **Community is active** - Many examples and patterns
4. **MCP standardizes integrations** - Use pre-built servers
5. **Orchestration patterns** available for study
6. **Production-ready collections** exist (VoltAgent, wshobson)
7. **YAML frontmatter** is standard format
8. **Regular updates** - Community actively developing

---

## Further Reading

- [01-SUBAGENTS-DEEP-DIVE.md](01-SUBAGENTS-DEEP-DIVE.md) - Understanding subagents
- [02-AGENT-SKILLS-ARCHITECTURE.md](02-AGENT-SKILLS-ARCHITECTURE.md) - Skills vs Agents
- [08-IMPLEMENTATION-EXAMPLES.md](08-IMPLEMENTATION-EXAMPLES.md) - Code samples
- [09-PROJECT-ALIGNMENT.md](09-PROJECT-ALIGNMENT.md) - Applying to our project
