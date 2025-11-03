# Python MCP Frameworks Comparison

## Overview

This document compares Python frameworks for building MCP servers, focusing on decorator patterns for ease of use.

## 🏆 Framework Rankings

### 1. FastMCP ⭐ RECOMMENDED (Official)

**Best for**: ALL Python MCP servers - it's the official decorator framework

#### Installation
```bash
pip install fastmcp
# or
pip install mcp  # Includes FastMCP
```

#### Key Features
- ✅ **Official framework** - Integrated into Python SDK
- ✅ **Perfect decorator pattern** - `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()`
- ✅ **Automatic schema generation** - From type hints and docstrings
- ✅ **Production-ready** - Created by Prefect team
- ✅ **Pythonic** - Feels like Flask/FastAPI
- ✅ **Zero-config OAuth** - Built-in authentication
- ✅ **FastMCP Cloud** - One-command deployment

#### Basic Examples

**Simple Tool:**
```python
from fastmcp import FastMCP

mcp = FastMCP("Calculator Server")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

@mcp.tool()
def multiply(a: float, b: float) -> float:
    """Multiply two numbers.

    Args:
        a: First number
        b: Second number

    Returns:
        Product of a and b
    """
    return a * b
```

**Resource with URI Template:**
```python
@mcp.resource("users://{user_id}/profile")
def get_user_profile(user_id: int) -> dict:
    """Get user profile by ID.

    Args:
        user_id: The unique user identifier

    Returns:
        User profile data
    """
    return {
        "id": user_id,
        "name": f"User {user_id}",
        "status": "active"
    }
```

**Dynamic Resource List:**
```python
@mcp.resource("file://{path}")
def read_file(path: str) -> str:
    """Read file contents.

    Args:
        path: File path relative to workspace

    Returns:
        File contents as string
    """
    with open(path, 'r') as f:
        return f.read()
```

**Prompt Template:**
```python
@mcp.prompt()
def code_review_prompt(language: str, style: str = "standard") -> str:
    """Generate a code review prompt.

    Args:
        language: Programming language
        style: Review style (default: "standard")

    Returns:
        Formatted prompt for code review
    """
    return f"""
    Please review the following {language} code.
    Apply {style} code review standards.
    Focus on:
    - Code quality
    - Best practices
    - Potential bugs
    - Performance issues
    """
```

**Prompt Returning Messages:**
```python
from fastmcp import FastMCP, Message

mcp = FastMCP("Prompt Server")

@mcp.prompt()
def generate_test_prompt(framework: str) -> list[Message]:
    """Generate testing prompt for a framework.

    Args:
        framework: Testing framework name

    Returns:
        List of messages for the prompt
    """
    return [
        Message(
            role="user",
            content=f"Write comprehensive tests using {framework}"
        ),
        Message(
            role="assistant",
            content="I'll help you write tests. What would you like to test?"
        ),
    ]
```

#### Advanced Features

**Context Access for Progress and Sampling:**
```python
from fastmcp import FastMCP, Context

mcp = FastMCP("Advanced Server")

@mcp.tool()
async def long_running_task(steps: int, context: Context) -> str:
    """Execute a long-running task with progress updates.

    Args:
        steps: Number of steps to execute
        context: MCP context for progress reporting

    Returns:
        Task completion status
    """
    for i in range(steps):
        # Report progress
        await context.report_progress(
            progress=i + 1,
            total=steps
        )

        # Do work
        await asyncio.sleep(0.1)

    return f"Completed {steps} steps"

@mcp.tool()
async def ask_llm(question: str, context: Context) -> str:
    """Ask the LLM a question using sampling.

    Args:
        question: Question to ask
        context: MCP context for LLM sampling

    Returns:
        LLM response
    """
    response = await context.sample(
        messages=[
            Message(role="user", content=question)
        ],
        max_tokens=1000
    )

    return response.content
```

**Server Composition:**
```python
from fastmcp import FastMCP

# Core server
core_mcp = FastMCP("Core")

@core_mcp.tool()
def core_operation():
    """Core operation."""
    return "Core"

# Feature server
feature_mcp = FastMCP("Features")

@feature_mcp.tool()
def feature_operation():
    """Feature operation."""
    return "Feature"

# Compose servers
main_mcp = FastMCP("Main")
main_mcp.mount("/core", core_mcp)
main_mcp.mount("/features", feature_mcp)

# Or import all tools
main_mcp.import_server(core_mcp)
main_mcp.import_server(feature_mcp)
```

**Authentication (Zero-Config OAuth):**
```python
from fastmcp import FastMCP
from fastmcp.auth import GoogleAuth, GitHubAuth

mcp = FastMCP("Secure Server", auth=GoogleAuth())

@mcp.tool()
def protected_operation(user_email: str) -> str:
    """This tool requires Google OAuth authentication."""
    return f"Hello {user_email}!"

# Also supports: GitHub, Microsoft, Auth0, WorkOS, Descope
# Plus API keys and JWT
```

#### Deployment Options

**Local Development:**
```bash
fastmcp run server.py
```

**FastMCP Cloud (Instant HTTPS):**
```bash
fastmcp deploy server.py
```

**Self-Hosted (Docker):**
```dockerfile
FROM python:3.11
COPY server.py .
RUN pip install fastmcp
CMD ["fastmcp", "run", "server.py"]
```

#### Pros
- 🟢 **Official** - Part of Python SDK, maintained by Anthropic/Prefect
- 🟢 **Perfect decorator pattern** - Exactly what you asked for
- 🟢 **Type hints = schemas** - No manual schema definition
- 🟢 **Pythonic** - Natural Python code style
- 🟢 **Production features** - Auth, deployment, testing
- 🟢 **Excellent docs** - https://gofastmcp.com/
- 🟢 **Active development** - Regular updates

#### Cons
- 🔴 None significant - it's the official recommended approach

---

## 🎯 Alternative: Official Python SDK (Low-Level)

**Best for**: When you need maximum control or custom architecture

#### Installation
```bash
pip install mcp
```

#### Example
```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("my-server")

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="add",
            description="Add two numbers",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "add":
        result = arguments["a"] + arguments["b"]
        return [TextContent(type="text", text=str(result))]
```

#### Pros
- 🟢 Maximum control
- 🟢 Direct protocol access
- 🟢 Official SDK

#### Cons
- 🔴 Much more verbose
- 🔴 Manual schema definition
- 🔴 No decorator pattern
- 🔴 More boilerplate

**Verdict**: Use FastMCP instead unless you have very specific low-level needs.

---

## 🏗️ For Optimized AI Project

**Recommendation: FastMCP**

Reasons:
1. ✅ **Matches your requirement**: "Just write function and wrap with decorator"
2. ✅ **Official** - Part of Python SDK, won't be deprecated
3. ✅ **Minimize**: Clean code, auto-generated schemas, minimal boilerplate
4. ✅ **Production-ready**: Auth, deployment, monitoring built-in
5. ✅ **Pythonic**: Natural Python patterns

### Example for Your Project

```python
# @optimized-ai/mcp-scripts (Python utilities)

from fastmcp import FastMCP, Context
import json
from pathlib import Path

mcp = FastMCP("Optimized AI Scripts")

# Core knowledge operations
@mcp.tool()
def get_pattern(pattern_type: str) -> dict:
    """Retrieve learned pattern from knowledge base.

    Args:
        pattern_type: Type of pattern to retrieve

    Returns:
        Pattern data from .ai-knowledge/patterns.json
    """
    knowledge_dir = Path(".ai-knowledge")
    patterns_file = knowledge_dir / "patterns.json"

    if not patterns_file.exists():
        return {"error": "No patterns found"}

    patterns = json.loads(patterns_file.read_text())
    return patterns.get(pattern_type, {"error": "Pattern not found"})

@mcp.tool()
def save_pattern(pattern_type: str, data: dict) -> dict:
    """Save new pattern to knowledge base.

    Args:
        pattern_type: Type of pattern
        data: Pattern data to save

    Returns:
        Success status
    """
    knowledge_dir = Path(".ai-knowledge")
    knowledge_dir.mkdir(exist_ok=True)
    patterns_file = knowledge_dir / "patterns.json"

    patterns = {}
    if patterns_file.exists():
        patterns = json.loads(patterns_file.read_text())

    patterns[pattern_type] = data
    patterns_file.write_text(json.dumps(patterns, indent=2))

    return {"status": "success", "type": pattern_type}

# Data analysis tool
@mcp.tool()
async def analyze_codebase(directory: str, context: Context) -> dict:
    """Analyze codebase structure and patterns.

    Args:
        directory: Directory to analyze
        context: MCP context for progress reporting

    Returns:
        Analysis results
    """
    from pathlib import Path

    path = Path(directory)
    files = list(path.rglob("*.py")) + list(path.rglob("*.ts"))

    results = {"files": [], "patterns": []}
    total = len(files)

    for i, file in enumerate(files):
        await context.report_progress(progress=i+1, total=total)

        # Analyze file
        content = file.read_text()
        results["files"].append({
            "path": str(file),
            "lines": len(content.splitlines()),
        })

    return results

# Resource for project preferences
@mcp.resource("preferences://coding")
def get_coding_preferences() -> dict:
    """Get coding preferences from .ai-knowledge.

    Returns:
        Coding preferences and standards
    """
    knowledge_dir = Path(".ai-knowledge")
    prefs_file = knowledge_dir / "preferences.json"

    if not prefs_file.exists():
        return {"error": "No preferences found"}

    return json.loads(prefs_file.read_text())

# Prompt for code review
@mcp.prompt()
def code_review_with_patterns(language: str) -> str:
    """Generate code review prompt using learned patterns.

    Args:
        language: Programming language

    Returns:
        Formatted code review prompt with learned patterns
    """
    # Load learned patterns
    patterns = get_pattern(f"{language}_patterns")

    return f"""
    Review this {language} code using these learned patterns:
    {json.dumps(patterns, indent=2)}

    Check for:
    - Adherence to learned patterns
    - Code quality
    - Best practices from our knowledge base
    """
```

## 📊 Feature Comparison

| Feature | FastMCP | Official SDK (Low-Level) |
|---------|---------|-------------------------|
| Decorator Pattern | ✅✅ Excellent | ❌ None |
| Type Hints → Schema | ✅ Automatic | ❌ Manual |
| Docstring → Description | ✅ Automatic | ❌ Manual |
| Context Access | ✅ Advanced | ✅ Advanced |
| Progress Reporting | ✅ Built-in | ⚠️ Manual |
| LLM Sampling | ✅ Built-in | ⚠️ Manual |
| Authentication | ✅ Zero-config OAuth | ❌ Manual |
| Server Composition | ✅ mount/import | ❌ Manual |
| Deployment Tools | ✅ fastmcp deploy | ❌ Manual |
| Testing Utilities | ✅ In-memory client | ⚠️ Limited |
| Boilerplate | ⚠️ Minimal | 🔴 High |
| Learning Curve | 🟢 Low | 🟡 Medium |
| Documentation | ✅ Excellent | ✅ Good |
| Official Support | ✅ Yes | ✅ Yes |

## 🚀 Quick Start

### FastMCP (Recommended)
```bash
# Install
pip install fastmcp

# Create server.py
cat > server.py << 'EOF'
from fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
def hello(name: str) -> str:
    """Say hello."""
    return f"Hello, {name}!"
EOF

# Run locally
fastmcp run server.py

# Deploy to cloud (optional)
fastmcp deploy server.py
```

## 📝 Decorator Patterns for Class Methods

**Important**: FastMCP decorators work best with functions. For class methods, register after instantiation:

```python
from fastmcp import FastMCP

mcp = FastMCP("Class Server")

class PatternManager:
    def __init__(self, knowledge_dir: str):
        self.knowledge_dir = knowledge_dir

    def get_pattern(self, pattern_type: str) -> dict:
        """Get pattern from knowledge base."""
        # Implementation
        return {}

    def save_pattern(self, pattern_type: str, data: dict) -> dict:
        """Save pattern to knowledge base."""
        # Implementation
        return {"status": "success"}

# Create instance
manager = PatternManager(".ai-knowledge")

# Register methods
mcp.tool()(manager.get_pattern)
mcp.tool()(manager.save_pattern)

# Alternative: Register in __init__
class AutoRegisterManager:
    def __init__(self, mcp: FastMCP):
        mcp.tool()(self.get_pattern)
        mcp.tool()(self.save_pattern)

    def get_pattern(self, pattern_type: str) -> dict:
        """Get pattern."""
        return {}

    def save_pattern(self, pattern_type: str, data: dict) -> dict:
        """Save pattern."""
        return {"status": "success"}

manager = AutoRegisterManager(mcp)
```

## 🎓 Learning Resources

### FastMCP
- **Official Docs**: https://gofastmcp.com/
- **GitHub**: https://github.com/jlowin/fastmcp
- **PyPI**: https://pypi.org/project/fastmcp/
- **Tutorial**: https://www.datacamp.com/tutorial/building-mcp-server-client-fastmcp
- **Guide**: https://www.pondhouse-data.com/blog/create-mcp-server-with-fastmcp

### Official Python SDK
- **GitHub**: https://github.com/modelcontextprotocol/python-sdk
- **Docs**: https://modelcontextprotocol.io/

## 🎯 Summary

**For Python MCP servers, use FastMCP.**

It's the official decorator framework, provides exactly the pattern you want ("write function, wrap with decorator"), and includes production features like auth and deployment.

The low-level SDK is only needed if you're building a custom framework or have very specific protocol requirements.
