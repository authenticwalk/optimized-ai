# Model Context Protocol (MCP) Research

Complete research on Claude MCP for the Optimized AI project.

## 📚 Research Documents

### Core Documentation
1. **[Official Documentation & Resources](01-official-docs.md)** - Links to official MCP docs, SDKs, and learning resources
2. **[TypeScript Frameworks Comparison](02-typescript-frameworks.md)** - Analysis of TS frameworks with decorator patterns
3. **[Python Frameworks Comparison](03-python-frameworks.md)** - Analysis of Python frameworks with decorator patterns
4. **[Code Examples & Patterns](04-code-examples.md)** - Comprehensive code snippets and implementation patterns
5. **[Best Practices & Architecture](05-best-practices.md)** - Production-ready patterns and scalability
6. **[Real-World Examples](06-real-world-examples.md)** - Production MCP servers and use cases
7. **[MCP vs Claude Code Skills](07-mcp-vs-skills.md)** - When to use each and how they complement
8. **[Recommendations for Optimized AI](08-recommendations.md)** - Framework choices and implementation strategy

## 🎯 Quick Summary

### What is MCP?
The Model Context Protocol (MCP) is an open protocol that standardizes how AI applications connect to external data sources and tools. It provides three core primitives:
- **Tools** - Executable functions LLMs can invoke
- **Resources** - Read-only data sources
- **Prompts** - Reusable message templates

### Recommended Frameworks

**For TypeScript:**
- **Official SDK**: `@modelcontextprotocol/sdk` - Full MCP implementation
- **Best Decorator Framework**: `@nestjs-mcp/server` - NestJS-based with clean decorators
- **Alternative**: `fastmcp` (TypeScript) - Express-like API

**For Python:**
- **FastMCP** - Official decorator-based framework integrated into Python SDK
- Uses `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()` decorators
- Type hints auto-generate schemas

### MCP vs Skills

| Feature | MCP Servers | Claude Skills |
|---------|-------------|---------------|
| **Purpose** | Connect to external systems | Teach workflows & procedures |
| **Best For** | APIs, databases, cloud services | Organizational standards, patterns |
| **Token Usage** | Network calls required | Minimal (30-50 tokens until loaded) |
| **Portability** | Works across all MCP clients | Claude ecosystem only |
| **Complexity** | Higher (protocol, auth, deployment) | Lower (just Markdown files) |

**Verdict**: Use both! Skills orchestrate MCP server calls with your organization's standards.

## 🚀 For Optimized AI Project

Based on project goals (MINIMIZE, SEPARATE, VALIDATE):

1. **Use Skills for core patterns** - Load on-demand, minimal tokens
2. **Use MCP for specific integrations** - IDE ops, Firebase, Supabase
3. **TypeScript Framework**: `@nestjs-mcp/server` - Clean decorators, NestJS ecosystem
4. **Python Framework**: `FastMCP` - Official, simple decorator pattern

See [Recommendations](08-recommendations.md) for detailed implementation strategy.

## 📅 Research Date
November 3, 2025

## 🔗 Key Links
- Official Docs: https://docs.claude.com/en/docs/mcp
- GitHub: https://github.com/modelcontextprotocol
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- FastMCP: https://github.com/jlowin/fastmcp
- NestJS MCP: https://www.npmjs.com/package/@nestjs-mcp/server
