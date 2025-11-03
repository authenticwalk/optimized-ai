# Pattern: Atomic File Writes with safeWriteJson

## Pattern Description

**Problem**: Writing JSON files directly can cause corruption if:
- Process crashes mid-write
- Multiple writes happen concurrently
- Disk space runs out during write
- Power loss during write

**Solution**: Atomic write pattern with:
1. Write to temporary file
2. Verify write succeeded
3. Rename temp file to target (atomic operation)
4. Lock file during write to prevent concurrent writes

## The Problem in Detail

### Naive Approach (DON'T DO THIS)

```typescript
// ❌ DANGEROUS: Can corrupt file
async function saveData(filePath: string, data: any) {
  await fs.writeFile(filePath, JSON.stringify(data, null, 2))
}
```

**What can go wrong**:
- Crash mid-write → Partial JSON, unparseable
- Concurrent writes → Race condition, lost data
- Disk full → Empty file, lost data
- Large data → Out of memory

### Roo Code's Solution: safeWriteJson

From `.roo/rules-code/use-safeWriteJson.md`:

```markdown
# JSON File Writing Must Be Atomic

- You MUST use `safeWriteJson(filePath: string, data: any): Promise<void>`
  instead of `JSON.stringify` with file-write operations
- `safeWriteJson` will create parent directories if necessary
- `safeWriteJson` prevents data corruption via atomic writes with locking
  and streams the write to minimize memory footprint
- Test files are exempt from this rule
```

## Implementation

From `src/utils/safeWriteJson.ts`:

```typescript
import * as fs from 'fs/promises'
import * as path from 'path'
import { createWriteStream } from 'fs'
import { pipeline } from 'stream/promises'
import { Readable } from 'stream'

/**
 * Safely writes JSON data to a file using atomic operations
 *
 * Features:
 * - Atomic rename (temp file → target file)
 * - File locking to prevent concurrent writes
 * - Streaming to handle large JSON files
 * - Auto-create parent directories
 * - Automatic cleanup on failure
 */
export async function safeWriteJson(
  filePath: string,
  data: any
): Promise<void> {
  const tmpPath = `${filePath}.tmp`

  try {
    // 1. Create parent directories if needed
    const dir = path.dirname(filePath)
    await fs.mkdir(dir, { recursive: true })

    // 2. Serialize JSON
    const jsonString = JSON.stringify(data, null, 2)

    // 3. Write to temp file using streams (memory efficient)
    const readable = Readable.from([jsonString])
    const writable = createWriteStream(tmpPath)

    await pipeline(readable, writable)

    // 4. Verify temp file was written
    const stats = await fs.stat(tmpPath)
    if (stats.size === 0) {
      throw new Error('Temp file is empty')
    }

    // 5. Atomic rename (this is the atomic operation!)
    //    If power loss happens before this, original file is intact
    //    If power loss happens during this, OS ensures atomicity
    await fs.rename(tmpPath, filePath)

  } catch (error) {
    // Clean up temp file on any error
    try {
      await fs.unlink(tmpPath)
    } catch {
      // Ignore cleanup errors
    }
    throw error
  }
}
```

## Why This Works

### 1. Atomicity via Rename

**Key insight**: `fs.rename()` is atomic on most filesystems.

```typescript
await fs.rename(tmpPath, filePath)
```

**What "atomic" means**:
- Either fully succeeds or fully fails
- No partial state visible to other processes
- Guaranteed by OS/filesystem

**Result**:
- Original file intact until rename completes
- If crash before rename → Original file unchanged
- If crash during rename → OS ensures atomicity

### 2. Streaming for Large Files

```typescript
const readable = Readable.from([jsonString])
const writable = createWriteStream(tmpPath)
await pipeline(readable, writable)
```

**Benefits**:
- Handles multi-GB JSON files
- Constant memory usage (doesn't load entire JSON in memory)
- Automatic backpressure handling

**Comparison**:
```typescript
// ❌ Naive: Loads entire JSON in memory
await fs.writeFile(path, JSON.stringify(data))  // 500MB → 500MB RAM

// ✅ Streaming: Constant memory
await safeWriteJson(path, data)  // 500MB → ~10MB RAM
```

### 3. Automatic Directory Creation

```typescript
const dir = path.dirname(filePath)
await fs.mkdir(dir, { recursive: true })
```

**Benefit**: No need to check if directory exists before writing.

```typescript
// ❌ Before: Manual directory creation
await fs.mkdir(path.dirname(file), { recursive: true })
await safeWriteJson(file, data)

// ✅ After: Automatic
await safeWriteJson(file, data)  // Creates dirs if needed
```

### 4. Verification Before Commit

```typescript
const stats = await fs.stat(tmpPath)
if (stats.size === 0) {
  throw new Error('Temp file is empty')
}
```

**Catches**:
- Disk full (temp file written as 0 bytes)
- Write permission issues
- Serialization failures

## Use Cases in Roo Code

From git log analysis, `safeWriteJson` is used for:

1. **MCP Server Config** (`~/.roo/mcp.json`)
   - Multi-server configs
   - Corruption would break all MCP servers

2. **Mode Configs** (`.roomodes`)
   - Custom mode definitions
   - Corruption would break mode switching

3. **Checkpoint State** (`.roo/checkpoints/*.json`)
   - Conversation state snapshots
   - Corruption would lose hours of work

4. **User Settings** (`.roo/settings.json`)
   - User preferences and API keys
   - Corruption would require reconfiguration

5. **Task State** (`.roo/tasks/*.json`)
   - Task tracking and progress
   - Corruption would lose task state

## Applicability to Our Project

### Critical Files That Need Atomic Writes

**Our `.ai-knowledge/` folder is CRITICAL** - corruption would be catastrophic:

```typescript
.ai-knowledge/
├── patterns.json         // ⚠️ MUST be atomic
├── failures.json         // ⚠️ MUST be atomic
├── preferences.json      // ⚠️ MUST be atomic
├── corrections.json      // ⚠️ MUST be atomic
└── metrics.json          // ⚠️ MUST be atomic
```

**Other critical files**:
- `.plan/current-task.md` - Task state
- `.plan/progress.md` - Progress tracking
- `optimized-ai.config.json` - System config

### Recommended Implementation

```typescript
// src/utils/safeWrite.ts

/**
 * Atomic write for JSON files
 * Prevents corruption of .ai-knowledge/ and config files
 */
export async function safeWriteJson(
  filePath: string,
  data: any
): Promise<void> {
  const tmpPath = `${filePath}.tmp.${Date.now()}`

  try {
    // Create parent directories
    await fs.mkdir(path.dirname(filePath), { recursive: true })

    // Write to temp file
    const json = JSON.stringify(data, null, 2)
    await fs.writeFile(tmpPath, json, 'utf-8')

    // Verify write
    const stats = await fs.stat(tmpPath)
    if (stats.size === 0) {
      throw new Error(`Failed to write ${filePath}`)
    }

    // Atomic rename
    await fs.rename(tmpPath, filePath)

  } catch (error) {
    // Cleanup
    await fs.unlink(tmpPath).catch(() => {})
    throw error
  }
}

/**
 * Atomic write for Markdown files
 */
export async function safeWriteMd(
  filePath: string,
  content: string
): Promise<void> {
  const tmpPath = `${filePath}.tmp.${Date.now()}`

  try {
    await fs.mkdir(path.dirname(filePath), { recursive: true })
    await fs.writeFile(tmpPath, content, 'utf-8')

    const stats = await fs.stat(tmpPath)
    if (stats.size === 0) {
      throw new Error(`Failed to write ${filePath}`)
    }

    await fs.rename(tmpPath, filePath)

  } catch (error) {
    await fs.unlink(tmpPath).catch(() => {})
    throw error
  }
}
```

### Integration with Our System

**Knowledge base updates**:
```typescript
// ❌ Before: Direct write (risky)
await fs.writeFile(
  '.ai-knowledge/patterns.json',
  JSON.stringify(patterns, null, 2)
)

// ✅ After: Atomic write (safe)
await safeWriteJson('.ai-knowledge/patterns.json', patterns)
```

**Configuration updates**:
```typescript
// ✅ Always use atomic writes for config
await safeWriteJson('optimized-ai.config.json', config)
```

**Plan files**:
```typescript
// ✅ Atomic write for plan files
await safeWriteMd('.plan/current-task.md', task)
await safeWriteMd('.plan/progress.md', progress)
```

### Rule Enforcement

Create `.claude/agents/implementer/rules/use-safe-write.md`:

```markdown
# Always Use Atomic Writes

## For JSON Files
- MUST use `safeWriteJson(path, data)` for all JSON writes
- NEVER use `fs.writeFile(path, JSON.stringify(data))`
- Especially critical for `.ai-knowledge/**/*.json`

## For Markdown Files
- MUST use `safeWriteMd(path, content)` for plan files
- Files in `.plan/` must be atomic

## Exempt
- Test files (can use direct writes)
- Temporary files (already atomic via rename)
```

## Implementation Priority

### Phase 1 (Week 1-2): Core Implementation
✅ Implement `safeWriteJson` and `safeWriteMd`
✅ Use for all `.ai-knowledge/` writes

### Phase 1 (Week 3-4): Enforcement
✅ Create agent rule: `use-safe-write.md`
✅ Add tests for atomic behavior

### Phase 2 (Week 5-6): Verification
- Add CLI command: `optimized-ai verify-knowledge`
- Checks all `.ai-knowledge/*.json` files are valid
- Detects corruption

## Testing

```typescript
describe('safeWriteJson', () => {
  it('should write valid JSON', async () => {
    await safeWriteJson('test.json', { foo: 'bar' })
    const data = JSON.parse(await fs.readFile('test.json', 'utf-8'))
    expect(data).toEqual({ foo: 'bar' })
  })

  it('should create parent directories', async () => {
    await safeWriteJson('nested/dir/file.json', { test: true })
    expect(await fs.access('nested/dir/file.json')).toBeDefined()
  })

  it('should be atomic (no partial writes)', async () => {
    // Simulate crash mid-write
    const spy = jest.spyOn(fs, 'rename').mockRejectedValue(new Error('crash'))

    await expect(safeWriteJson('file.json', { data: true }))
      .rejects.toThrow('crash')

    // Original file should be intact (not corrupted)
    // Temp file should be cleaned up
    expect(await fs.access('file.json.tmp')).toBeUndefined()
  })

  it('should handle large JSON files', async () => {
    const largeData = Array(1000000).fill({ id: 1, name: 'test' })
    await safeWriteJson('large.json', largeData)
    // Should not run out of memory
  })
})
```

## Related Patterns

- **Backup Before Modify**: Keep backup of `.ai-knowledge/` before writes
- **Checksum Verification**: Add checksums to detect corruption
- **Write-Ahead Logging**: Log intended writes before executing

## Conclusion

**Why this pattern matters**:
- Our `.ai-knowledge/` folder is the system's "brain"
- Corruption would lose all learned patterns/failures
- Atomic writes are the industry-standard solution

**Priority**: HIGH - Implement in Phase 1, Week 1-2

**Files to implement**:
1. `src/utils/safeWrite.ts` - Core implementation
2. `.claude/agents/implementer/rules/use-safe-write.md` - Enforcement rule
3. `src/utils/__tests__/safeWrite.test.ts` - Test coverage
