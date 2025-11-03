# MCP Best Practices & Architecture Patterns

## 🏗️ Architecture Principles

### 1. Single Responsibility Pattern ⭐ CRITICAL

**Rule**: Each MCP server should have ONE clear, well-defined purpose.

**Why**: Aligns with microservices principles, easier to maintain, better separation of concerns.

**Good Example:**
```python
# ✅ GOOD: Focused servers
knowledge_server.py  # Only manages knowledge base
firebase_server.py   # Only Firebase operations
supabase_server.py   # Only Supabase operations
```

**Bad Example:**
```python
# ❌ BAD: Monolithic server
everything_server.py  # Does knowledge + Firebase + Supabase + IDE + ...
```

**For Optimized AI Project:**
```
@optimized-ai/mcp-core          # Core: patterns, preferences, metrics
@optimized-ai/skill-firebase    # Skill: Firebase-specific operations
@optimized-ai/skill-supabase    # Skill: Supabase-specific operations
@optimized-ai/skill-ide         # Skill: IDE operations
```

---

### 2. Tool Design Best Practices

#### Make Tools Idempotent

**Why**: AI agents may retry or parallelize requests.

```python
# ✅ GOOD: Idempotent
@mcp.tool()
def save_pattern(pattern_type: str, data: dict, overwrite: bool = True) -> dict:
    """Save pattern - safe to call multiple times."""
    # Check if exists
    if pattern_exists(pattern_type) and not overwrite:
        return {"status": "already_exists", "pattern_type": pattern_type}

    # Save (same result if called multiple times)
    patterns[pattern_type] = data
    return {"status": "success"}

# ❌ BAD: Not idempotent
@mcp.tool()
def append_to_log(message: str) -> dict:
    """Appends - multiple calls = duplicate entries."""
    log.append(message)  # Different result each time
    return {"status": "appended"}
```

#### Accept Client-Generated Request IDs

```python
@mcp.tool()
def create_resource(name: str, data: dict, request_id: str | None = None) -> dict:
    """Create resource with request ID to prevent duplicates."""
    if request_id and request_id_exists(request_id):
        return get_existing_resource(request_id)

    resource = create(name, data)
    if request_id:
        save_request_id(request_id, resource.id)

    return resource
```

#### Return Deterministic Results

```python
# ✅ GOOD: Deterministic
@mcp.tool()
def calculate_metrics(data: list) -> dict:
    """Same input = same output."""
    return {
        "mean": sum(data) / len(data),
        "count": len(data),
        "sum": sum(data),
    }

# ❌ BAD: Non-deterministic
@mcp.tool()
def get_current_status() -> dict:
    """Different output each time."""
    return {
        "timestamp": datetime.now(),  # Changes every call
        "random_id": uuid.uuid4(),    # Different each time
    }
```

#### Use Pagination for Lists

```python
@mcp.tool()
def list_patterns(
    cursor: str | None = None,
    limit: int = 50
) -> dict:
    """List patterns with pagination."""
    patterns, next_cursor = get_patterns_page(cursor, limit)

    return {
        "patterns": patterns,
        "cursor": next_cursor,
        "has_more": next_cursor is not None
    }
```

---

### 3. Schema Design

#### Use Type Hints for Auto-Generation (Python)

```python
from typing import List, Optional, Literal

@mcp.tool()
def query_database(
    collection: str,
    filters: dict,
    limit: int = 10,
    order_by: Optional[str] = None,
    direction: Literal["asc", "desc"] = "asc"
) -> List[dict]:
    """Type hints auto-generate JSON schema."""
    # FastMCP creates schema from type hints
    pass
```

#### Use Zod for Validation (TypeScript)

```typescript
import { z } from 'zod';

const QuerySchema = z.object({
  collection: z.string().min(1).max(100),
  filters: z.record(z.any()),
  limit: z.number().int().min(1).max(1000).default(10),
  orderBy: z.string().optional(),
  direction: z.enum(['asc', 'desc']).default('asc'),
});

@Tool({
  name: 'query_database',
  description: 'Query database with validation',
  paramsSchema: QuerySchema,
})
async queryDatabase(params: z.infer<typeof QuerySchema>) {
  // Params are validated automatically
}
```

#### Provide Clear Descriptions

```python
@mcp.tool()
def save_pattern(pattern_type: str, data: dict, category: str = "general") -> dict:
    """Save a new pattern to the knowledge base.

    This tool stores reusable patterns that the AI can learn from.
    Patterns are organized by category for easy retrieval.

    Args:
        pattern_type: Unique identifier (e.g., "firebase_auth", "error_handling")
        data: Pattern data including code snippets, best practices, etc.
        category: Category for organization (e.g., "firebase", "testing", "security")

    Returns:
        Success status and pattern metadata

    Examples:
        save_pattern(
            "firebase_auth",
            {"setup": "...", "patterns": [...]},
            "firebase"
        )
    """
    pass
```

---

### 4. Security Best Practices

#### Authentication & Authorization

**OAuth 2.1 (Python - FastMCP):**
```python
from fastmcp import FastMCP
from fastmcp.auth import GoogleAuth, GitHubAuth, ApiKeyAuth

# OAuth
mcp = FastMCP("Secure Server", auth=GoogleAuth())

# API Key
mcp = FastMCP("API Server", auth=ApiKeyAuth(
    api_keys={"key123": "user1", "key456": "user2"}
))

@mcp.tool()
def protected_operation(user_id: str) -> dict:
    """This requires authentication."""
    return {"user": user_id}
```

**Guards (TypeScript - NestJS):**
```typescript
import { UseGuards } from '@nestjs-mcp/server';

@Resolver('protected')
export class ProtectedResolver {
  @UseGuards(AuthGuard, RoleGuard)
  @Tool({
    name: 'admin_action',
    annotations: { destructiveHint: true }
  })
  adminAction() {
    // Only executes if guards pass
  }
}
```

#### Input Validation

```python
from pathlib import Path

@mcp.tool()
def read_file(file_path: str) -> str:
    """Read file with path validation."""
    # Validate path
    path = Path(file_path).resolve()

    # Check it's within allowed directory
    workspace = Path.cwd().resolve()
    if not path.is_relative_to(workspace):
        raise ValueError("Path outside workspace")

    # Check file exists and is a file
    if not path.exists():
        raise FileNotFoundError(f"File not found: {file_path}")
    if not path.is_file():
        raise ValueError(f"Not a file: {file_path}")

    return path.read_text()
```

#### Rate Limiting (TypeScript)

```typescript
import { Injectable } from '@nestjs/common';

class RateLimiter {
  private requests = new Map<string, number[]>();

  check(userId: string, maxRequests: number = 100, windowMs: number = 60000): boolean {
    const now = Date.now();
    const userRequests = this.requests.get(userId) || [];

    // Remove old requests
    const recentRequests = userRequests.filter(time => now - time < windowMs);

    // Check limit
    if (recentRequests.length >= maxRequests) {
      return false;
    }

    // Add new request
    recentRequests.push(now);
    this.requests.set(userId, recentRequests);

    return true;
  }
}
```

---

### 5. Performance & Scalability

#### Caching Strategies

**In-Memory Cache (Python):**
```python
from functools import lru_cache
from datetime import datetime, timedelta

class CachedKnowledge:
    def __init__(self):
        self._cache_time = {}

    @lru_cache(maxsize=100)
    def get_pattern(self, pattern_type: str) -> dict:
        """Cached pattern retrieval."""
        # Cache for 5 minutes
        cache_key = f"pattern:{pattern_type}"
        if cache_key in self._cache_time:
            if datetime.now() - self._cache_time[cache_key] < timedelta(minutes=5):
                # Return cached (from lru_cache)
                pass

        # Refresh cache timestamp
        self._cache_time[cache_key] = datetime.now()

        # Load from disk
        return load_pattern_from_disk(pattern_type)
```

**Redis Cache (TypeScript):**
```typescript
import { Injectable } from '@nestjs/common';
import Redis from 'ioredis';

@Injectable()
export class CacheService {
  private redis = new Redis();

  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set(key: string, value: any, ttlSeconds: number = 300) {
    await this.redis.setex(key, ttlSeconds, JSON.stringify(value));
  }
}

@Resolver('cached')
export class CachedResolver {
  constructor(private cache: CacheService) {}

  @Tool({ name: 'get_expensive_data' })
  async getExpensiveData({ id }: { id: string }) {
    // Try cache first
    const cached = await this.cache.get(`data:${id}`);
    if (cached) return cached;

    // Compute
    const data = await this.computeExpensiveData(id);

    // Cache result
    await this.cache.set(`data:${id}`, data, 300);

    return data;
  }
}
```

#### Connection Pooling

```python
from contextlib import contextmanager
import psycopg2.pool

class DatabasePool:
    def __init__(self):
        self.pool = psycopg2.pool.ThreadedConnectionPool(
            minconn=1,
            maxconn=10,
            database="mydb",
            user="user",
            password="pass"
        )

    @contextmanager
    def get_connection(self):
        conn = self.pool.getconn()
        try:
            yield conn
        finally:
            self.pool.putconn(conn)

# Use in tools
db_pool = DatabasePool()

@mcp.tool()
def query_database(sql: str) -> list:
    """Query with connection pooling."""
    with db_pool.get_connection() as conn:
        cursor = conn.cursor()
        cursor.execute(sql)
        return cursor.fetchall()
```

#### Async Operations

```python
import asyncio
from fastmcp import FastMCP, Context

mcp = FastMCP("Async Server")

@mcp.tool()
async def batch_process(items: list[str], context: Context) -> dict:
    """Process multiple items concurrently."""
    async def process_item(item: str) -> dict:
        # Simulate work
        await asyncio.sleep(0.1)
        return {"item": item, "status": "processed"}

    # Process concurrently
    tasks = [process_item(item) for item in items]
    results = await asyncio.gather(*tasks)

    return {"results": results, "count": len(results)}
```

---

### 6. Error Handling & Resilience

#### Comprehensive Error Handling

```python
from enum import Enum
from typing import Union

class ErrorCode(Enum):
    VALIDATION_ERROR = "VALIDATION_ERROR"
    NOT_FOUND = "NOT_FOUND"
    PERMISSION_DENIED = "PERMISSION_DENIED"
    INTERNAL_ERROR = "INTERNAL_ERROR"

def error_response(code: ErrorCode, message: str, details: dict = None) -> dict:
    """Standard error response format."""
    return {
        "error": True,
        "code": code.value,
        "message": message,
        "details": details or {}
    }

@mcp.tool()
def safe_operation(input: str) -> Union[dict, dict]:  # Success or Error
    """Operation with comprehensive error handling."""
    try:
        # Validate
        if not input:
            return error_response(
                ErrorCode.VALIDATION_ERROR,
                "Input cannot be empty"
            )

        # Check existence
        if not resource_exists(input):
            return error_response(
                ErrorCode.NOT_FOUND,
                f"Resource not found: {input}",
                {"searched_for": input}
            )

        # Perform operation
        result = perform_operation(input)

        return {
            "success": True,
            "data": result
        }

    except PermissionError:
        return error_response(
            ErrorCode.PERMISSION_DENIED,
            "Insufficient permissions"
        )
    except Exception as e:
        return error_response(
            ErrorCode.INTERNAL_ERROR,
            "Operation failed",
            {"exception": str(e)}
        )
```

#### Retry Logic

```python
import asyncio
from functools import wraps

def retry(max_attempts: int = 3, delay: float = 1.0):
    """Retry decorator for transient failures."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except TransientError as e:
                    if attempt == max_attempts - 1:
                        raise
                    await asyncio.sleep(delay * (2 ** attempt))  # Exponential backoff
            return None
        return wrapper
    return decorator

@mcp.tool()
@retry(max_attempts=3, delay=1.0)
async def unreliable_api_call(endpoint: str) -> dict:
    """Call unreliable API with retries."""
    return await fetch_from_api(endpoint)
```

---

### 7. Testing Best Practices

#### Unit Testing (Python)

```python
from fastmcp import FastMCP
import pytest

# Create server
mcp = FastMCP("Test Server")

@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

# Test with FastMCP client
@pytest.mark.asyncio
async def test_add_numbers():
    from fastmcp import Client

    # Connect to server in-memory
    async with Client(mcp) as client:
        result = await client.call_tool("add_numbers", {"a": 5, "b": 3})
        assert result == 8
```

#### Integration Testing (TypeScript)

```typescript
import { Test } from '@nestjs/testing';
import { PatternsResolver } from './patterns.resolver';

describe('PatternsResolver', () => {
  let resolver: PatternsResolver;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [PatternsResolver],
    }).compile();

    resolver = module.get(PatternsResolver);
  });

  it('should save and retrieve pattern', async () => {
    const pattern = { data: 'test' };

    await resolver.savePattern({
      patternType: 'test_pattern',
      data: pattern,
      category: 'test',
    });

    const result = await resolver.getPattern({
      patternType: 'test_pattern',
    });

    expect(result.data).toEqual(pattern);
  });
});
```

---

### 8. Observability

#### Logging

```python
import logging
from fastmcp import FastMCP, Context

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

mcp = FastMCP("Observable Server")

@mcp.tool()
async def tracked_operation(input: str, context: Context) -> dict:
    """Operation with comprehensive logging."""
    logger.info(f"Starting operation with input: {input}")

    try:
        result = await perform_operation(input)

        logger.info(f"Operation successful: {result}")

        return {"success": True, "result": result}

    except Exception as e:
        logger.error(f"Operation failed: {e}", exc_info=True)
        raise
```

#### Metrics

```python
from prometheus_client import Counter, Histogram
import time

# Define metrics
tool_calls = Counter('mcp_tool_calls_total', 'Total tool calls', ['tool_name', 'status'])
tool_duration = Histogram('mcp_tool_duration_seconds', 'Tool execution duration', ['tool_name'])

@mcp.tool()
def monitored_operation(input: str) -> dict:
    """Operation with metrics."""
    start = time.time()

    try:
        result = perform_operation(input)

        tool_calls.labels(tool_name='monitored_operation', status='success').inc()
        tool_duration.labels(tool_name='monitored_operation').observe(time.time() - start)

        return result

    except Exception as e:
        tool_calls.labels(tool_name='monitored_operation', status='error').inc()
        raise
```

---

### 9. Organization Patterns for Multi-Server Setup

#### By Domain (Recommended)

```
servers/
├── core/                    # Core knowledge operations
│   ├── patterns_server.py
│   └── preferences_server.py
├── skills/
│   ├── firebase_server.py   # Firebase skill
│   ├── supabase_server.py   # Supabase skill
│   └── ide_server.py        # IDE operations
└── utils/
    └── testing_server.py    # Testing utilities
```

#### By Permission Level

```
servers/
├── read_only/           # Read operations
│   └── query_server.py
├── read_write/          # Standard operations
│   └── crud_server.py
└── admin/               # Destructive operations
    └── admin_server.py
```

#### By Performance Tier

```
servers/
├── fast/               # Quick operations (< 100ms)
│   └── cache_server.py
├── standard/           # Normal operations (< 1s)
│   └── api_server.py
└── batch/              # Long-running (> 1s)
    └── processing_server.py
```

---

## 🎯 Summary: Key Takeaways

1. **Single Responsibility**: One server = one purpose
2. **Idempotent Tools**: Safe to retry
3. **Type Safety**: Use Zod (TS) or type hints (Python)
4. **Security**: Authentication, validation, rate limiting
5. **Performance**: Caching, pooling, async operations
6. **Resilience**: Error handling, retries, timeouts
7. **Observability**: Logging, metrics, tracing
8. **Testing**: Unit and integration tests
9. **Organization**: Group by domain/permission/performance

## 📚 For Optimized AI Project

Based on your MINIMIZE, SEPARATE, VALIDATE principles:

- ✅ **MINIMIZE**: Single-purpose servers, efficient caching, minimal boilerplate
- ✅ **SEPARATE**: Domain-based organization (core, skills), load on-demand
- ✅ **VALIDATE**: Comprehensive testing, metrics, experimental validation
