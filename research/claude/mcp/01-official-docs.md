# Official MCP Documentation & Resources

## 📖 Official Documentation

### Primary Resources

**Claude Documentation**
- URL: https://docs.claude.com/en/docs/mcp
- Description: Official Claude documentation for MCP integration
- Status: As of November 2025

**MCP Protocol Specification**
- URL: https://modelcontextprotocol.io/
- Description: Complete protocol specification and architecture
- GitHub: https://github.com/modelcontextprotocol

**Official Announcement**
- URL: https://www.anthropic.com/news/model-context-protocol
- Date: November 2024
- Description: Anthropic's official announcement open-sourcing MCP

### Learning Resources

**Anthropic Courses**
- URL: https://anthropic.skilljar.com/introduction-to-model-context-protocol
- Description: Official course teaching how to build MCP servers and clients from scratch using Python
- Topics Covered:
  - MCP's three core primitives (tools, resources, prompts)
  - Connecting Claude with external services
  - Building servers from scratch

**Microsoft MCP for Beginners**
- GitHub: https://github.com/microsoft/mcp-for-beginners
- Description: Open-source curriculum with real-world, cross-language examples
- Languages: .NET, Java, TypeScript, JavaScript, Rust, Python
- Format: 13-lab hands-on learning path
- Focus: Building production-ready MCP servers with PostgreSQL integration

## 🛠️ Official SDKs

### TypeScript SDK
- **Repository**: https://github.com/modelcontextprotocol/typescript-sdk
- **Package**: `@modelcontextprotocol/sdk`
- **Stars**: 10.6k
- **License**: MIT
- **Description**: Full MCP specification implementation for TypeScript/JavaScript
- **Install**: `npm install @modelcontextprotocol/sdk`

**Features:**
- Server and client implementations
- Support for tools, resources, and prompts
- Multiple transports (stdio, HTTP)
- Zod schema validation
- Notification debouncing
- Dynamic server capabilities
- Backwards compatibility

### Python SDK
- **Repository**: https://github.com/modelcontextprotocol/python-sdk
- **Package**: `mcp` (includes FastMCP)
- **Description**: Official Python SDK with high-level decorator framework
- **Install**: `pip install mcp` or `pip install fastmcp`

**Features:**
- FastMCP decorator-based framework
- Automatic schema generation from type hints
- Support for all three primitives
- Multiple transport options
- Testing utilities
- Production deployment tools

### Other Official SDKs

The MCP specification has implementations in:
- **Java/Kotlin**
- **C# (.NET)**
- **Ruby**
- **Swift**
- **Go**
- **Rust**
- **PHP**

All available at: https://github.com/modelcontextprotocol

## 📚 Reference Implementations

### Official MCP Servers Repository
- **URL**: https://github.com/modelcontextprotocol/servers
- **Description**: Collection of reference MCP server implementations

**Reference Servers Included:**
1. **Everything** - Reference/test server with prompts, resources, and tools
2. **Fetch** - Web content fetching and conversion for LLM usage
3. **Filesystem** - Secure file operations with configurable access controls
4. **Git** - Tools to read, search, and manipulate Git repositories
5. **Memory** - Knowledge graph-based persistent memory system
6. **Sequential Thinking** - Dynamic and reflective problem-solving
7. **Time** - Time and timezone conversion capabilities

### GitHub's Official MCP Server
- **Repository**: https://github.com/github/github-mcp-server
- **Description**: GitHub's official MCP server
- **Features** (as of October 2025):
  - Server instructions for guiding model behavior
  - Better tools for repository and code management
  - GitHub Projects support
  - Issue and PR management
  - Code analysis capabilities

## 🌐 Community Resources

### Awesome Lists

**awesome-mcp-servers** (Multiple Curators)
- https://github.com/punkpeye/awesome-mcp-servers
- https://github.com/wong2/awesome-mcp-servers
- https://github.com/appcypher/awesome-mcp-servers
- https://github.com/habitoai/awesome-mcp-servers
- https://github.com/TensorBlock/awesome-mcp-servers

Collections of 150+ MCP servers including:
- Cloud providers (AWS, Azure, Alibaba Cloud)
- SaaS platforms (Slack, Jira, Salesforce)
- Databases (PostgreSQL, Snowflake, Astra DB)
- AI/Analytics tools
- Payment systems

**awesome-mcp-clients**
- https://github.com/punkpeye/awesome-mcp-clients
- Collection of MCP client implementations

### Skills Resources

**Official Skills Repository**
- https://github.com/anthropics/skills
- Public repository for Claude Skills
- Examples and templates

**Community Skills**
- https://github.com/travisvn/awesome-claude-skills
- Curated list of awesome Claude Skills and resources
- https://github.com/mrgoonie/claudekit-skills
- ClaudeKit.cc powerful skills collection

## 📖 Documentation Sites

**MCP Directory & Blog**
- https://www.mcplist.ai/blog/
- Guides and comparisons for MCP implementations

**FastMCP Official Docs**
- https://gofastmcp.com/
- Comprehensive FastMCP documentation
- Patterns and best practices

**Model Context Protocol Info**
- https://modelcontextprotocol.info/
- Community documentation site
- Best practices and guides

## 🎓 Tutorials & Guides

### Official Guides
- **How to Build MCP Servers** - https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/
- **GitHub MCP Server Guide** - https://github.blog/ai-and-ml/generative-ai/a-practical-guide-on-how-to-use-the-github-mcp-server/
- **VS Code MCP Integration** - https://code.visualstudio.com/docs/copilot/chat/mcp-servers

### Third-Party Tutorials
- **Composio MCP Guide** - https://composio.dev/blog/cluade-code-with-mcp-is-all-you-need
- **DataCamp FastMCP Tutorial** - https://www.datacamp.com/tutorial/building-mcp-server-client-fastmcp
- **Scrapfly Python Guide** - https://scrapfly.io/blog/posts/how-to-build-an-mcp-server-in-python-a-complete-guide

## 🗞️ Recent Updates (2025)

### March 2025
- **OpenAI Adoption**: OpenAI officially adopted MCP across ChatGPT desktop app, Agents SDK, and Responses API

### October 2025
- **GitHub Projects Support**: GitHub MCP server added support for GitHub Projects
- **Server Instructions**: New MCP feature acting like system prompts for servers
- **Enhanced Tools**: Improved repository and code management tools

## 📝 Key Documentation Pages

### Core Concepts
- **Architecture**: https://modelcontextprotocol.io/specification/2025-06-18/architecture
- **Prompts**: https://modelcontextprotocol.io/docs/concepts/prompts
- **Best Practices**: https://modelcontextprotocol.info/docs/best-practices/

### Integration Guides
- **Claude.ai**: Use MCP connectors for teams
- **Claude Desktop**: Add MCP servers to desktop app
- **Claude Code**: Add MCP servers to coding environment
- **Messages API**: Use MCP connector in API calls

## 🔍 Additional Resources

### Blog Posts & Articles
- **Simon Willison**: "Claude Skills are awesome, maybe a bigger deal than MCP" - https://simonwillison.net/2025/Oct/16/claude-skills/
- **IntuitionLabs**: Technical comparison of Skills vs MCP - https://intuitionlabs.ai/articles/claude-skills-vs-mcp
- **Medium Deep Dive**: https://medium.com/@amanatulla1606/anthropics-model-context-protocol-mcp-a-deep-dive-for-developers-1d3db39c9fdc

### Video Courses
- DataCamp interactive tutorials
- Anthropic Skilljar courses
- Community YouTube tutorials

## 📌 Quick Reference

**Get Started:**
1. Read official docs: https://docs.claude.com/en/docs/mcp
2. Take Anthropic course: https://anthropic.skilljar.com/introduction-to-model-context-protocol
3. Explore reference servers: https://github.com/modelcontextprotocol/servers
4. Pick your SDK: TypeScript or Python
5. Build your first server using examples

**Stay Updated:**
- Follow https://github.com/modelcontextprotocol
- Check GitHub Changelog: https://github.blog/changelog/
- Join Discord communities
- Monitor awesome-mcp-servers lists
