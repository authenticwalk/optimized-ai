# Search & Discovery Tools

**Tools Covered**: Glob, Grep

## Overview

Search tools help Claude discover files and content without reading entire codebases. Efficient use of these tools is critical for token minimization.

## Glob Tool

### How It Works

**Purpose**: Fast file pattern matching using glob patterns

**Technical Implementation**:
- Uses file system glob matching (not regex)
- Returns results sorted by **modification time** (most recent first)
- Works with any codebase size
- No content reading - just file paths

**Pattern Syntax** (glob, not regex):
- `*` - Matches any characters except `/`
- `**` - Matches any characters including `/` (recursive)
- `?` - Matches single character
- `[abc]` - Matches any character in brackets
- `{a,b}` - Matches either pattern

### Parameters

```typescript
{
  pattern: string;        // REQUIRED: Glob pattern
  path?: string;          // OPTIONAL: Directory to search (defaults to cwd)
}
```

**IMPORTANT**: Omit `path` parameter to use default directory. Do NOT enter "undefined" or "null" - simply omit the field.

### Common Patterns

```typescript
// All TypeScript files in project
Glob({ pattern: "**/*.ts" })

// All test files
Glob({ pattern: "**/*.test.ts" })
Glob({ pattern: "**/*.spec.ts" })

// Files in specific directory
Glob({ pattern: "src/**/*.tsx" })

// Multiple extensions
Glob({ pattern: "**/*.{ts,tsx,js,jsx}" })

// Specific filename anywhere in project
Glob({ pattern: "**/auth.ts" })

// Files starting with 'user'
Glob({ pattern: "**/user*.ts" })
```

### Sorting Behavior

**Most Recent First**:
```typescript
// Returns files in order of modification time
Glob({ pattern: "**/*.ts" })
// Result: [
//   "src/recently-edited.ts",      // Modified 5 min ago
//   "src/another-file.ts",          // Modified 1 hour ago
//   "src/old-file.ts"               // Modified 1 week ago
// ]
```

**Why this matters**:
- Recently modified files are likely relevant
- Can stop reading results when you hit old files
- Helps find "what changed recently"

### Best Practices for Token Efficiency

**DO**:
- ✅ Use specific patterns to limit results
- ✅ Start specific, broaden if no results
- ✅ Combine with Grep to find content before reading
- ✅ Run multiple Glob searches in parallel

**DON'T**:
- ❌ Use overly broad patterns (`**/*`)
- ❌ Search when you know the exact file path (just Read it)
- ❌ Use for content search (use Grep instead)

### Token Cost Analysis

**Glob is cheap**:
- Tool invocation: ~30 tokens
- Results (100 file paths): ~200 tokens
- **Total**: ~230 tokens

**Comparison**:
- Glob for files + Read specific ones: ~500 tokens
- Read all files to find the right one: ~10,000+ tokens

**Savings**: 95%+ when searching large codebases

### Parallel Search Pattern

**Efficient approach**:
```typescript
// Search multiple patterns simultaneously
Promise.all([
  Glob({ pattern: "**/*.ts" }),
  Glob({ pattern: "**/*.tsx" }),
  Glob({ pattern: "**/*.test.ts" })
])
```

### Limitations

**Cannot do**:
- Search file contents (use Grep)
- Use regex patterns (glob syntax only)
- Filter by file size, date, or other metadata
- Return file contents (only paths)

## Grep Tool

### How It Works

**Purpose**: Content search using ripgrep with regex support

**Technical Implementation**:
- Powered by ripgrep (rg) under the hood
- Supports full regex syntax
- Can filter by file type or glob pattern
- Three output modes: content, files_with_matches, count
- **Known reliability issue**: ~50% failure rate in some scenarios

**Pattern Syntax**: Full regex (not glob)
- `.*` - Match any characters
- `\s` - Whitespace
- `\w` - Word character
- `(pattern1|pattern2)` - Alternation
- `[a-z]` - Character class
- etc.

### Parameters

```typescript
{
  pattern: string;              // REQUIRED: Regex pattern
  path?: string;                // Optional: File/directory to search
  output_mode?: string;         // Optional: "content" | "files_with_matches" | "count"
  glob?: string;                // Optional: Filter files (e.g., "*.ts")
  type?: string;                // Optional: File type (js, py, rust, go, etc.)
  "-i"?: boolean;               // Optional: Case insensitive
  "-n"?: boolean;               // Optional: Show line numbers (content mode only)
  "-A"?: number;                // Optional: Lines after match (content mode only)
  "-B"?: number;                // Optional: Lines before match (content mode only)
  "-C"?: number;                // Optional: Lines before and after (content mode only)
  head_limit?: number;          // Optional: Limit output lines
  multiline?: boolean;          // Optional: Pattern can span lines
}
```

### Output Modes

**1. files_with_matches (default)**:
- Returns only file paths containing matches
- Fastest, most token-efficient
- Use when you just need to know which files

```typescript
Grep({
  pattern: "getUserId",
  output_mode: "files_with_matches"
})
// Result: ["src/auth.ts", "src/user-service.ts"]
```

**2. content**:
- Returns matching lines with their content
- More expensive but shows context
- Supports -A, -B, -C for context lines

```typescript
Grep({
  pattern: "getUserId",
  output_mode: "content",
  "-n": true,    // Show line numbers
  "-C": 2        // 2 lines context before/after
})
// Result:
// src/auth.ts:45: function getUserId(token) {
// src/auth.ts:46:   return parseToken(token).userId;
```

**3. count**:
- Returns count of matches per file
- Useful for finding frequency

```typescript
Grep({
  pattern: "getUserId",
  output_mode: "count"
})
// Result:
// src/auth.ts:15
// src/user-service.ts:8
```

### File Filtering

**By Glob Pattern**:
```typescript
Grep({
  pattern: "TODO",
  glob: "*.ts"            // Only TypeScript files
})

Grep({
  pattern: "getUserId",
  glob: "**/*.{ts,tsx}"   // TypeScript and TSX
})
```

**By File Type**:
```typescript
Grep({
  pattern: "def.*test",
  type: "py"              // Only Python files
})

// Common types: js, ts, py, rust, go, java, etc.
```

### Regex Patterns

**Basic patterns**:
```typescript
// Literal string
Grep({ pattern: "getUserId" })

// Any log followed by Error
Grep({ pattern: "log.*Error" })

// Function definitions
Grep({ pattern: "function\\s+\\w+" })

// Import statements
Grep({ pattern: "import.*from" })

// Class definitions
Grep({ pattern: "class\\s+\\w+\\s+extends" })
```

**IMPORTANT**: Escape special regex characters
```typescript
// WRONG: Searching for interface{} in Go code
Grep({ pattern: "interface{}" })
// Matches "interfaceabc", "interface123", etc.

// CORRECT: Escape braces
Grep({ pattern: "interface\\{\\}" })
// Matches literal "interface{}"
```

### Multiline Mode

**Default**: Patterns match within single lines only

```typescript
// This WON'T match across lines:
Grep({ pattern: "struct \\{[\\s\\S]*?field" })

// Enable multiline mode:
Grep({
  pattern: "struct \\{[\\s\\S]*?field",
  multiline: true
})
```

**When to use**:
- Searching for multi-line code blocks
- Finding patterns that span lines
- Complex structural matches

**Warning**: Multiline mode is slower and uses more tokens

### Context Lines

**Show surrounding code**:
```typescript
// 3 lines after each match
Grep({
  pattern: "getUserId",
  output_mode: "content",
  "-A": 3
})

// 2 lines before each match
Grep({
  pattern: "getUserId",
  output_mode: "content",
  "-B": 2
})

// 2 lines before AND after
Grep({
  pattern: "getUserId",
  output_mode: "content",
  "-C": 2
})
```

**Use case**: Understanding context without reading full files

### Head Limit

**Limit output**:
```typescript
Grep({
  pattern: "TODO",
  output_mode: "content",
  head_limit: 10        // First 10 matches only
})
```

**Works across all modes**:
- content: Limits output lines
- files_with_matches: Limits file paths
- count: Limits count entries

### Best Practices for Token Efficiency

**DO**:
- ✅ Use files_with_matches mode by default (cheapest)
- ✅ Filter with glob or type to reduce results
- ✅ Use head_limit when you don't need all results
- ✅ Case-insensitive search (-i) when appropriate
- ✅ Run multiple Grep searches in parallel

**DON'T**:
- ❌ Use content mode when you just need file names
- ❌ Search without file filters (wastes time/tokens)
- ❌ Use multiline mode unless necessary
- ❌ Return unlimited results for common patterns
- ❌ Use as a Bash command (use the Grep tool)

### Known Issues & Workarounds

**Issue: ~50% Failure Rate**
- Grep tool has documented reliability problems
- Fails to find content that definitively exists
- May timeout or return incomplete results

**Workarounds**:
1. **Try multiple times**: Sometimes succeeds on retry
2. **Use narrower patterns**: More specific = more reliable
3. **Fall back to Bash + ripgrep**: `rg "pattern" --type ts`
4. **Use Glob + Read**: For critical searches

**Example**:
```typescript
// If Grep fails:
Grep({ pattern: "getUserId", glob: "*.ts" })
// Fallback to:
Bash({ command: 'rg "getUserId" --type ts --files-with-matches' })
```

**Issue: Ignores files**
- Respects .gitignore automatically
- Skips node_modules, .git, etc.

**Workaround**:
```typescript
// To search ignored files, use Bash + rg with flags:
Bash({ command: 'rg "pattern" --no-ignore --hidden' })
```

### Token Cost Analysis

**Grep costs** (approximate):

```typescript
// files_with_matches mode
Grep({ pattern: "getUserId", output_mode: "files_with_matches" })
// Input: ~50 tokens
// Output (10 file paths): ~100 tokens
// Total: ~150 tokens

// content mode
Grep({ pattern: "getUserId", output_mode: "content", "-C": 2 })
// Input: ~60 tokens
// Output (50 matches with context): ~1000 tokens
// Total: ~1060 tokens

// count mode
Grep({ pattern: "getUserId", output_mode: "count" })
// Input: ~50 tokens
// Output (counts): ~50 tokens
// Total: ~100 tokens
```

**Optimization**:
1. Use files_with_matches first
2. Read specific files if needed
3. Total: ~150 + (3 × 800) = ~2550 tokens

vs.

1. Read all files to search content
2. Total: ~20,000 tokens

**Savings**: 87%

### Parallel Search Pattern

**Efficient approach**:
```typescript
// Search for multiple patterns simultaneously
Promise.all([
  Grep({ pattern: "getUserId", glob: "*.ts" }),
  Grep({ pattern: "getAccountId", glob: "*.ts" }),
  Grep({ pattern: "auth", glob: "*.ts" })
])
```

## Search Strategy: Glob → Grep → Read

### Recommended Workflow

**1. Find Files (Glob)**
```typescript
// Start broad but focused
Glob({ pattern: "src/**/*.ts" })
// Returns: 50 TypeScript files
```

**2. Find Content (Grep)**
```typescript
// Search within those files
Grep({
  pattern: "getUserId",
  glob: "src/**/*.ts",
  output_mode: "files_with_matches"
})
// Returns: 3 files containing pattern
```

**3. Read Specific Files**
```typescript
// Only read files that matched
Read({ file_path: "src/auth.ts" })
Read({ file_path: "src/user-service.ts" })
Read({ file_path: "src/utils.ts" })
// Returns: ~2400 tokens total
```

**Total cost**: ~150 (Grep) + ~2400 (Read) = ~2550 tokens

**vs. Reading all 50 files**: ~40,000 tokens

**Savings**: 93%

### Alternative: Agent Tool for Complex Searches

**When to use Agent instead**:
- Open-ended exploration ("find all authentication logic")
- Multiple rounds of search needed
- Ambiguous search terms
- Need to understand codebase structure

**Example**:
```typescript
// Instead of:
Glob({ pattern: "**/*auth*" })
Grep({ pattern: "auth" })
Grep({ pattern: "login" })
Grep({ pattern: "authenticate" })
// ... iterative searching

// Use:
Task({
  subagent_type: "Explore",
  prompt: "Find all authentication-related code in the codebase. Look for auth functions, login flows, token handling, and session management.",
  description: "Explore authentication code"
})
```

**Benefits**:
- Subagent explores autonomously
- Fresh context window
- Can iterate without main context bloat
- Returns comprehensive report

## Summary: Search Optimization Strategies

### Token Minimization

1. **Glob before Grep**: Find files first, then search content
2. **files_with_matches mode**: Default for Grep
3. **Filter early**: Use glob/type parameters
4. **Limit results**: Use head_limit when appropriate
5. **Parallel searches**: Run independent searches together

### Reliability

1. **Handle Grep failures**: Retry or fall back to Bash
2. **Specific patterns**: More specific = more reliable
3. **Test patterns**: Verify regex matches what you expect
4. **Use Agent for uncertainty**: When you don't know what to search for

### Context Isolation (SEPARATE principle)

1. **Subagents for exploration**: Use Task with Explore agent
2. **Keep searches focused**: Don't pollute main context
3. **Skills for search patterns**: Load domain-specific search skills

---

**Next**: [Execution & System Tools](./03-EXECUTION-SYSTEM.md)
