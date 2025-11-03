# File Operations Tools

**Tools Covered**: Read, Edit, MultiEdit, Write

## Overview

File operations form the foundation of Claude Code's code modification capabilities. Understanding how these tools work internally is critical for optimization.

## Read Tool

### How It Works

**Purpose**: Retrieve file contents from the local filesystem

**Technical Implementation**:
- Uses Node.js `fs.readFile` under the hood
- Formats output using `cat -n` style with line numbers
- **No smart indexing or symbol parsing**
- **No project knowledge database** (unlike Cursor)
- Each read is independent - no caching or memory between calls

**Line Number Format**:
```
     1→# First line of file
     2→import { something } from 'somewhere'
     3→
     4→function example() {
   ...
```

Format: `spaces + line_number + tab (→) + actual_content`

**CRITICAL**: The line number prefix is NOT part of the file content. When using Edit tool, do NOT include this prefix in old_string or new_string.

### Parameters

```typescript
{
  file_path: string;      // REQUIRED: Absolute path only (not relative)
  offset?: number;        // Optional: Starting line number
  limit?: number;         // Optional: Number of lines to read
}
```

### Capabilities & Limits

**Default Behavior**:
- Reads first 2000 lines
- Lines longer than 2000 characters are truncated
- Returns numbered output starting at line 1

**Multimodal Support**:
- **Images** (PNG, JPG): Displays visually to the model
- **PDFs**: Processes page-by-page, extracting text and visuals
- **Jupyter Notebooks**: Use NotebookRead instead (returns cells with outputs)
- **Text files**: Standard line-numbered output

**Partial Reads** (for large files):
```typescript
// Read lines 100-200 of a large file
Read({
  file_path: "/path/to/large/file.ts",
  offset: 100,
  limit: 100
})
```

### Best Practices for Token Efficiency

**DO**:
- ✅ Use offset/limit for large files to avoid reading entire file
- ✅ Read multiple files in parallel when independent
- ✅ Read specific sections when you know the line numbers

**DON'T**:
- ❌ Read entire 5000+ line files when you only need a function
- ❌ Re-read files unnecessarily (remember what you've read)
- ❌ Use Read for directory listings (use Bash `ls` or Glob instead)

### Safety Enforcements

**Read-Before-Write Validation**:
- System tracks which files have been read in current session
- Write and Edit operations FAIL if file exists but hasn't been read
- This prevents accidental overwrites

**Why this matters**: Forces explicit awareness of file state

### Token Cost Analysis

**Approximate costs** (based on typical code files):
- Small file (100 lines): ~400 tokens
- Medium file (500 lines): ~2,000 tokens
- Large file (2000 lines): ~8,000 tokens
- Reading 10 medium files: ~20,000 tokens (~$0.06 with Sonnet 4)

**Optimization strategy**: Only read what you need

## Edit Tool

### How It Works

**Purpose**: Perform exact string replacements in files

**Technical Implementation**:
- **NOT regex-based** - uses exact string matching
- **NOT symbol-aware** - does not parse AST or use compiler
- **NOT smart refactoring** - just find-and-replace
- Operates on the file's text content directly
- Strips line number prefixes before matching

**Mechanism**:
1. Read file content (internally, or verify previous Read)
2. Search for old_string
3. If found exactly once (or replace_all=true): replace
4. If not found or multiple matches without replace_all: FAIL
5. Write updated content back

### Parameters

```typescript
{
  file_path: string;      // REQUIRED: Absolute path
  old_string: string;     // REQUIRED: Exact text to find
  new_string: string;     // REQUIRED: Replacement text
  replace_all?: boolean;  // Optional: Replace all occurrences (default: false)
}
```

### Critical Requirements

**MUST Read First**:
```typescript
// This FAILS:
Edit({ file_path: "file.ts", old_string: "foo", new_string: "bar" })

// This SUCCEEDS:
Read({ file_path: "file.ts" })
Edit({ file_path: "file.ts", old_string: "foo", new_string: "bar" })
```

**Uniqueness Requirement**:
```typescript
// If old_string appears 3 times in file, this FAILS:
Edit({ file_path: "file.ts", old_string: "const x", new_string: "const y" })

// Solutions:
// 1. Make string more specific (unique context)
Edit({
  file_path: "file.ts",
  old_string: "const x = 5;\n  return x;",
  new_string: "const y = 5;\n  return y;"
})

// 2. Use replace_all if you want to change all
Edit({
  file_path: "file.ts",
  old_string: "const x",
  new_string: "const y",
  replace_all: true
})
```

**Whitespace Sensitivity**:
- Tabs vs spaces matter
- Line endings matter (\n vs \r\n)
- Indentation must match EXACTLY

```typescript
// From Read output:
//      45→    function example() {

// WRONG - includes line number:
old_string: "     45→    function example() {"

// CORRECT - actual content only:
old_string: "    function example() {"
```

### When Full File Rewrites Happen

Claude may choose to use Write instead of Edit when:
1. **Too many changes needed**: 5+ separate edits might trigger a rewrite
2. **Structural changes**: Adding/removing many imports, reorganizing
3. **Edit failures**: If Edit attempts fail, falls back to Write
4. **Formatting changes**: Changing indentation across entire file

### How to Avoid Full Rewrites

**Strategy 1: Use MultiEdit**
```typescript
// Instead of 5 separate Edit calls, use one MultiEdit:
MultiEdit({
  file_path: "file.ts",
  edits: [
    { old_string: "foo", new_string: "bar" },
    { old_string: "baz", new_string: "qux" },
    // ... up to ~10 changes
  ]
})
```

**Strategy 2: Make strings unique**
```typescript
// Bad - might match multiple places:
old_string: "const user"

// Good - specific context:
old_string: "const user = await getUser();\n  if (!user) {"
```

**Strategy 3: Request minimal changes**
```
User: "Rename userId to accountId in the getAccount function"
// Claude will likely Edit

User: "Refactor the entire authentication system"
// Claude will likely Write new files
```

### Replace-All Capability

**Use Cases**:
- Renaming a variable throughout a file
- Changing all occurrences of a pattern
- Find-and-replace operations

**Example**:
```typescript
// Rename all instances of 'userId' to 'accountId'
Edit({
  file_path: "auth.ts",
  old_string: "userId",
  new_string: "accountId",
  replace_all: true
})
```

**Limitations**:
- Still exact string match (not regex)
- Changes ALL occurrences (no selective replacement)
- Cannot use replace_all for different context-dependent changes

### Best Practices for Token Efficiency

**DO**:
- ✅ Use specific, unique strings to avoid failures
- ✅ Include surrounding context for uniqueness
- ✅ Use replace_all for variable renaming
- ✅ Prefer Edit over Write when possible (cheaper)

**DON'T**:
- ❌ Use overly generic strings that match multiple times
- ❌ Include line number prefixes from Read output
- ❌ Make tiny 1-character edits when batching is better
- ❌ Use Edit for structural changes (use Write)

### Failure Modes & Recovery

**Common Failures**:

1. **"old_string not found"**
   - File was modified since Read
   - String doesn't match exactly (whitespace issue)
   - Line number prefix included in old_string
   - **Solution**: Re-read file, verify exact match

2. **"old_string matches multiple times"**
   - String too generic
   - **Solution**: Add more context or use replace_all

3. **"Must read file first"**
   - Haven't called Read yet
   - **Solution**: Read file before Edit

### Token Cost Analysis

**Edit operation costs** (input tokens):
- Tool name + parameters: ~50 tokens
- old_string (typical): 20-100 tokens
- new_string (typical): 20-100 tokens
- **Total per Edit**: ~90-250 tokens

**Comparison**:
- 1 Edit: ~150 tokens
- 5 Edits: ~750 tokens
- 1 MultiEdit with 5 changes: ~400 tokens
- 1 Write (full file, 200 lines): ~1000 tokens

**Optimization**: Use MultiEdit for multiple changes

## MultiEdit Tool

### How It Works

**Purpose**: Make multiple find-and-replace operations on one file in a single atomic operation

**Technical Implementation**:
- Processes edits sequentially in array order
- Earlier edits affect later edits (string positions shift)
- All edits succeed or none apply (atomic transaction)
- More efficient than multiple Edit calls

### Parameters

```typescript
{
  file_path: string;      // REQUIRED: Absolute path
  edits: Array<{          // REQUIRED: Array of edit operations
    old_string: string;
    new_string: string;
    replace_all?: boolean;
  }>;
}
```

### Sequential Processing Behavior

**CRITICAL**: Edits apply in order, affecting subsequent matches

```typescript
// File content: "const foo = 1; const foo = 2;"

MultiEdit({
  file_path: "file.ts",
  edits: [
    { old_string: "const foo = 1", new_string: "const bar = 1" },
    { old_string: "const foo = 2", new_string: "const baz = 2" }
  ]
})

// Result: "const bar = 1; const baz = 2;"
// ✅ Works because first edit doesn't affect second match

// But this FAILS:
MultiEdit({
  file_path: "file.ts",
  edits: [
    { old_string: "const foo", new_string: "const bar", replace_all: true },
    { old_string: "const foo = 2", new_string: "const baz = 2" }
  ]
})
// ❌ Fails because first edit removes "const foo",
//    so second edit can't find its old_string
```

### When to Use MultiEdit vs Multiple Edits

**Use MultiEdit when**:
- ✅ Making 2-10 related changes in one file
- ✅ Changes are independent (don't overlap)
- ✅ Want atomic behavior (all or nothing)
- ✅ Optimizing for token efficiency

**Use separate Edits when**:
- ❌ Changes are in different files
- ❌ Need to check results between edits
- ❌ Edits might conflict (need to handle failures separately)

### Best Practices

**DO**:
- ✅ Order edits from bottom-to-top of file (later line numbers first)
- ✅ Make edits independent of each other
- ✅ Use for related changes (e.g., updating all parameters of a function)
- ✅ Batch 2-10 changes for efficiency

**DON'T**:
- ❌ Create edits that depend on each other's string content
- ❌ Mix replace_all with specific replacements of the same string
- ❌ Use for unrelated changes scattered across file

### Token Cost Analysis

**Efficiency gains**:
```
5 separate Edit calls:
- 5 tool invocations × 150 tokens = 750 tokens

1 MultiEdit with 5 changes:
- 1 tool invocation + 5 edit specs = ~400 tokens

Savings: ~47%
```

## Write Tool

### How It Works

**Purpose**: Create new files or completely overwrite existing files

**Technical Implementation**:
- Writes content to specified path
- Creates parent directories if needed (in some implementations)
- Requires Read before overwriting existing files
- No safety checks beyond read-before-write

### Parameters

```typescript
{
  file_path: string;      // REQUIRED: Absolute path
  content: string;        // REQUIRED: Complete file content
}
```

### When Claude Chooses Write vs Edit

**Write is preferred for**:
- Creating brand new files
- Complete file restructuring
- Changing >30% of file content
- Formatting changes across entire file
- When Edit operations would be too complex

**Edit is preferred for**:
- Small, targeted changes
- Changing <30% of file content
- Preserving most of existing file
- When change is clearly defined

### Best Practices

**DO**:
- ✅ Use for new file creation
- ✅ Use when restructuring entire file
- ✅ Read existing file first if it exists
- ✅ Ensure content is complete and correct

**DON'T**:
- ❌ Use Write for small changes (use Edit)
- ❌ Overwrite files without reading first
- ❌ Write to paths with insufficient permissions

### Token Cost Analysis

**Write costs**:
- Tool name + parameters: ~30 tokens
- Content (typical 200-line file): ~1000 tokens
- **Total**: ~1030 tokens

**Comparison to alternatives**:
- Write 200-line file: ~1030 tokens
- Edit 5 places in 200-line file: ~750 tokens (5 × 150)
- Read + Write: ~1800 tokens (800 read + 1000 write)

**Optimization**: Prefer Edit for small changes

## Compiler / Symbol Integration

### What's NOT Built-In

Claude Code does NOT have:
- ❌ AST (Abstract Syntax Tree) parsing
- ❌ Compiler integration
- ❌ Symbol table or index
- ❌ "Find all references" capability
- ❌ Semantic understanding of code structure
- ❌ Automatic refactoring engine
- ❌ Project knowledge database (like Cursor)

### Why This Matters

**All file operations are text-based**:
```typescript
// Claude CANNOT do:
"Find all references to function getUserId and rename to getAccountId"
// (without reading all files and searching text)

// Claude CAN do:
Edit({
  file_path: "auth.ts",
  old_string: "function getUserId(",
  new_string: "function getAccountId(",
})
```

### Recommended: IDE Integration via MCP

From the project spec (`.plan/initial-design/SPEC.md`):

```typescript
// Proposed MCP server: optimized-ai-ide
ide.renameSymbol(oldName, newName)     // Use IDE's refactoring
ide.organizeImports(file)               // Clean up imports
ide.formatDocument(file)                // Format with project settings
ide.showReferences(symbol)              // Find all references
ide.findDefinition(symbol)              // Go to definition
```

**Benefits**:
- Leverage IDE's symbol parsing (VS Code, Cursor, etc.)
- Semantic refactoring (not text-based)
- Faster and more reliable
- No need to recreate compiler functionality

**Implementation priority**: Phase 2-3 (after experimental framework)

## File Operations Workflow

### Recommended Pattern

```typescript
// 1. Search for files
Glob({ pattern: "**/*.ts" })

// 2. Search content
Grep({ pattern: "getUserId", glob: "*.ts" })

// 3. Read relevant files (in parallel)
Promise.all([
  Read({ file_path: "auth.ts" }),
  Read({ file_path: "user-service.ts" })
])

// 4. Make changes
MultiEdit({
  file_path: "auth.ts",
  edits: [
    { old_string: "getUserId", new_string: "getAccountId", replace_all: true }
  ]
})

// 5. Write new files if needed
Write({
  file_path: "account-service.ts",
  content: "..."
})
```

### Anti-Patterns

**❌ Read everything first**:
```typescript
// Wastes tokens on files you won't edit
Read("file1.ts")
Read("file2.ts")
Read("file3.ts")
// ... then only edit file2.ts
```

**❌ Multiple small edits**:
```typescript
// Inefficient: 5 separate Edit calls
Edit({ file_path: "file.ts", old_string: "a", new_string: "b" })
Edit({ file_path: "file.ts", old_string: "c", new_string: "d" })
// ... use MultiEdit instead
```

**❌ Write when Edit would work**:
```typescript
// Inefficient: Write entire file for 1-line change
Read("file.ts")  // 1000 tokens
Write("file.ts", newContent)  // 1000 tokens
// Total: 2000 tokens

// Efficient: Edit the line
Read("file.ts")  // 1000 tokens
Edit({ old_string: "...", new_string: "..." })  // 150 tokens
// Total: 1150 tokens
// Savings: 43%
```

## Summary: Optimization Strategies

### Token Minimization

1. **Read selectively**: Use offset/limit for large files
2. **Batch operations**: Use MultiEdit for multiple changes
3. **Prefer Edit over Write**: For small changes
4. **Parallel reads**: Read independent files simultaneously
5. **Avoid re-reading**: Remember what you've read

### Reliability

1. **Make strings unique**: Include context for Edit
2. **Read before write**: Always for existing files
3. **Use replace_all carefully**: Only when you want all occurrences
4. **Handle failures**: Re-read and retry with more context

### Context Isolation (aligns with SEPARATE principle)

1. **Subagents for large operations**: Use Task tool to spawn subagent for file operations
2. **Skills for file patterns**: Load file-operation skills on demand
3. **Hooks for automation**: Post-edit formatting, linting

---

**Next**: [Search & Discovery Tools](./02-SEARCH-DISCOVERY.md)
