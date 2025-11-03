# MCP Server Code Examples & Patterns

## Complete Working Examples

This document provides copy-paste ready code examples for building MCP servers.

## 🐍 Python Examples (FastMCP)

### Example 1: Knowledge Base Server

**File: `knowledge_server.py`**

```python
from fastmcp import FastMCP
from pathlib import Path
import json
from typing import Dict, List, Optional
from datetime import datetime

mcp = FastMCP("Knowledge Base Server", version="1.0.0")

KNOWLEDGE_DIR = Path(".ai-knowledge")

# ============================================================================
# TOOLS - Executable Functions
# ============================================================================

@mcp.tool()
def get_pattern(pattern_type: str) -> Dict:
    """Retrieve a learned pattern from the knowledge base.

    Args:
        pattern_type: Type of pattern (e.g., "firebase_auth", "testing")

    Returns:
        Pattern data or error message
    """
    patterns_file = KNOWLEDGE_DIR / "patterns.json"

    if not patterns_file.exists():
        return {"error": "No patterns found", "patterns": []}

    patterns = json.loads(patterns_file.read_text())
    pattern = patterns.get(pattern_type)

    if not pattern:
        available = list(patterns.keys())
        return {
            "error": f"Pattern '{pattern_type}' not found",
            "available_patterns": available
        }

    return {
        "type": pattern_type,
        "data": pattern,
        "retrieved_at": datetime.now().isoformat()
    }

@mcp.tool()
def save_pattern(pattern_type: str, data: Dict, category: str = "general") -> Dict:
    """Save a new pattern to the knowledge base.

    Args:
        pattern_type: Unique identifier for the pattern
        data: Pattern data to save
        category: Category for organization (default: "general")

    Returns:
        Success status and saved pattern info
    """
    KNOWLEDGE_DIR.mkdir(exist_ok=True)
    patterns_file = KNOWLEDGE_DIR / "patterns.json"

    # Load existing patterns
    patterns = {}
    if patterns_file.exists():
        patterns = json.loads(patterns_file.read_text())

    # Add metadata
    pattern_with_meta = {
        "category": category,
        "data": data,
        "created_at": datetime.now().isoformat(),
        "version": "1.0"
    }

    patterns[pattern_type] = pattern_with_meta

    # Save back
    patterns_file.write_text(json.dumps(patterns, indent=2))

    return {
        "status": "success",
        "pattern_type": pattern_type,
        "category": category
    }

@mcp.tool()
def list_patterns(category: Optional[str] = None) -> List[Dict]:
    """List all available patterns, optionally filtered by category.

    Args:
        category: Optional category filter

    Returns:
        List of pattern summaries
    """
    patterns_file = KNOWLEDGE_DIR / "patterns.json"

    if not patterns_file.exists():
        return []

    patterns = json.loads(patterns_file.read_text())
    result = []

    for pattern_type, pattern_data in patterns.items():
        if category and pattern_data.get("category") != category:
            continue

        result.append({
            "type": pattern_type,
            "category": pattern_data.get("category", "unknown"),
            "created_at": pattern_data.get("created_at"),
        })

    return result

@mcp.tool()
def record_success(task: str, approach: str, metrics: Dict) -> Dict:
    """Record a successful task completion.

    Args:
        task: Task description
        approach: Approach used
        metrics: Performance metrics (tokens, time, etc.)

    Returns:
        Success status
    """
    KNOWLEDGE_DIR.mkdir(exist_ok=True)
    successes_file = KNOWLEDGE_DIR / "successes.json"

    successes = []
    if successes_file.exists():
        successes = json.loads(successes_file.read_text())

    successes.append({
        "task": task,
        "approach": approach,
        "metrics": metrics,
        "timestamp": datetime.now().isoformat()
    })

    successes_file.write_text(json.dumps(successes, indent=2))

    return {"status": "success", "count": len(successes)}

@mcp.tool()
def record_failure(task: str, approach: str, reason: str) -> Dict:
    """Record a failed task attempt to avoid repeating mistakes.

    Args:
        task: Task description
        approach: Approach that failed
        reason: Why it failed

    Returns:
        Success status
    """
    KNOWLEDGE_DIR.mkdir(exist_ok=True)
    failures_file = KNOWLEDGE_DIR / "failures.json"

    failures = []
    if failures_file.exists():
        failures = json.loads(failures_file.read_text())

    failures.append({
        "task": task,
        "approach": approach,
        "reason": reason,
        "timestamp": datetime.now().isoformat()
    })

    failures_file.write_text(json.dumps(failures, indent=2))

    return {"status": "recorded", "count": len(failures)}

# ============================================================================
# RESOURCES - Read-Only Data
# ============================================================================

@mcp.resource("knowledge://preferences")
def get_preferences() -> str:
    """Get coding preferences and standards.

    Returns:
        JSON string of preferences
    """
    prefs_file = KNOWLEDGE_DIR / "preferences.json"

    if not prefs_file.exists():
        return json.dumps({"error": "No preferences found"})

    return prefs_file.read_text()

@mcp.resource("knowledge://patterns/{category}")
def get_patterns_by_category(category: str) -> str:
    """Get all patterns in a specific category.

    Args:
        category: Category name

    Returns:
        JSON string of patterns in category
    """
    patterns = list_patterns(category=category)
    return json.dumps(patterns, indent=2)

@mcp.resource("knowledge://metrics")
def get_metrics() -> str:
    """Get performance metrics summary.

    Returns:
        JSON string of metrics
    """
    metrics_file = KNOWLEDGE_DIR / "metrics.json"

    if not metrics_file.exists():
        return json.dumps({"error": "No metrics found"})

    return metrics_file.read_text()

# ============================================================================
# PROMPTS - Reusable Templates
# ============================================================================

@mcp.prompt()
def task_planner(task_description: str, complexity: str = "medium") -> str:
    """Generate a task planning prompt with learned patterns.

    Args:
        task_description: Description of the task
        complexity: Task complexity (low, medium, high)

    Returns:
        Formatted planning prompt
    """
    # Load relevant patterns
    patterns_file = KNOWLEDGE_DIR / "patterns.json"
    context = ""

    if patterns_file.exists():
        patterns = json.loads(patterns_file.read_text())
        context = f"\n\nRelevant patterns: {json.dumps(patterns, indent=2)}"

    return f"""
# Task Planning

**Task**: {task_description}
**Complexity**: {complexity}

{context}

## Planning Steps

1. **Analyze Requirements**
   - What are the key requirements?
   - What patterns apply?
   - What could go wrong?

2. **Break Down Task**
   - List subtasks
   - Identify dependencies
   - Estimate effort

3. **Select Approach**
   - Review similar past tasks
   - Choose proven patterns
   - Avoid known failures

4. **Define Success Criteria**
   - How do we know it's done?
   - What metrics matter?
   - What tests are needed?

Please create a detailed plan following this structure.
"""

# ============================================================================
# RUN SERVER
# ============================================================================

if __name__ == "__main__":
    mcp.run()
```

**Run it:**
```bash
fastmcp run knowledge_server.py
```

---

### Example 2: Firebase Integration Server

**File: `firebase_server.py`**

```python
from fastmcp import FastMCP, Context
from typing import Dict, List, Optional
import json

mcp = FastMCP("Firebase Helper Server")

# Note: In production, use firebase-admin SDK
# This is a simplified example

@mcp.tool()
async def query_firestore(
    collection: str,
    filters: Dict,
    limit: int = 10,
    context: Context = None
) -> List[Dict]:
    """Query Firestore collection with filters.

    Args:
        collection: Collection name
        filters: Query filters as dict
        limit: Maximum results to return
        context: MCP context (auto-injected)

    Returns:
        List of matching documents
    """
    # Report progress
    if context:
        await context.report_progress(progress=1, total=3)

    # In production, use firebase-admin:
    # import firebase_admin
    # from firebase_admin import firestore
    # db = firestore.client()
    # query = db.collection(collection)
    # for field, value in filters.items():
    #     query = query.where(field, "==", value)
    # docs = query.limit(limit).stream()

    # Simplified example
    results = [
        {"id": "doc1", "data": {"example": "data"}},
        {"id": "doc2", "data": {"example": "data2"}},
    ]

    if context:
        await context.report_progress(progress=3, total=3)

    return results

@mcp.tool()
def validate_security_rules(rules_path: str) -> Dict:
    """Validate Firebase security rules.

    Args:
        rules_path: Path to firestore.rules file

    Returns:
        Validation results
    """
    from pathlib import Path

    rules_file = Path(rules_path)

    if not rules_file.exists():
        return {"valid": False, "error": "Rules file not found"}

    rules_content = rules_file.read_text()

    # Basic validation (in production, use Firebase CLI)
    issues = []
    if "match /{document=**}" in rules_content:
        issues.append("Overly permissive wildcard rule detected")

    return {
        "valid": len(issues) == 0,
        "issues": issues,
        "rules_path": str(rules_file)
    }

@mcp.resource("firebase://config")
def get_firebase_config() -> str:
    """Get Firebase configuration.

    Returns:
        JSON string of Firebase config
    """
    config = {
        "projectId": "your-project-id",
        "region": "us-central1",
        "collections": ["users", "posts", "comments"]
    }
    return json.dumps(config, indent=2)

@mcp.prompt()
def firestore_query_prompt(collection: str, requirements: str) -> str:
    """Generate prompt for Firestore query design.

    Args:
        collection: Collection name
        requirements: Query requirements

    Returns:
        Formatted prompt for query design
    """
    return f"""
# Firestore Query Design

**Collection**: {collection}
**Requirements**: {requirements}

## Design Considerations

1. **Index Requirements**
   - What compound indexes are needed?
   - Check firestore.indexes.json

2. **Query Efficiency**
   - Use equality filters before range filters
   - Limit results appropriately
   - Consider pagination

3. **Security Rules**
   - Ensure rules allow this query
   - Check user permissions
   - Validate data access patterns

4. **Performance**
   - Avoid large reads
   - Use query cursors for pagination
   - Cache when appropriate

Please design an efficient Firestore query following these guidelines.
"""

if __name__ == "__main__":
    mcp.run()
```

---

## 📘 TypeScript Examples (@nestjs-mcp/server)

### Example 1: Pattern Management Module

**File: `src/patterns/patterns.resolver.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { Tool, Resource, Resolver } from '@nestjs-mcp/server';
import { z } from 'zod';
import { promises as fs } from 'fs';
import path from 'path';

const KNOWLEDGE_DIR = '.ai-knowledge';

interface Pattern {
  category: string;
  data: any;
  created_at: string;
  version: string;
}

@Injectable()
@Resolver('patterns')
export class PatternsResolver {
  // ========================================================================
  // TOOLS
  // ========================================================================

  @Tool({
    name: 'get_pattern',
    description: 'Retrieve a learned pattern from the knowledge base',
    paramsSchema: {
      patternType: z.string().describe('Type of pattern to retrieve'),
    },
  })
  async getPattern({ patternType }: { patternType: string }) {
    const patternsFile = path.join(KNOWLEDGE_DIR, 'patterns.json');

    try {
      const content = await fs.readFile(patternsFile, 'utf-8');
      const patterns: Record<string, Pattern> = JSON.parse(content);

      if (!patterns[patternType]) {
        return {
          error: `Pattern '${patternType}' not found`,
          available_patterns: Object.keys(patterns),
        };
      }

      return {
        type: patternType,
        data: patterns[patternType],
        retrieved_at: new Date().toISOString(),
      };
    } catch (error) {
      return { error: 'No patterns found', patterns: [] };
    }
  }

  @Tool({
    name: 'save_pattern',
    description: 'Save a new pattern to the knowledge base',
    paramsSchema: {
      patternType: z.string().describe('Unique identifier for pattern'),
      data: z.object({}).passthrough().describe('Pattern data'),
      category: z.string().default('general').describe('Pattern category'),
    },
  })
  async savePattern({
    patternType,
    data,
    category = 'general',
  }: {
    patternType: string;
    data: any;
    category?: string;
  }) {
    const patternsFile = path.join(KNOWLEDGE_DIR, 'patterns.json');

    // Ensure directory exists
    await fs.mkdir(KNOWLEDGE_DIR, { recursive: true });

    // Load existing patterns
    let patterns: Record<string, Pattern> = {};
    try {
      const content = await fs.readFile(patternsFile, 'utf-8');
      patterns = JSON.parse(content);
    } catch {
      // File doesn't exist yet
    }

    // Add new pattern
    patterns[patternType] = {
      category,
      data,
      created_at: new Date().toISOString(),
      version: '1.0',
    };

    // Save
    await fs.writeFile(patternsFile, JSON.stringify(patterns, null, 2));

    return {
      status: 'success',
      pattern_type: patternType,
      category,
    };
  }

  @Tool({
    name: 'list_patterns',
    description: 'List all available patterns',
    paramsSchema: {
      category: z.string().optional().describe('Filter by category'),
    },
  })
  async listPatterns({ category }: { category?: string }) {
    const patternsFile = path.join(KNOWLEDGE_DIR, 'patterns.json');

    try {
      const content = await fs.readFile(patternsFile, 'utf-8');
      const patterns: Record<string, Pattern> = JSON.parse(content);

      const result = Object.entries(patterns)
        .filter(([, pattern]) => !category || pattern.category === category)
        .map(([type, pattern]) => ({
          type,
          category: pattern.category,
          created_at: pattern.created_at,
        }));

      return result;
    } catch {
      return [];
    }
  }

  // ========================================================================
  // RESOURCES
  // ========================================================================

  @Resource({
    uri: 'knowledge://patterns/{category}',
    name: 'patterns_by_category',
    description: 'Get all patterns in a specific category',
  })
  async getPatternsByCategory({ category }: { category: string }) {
    const patterns = await this.listPatterns({ category });

    return {
      uri: `knowledge://patterns/${category}`,
      text: JSON.stringify(patterns, null, 2),
      mimeType: 'application/json',
    };
  }
}
```

**File: `src/patterns/patterns.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { PatternsResolver } from './patterns.resolver';

@Module({
  providers: [PatternsResolver],
  exports: [PatternsResolver],
})
export class PatternsModule {}
```

**File: `src/app.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { McpModule } from '@nestjs-mcp/server';
import { PatternsModule } from './patterns/patterns.module';

@Module({
  imports: [
    McpModule.forRoot({
      name: 'Optimized AI Core',
      version: '1.0.0',
    }),
    PatternsModule,
  ],
})
export class AppModule {}
```

**File: `src/main.ts`**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { McpService } from '@nestjs-mcp/server';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Get MCP service and start
  const mcpService = app.get(McpService);
  await mcpService.start();
}

bootstrap();
```

---

### Example 2: With Guards and DI

**File: `src/guards/auth.guard.ts`**

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    // Implement your auth logic
    return true; // or false to deny
  }
}
```

**File: `src/firebase/firebase.resolver.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { Tool, Resolver, UseGuards } from '@nestjs-mcp/server';
import { z } from 'zod';
import { AuthGuard } from '../guards/auth.guard';
import { FirebaseService } from './firebase.service';

@Injectable()
@Resolver('firebase')
export class FirebaseResolver {
  constructor(private readonly firebaseService: FirebaseService) {}

  @Tool({
    name: 'query_firestore',
    description: 'Query Firestore collection',
    paramsSchema: {
      collection: z.string(),
      filters: z.object({}).passthrough(),
      limit: z.number().default(10),
    },
  })
  async queryFirestore({
    collection,
    filters,
    limit = 10,
  }: {
    collection: string;
    filters: any;
    limit?: number;
  }) {
    // Use injected service
    return this.firebaseService.query(collection, filters, limit);
  }

  @UseGuards(AuthGuard)
  @Tool({
    name: 'delete_collection',
    description: 'Delete entire Firestore collection (requires auth)',
    paramsSchema: {
      collection: z.string(),
    },
    annotations: {
      destructiveHint: true,
    },
  })
  async deleteCollection({ collection }: { collection: string }) {
    return this.firebaseService.deleteCollection(collection);
  }
}
```

---

## 🔄 Common Patterns

### Pattern 1: Progress Reporting (Python)

```python
from fastmcp import FastMCP, Context
import asyncio

mcp = FastMCP("Progress Server")

@mcp.tool()
async def process_large_dataset(items_count: int, context: Context) -> dict:
    """Process large dataset with progress reporting."""
    results = []

    for i in range(items_count):
        # Report progress
        await context.report_progress(
            progress=i + 1,
            total=items_count
        )

        # Do work
        await asyncio.sleep(0.1)
        results.append(f"Item {i}")

    return {"processed": len(results)}
```

### Pattern 2: Error Handling (TypeScript)

```typescript
@Tool({
  name: 'safe_operation',
  description: 'Operation with comprehensive error handling',
  paramsSchema: { input: z.string() },
})
async safeOperation({ input }: { input: string }) {
  try {
    // Validate input
    if (!input || input.trim().length === 0) {
      return {
        error: 'Invalid input',
        code: 'VALIDATION_ERROR',
      };
    }

    // Do operation
    const result = await this.performOperation(input);

    return {
      status: 'success',
      data: result,
    };
  } catch (error) {
    return {
      error: error.message,
      code: 'OPERATION_FAILED',
      details: error.stack,
    };
  }
}
```

### Pattern 3: Caching (Python)

```python
from functools import lru_cache
import json

class CachedPatternManager:
    def __init__(self, mcp: FastMCP):
        mcp.tool()(self.get_pattern)

    @lru_cache(maxsize=100)
    def get_pattern(self, pattern_type: str) -> dict:
        """Get pattern with caching."""
        # This result will be cached
        with open('.ai-knowledge/patterns.json') as f:
            patterns = json.load(f)
        return patterns.get(pattern_type, {})
```

### Pattern 4: Validation (TypeScript)

```typescript
const FirestoreQuerySchema = z.object({
  collection: z.string().min(1).max(100),
  filters: z.record(z.any()),
  limit: z.number().min(1).max(1000),
  orderBy: z.string().optional(),
});

@Tool({
  name: 'query_with_validation',
  description: 'Firestore query with strict validation',
  paramsSchema: FirestoreQuerySchema,
})
async queryWithValidation(params: z.infer<typeof FirestoreQuerySchema>) {
  // Params are automatically validated by Zod
  return this.firebaseService.query(params);
}
```

---

## 🔗 Integration Patterns

### Pattern: Compose Multiple Servers (Python)

```python
from fastmcp import FastMCP

# Core server
core = FastMCP("Core")

@core.tool()
def core_op():
    return "core"

# Firebase skill
firebase = FastMCP("Firebase")

@firebase.tool()
def firebase_op():
    return "firebase"

# Supabase skill
supabase = FastMCP("Supabase")

@supabase.tool()
def supabase_op():
    return "supabase"

# Main server that composes all
main = FastMCP("Main")
main.import_server(core)  # Import all tools
main.mount("/firebase", firebase)  # Mount under prefix
main.mount("/supabase", supabase)

if __name__ == "__main__":
    main.run()
```

This pattern aligns perfectly with your project's SEPARATE principle - load skills on-demand!
