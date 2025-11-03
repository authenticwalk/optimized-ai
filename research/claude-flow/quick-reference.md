# Claude-Flow Quick Reference Card

**One-page cheat sheet for optimization patterns**

---

## 🎯 STEAL THESE 5 PATTERNS

### 1. MCP Coordination Model
```
MCP tools = Strategy/coordination
Execution tools = Actual work
DON'T mix them!
```

### 2. Hybrid Fallback
```javascript
try { useAgentDB() }        // Fast vector search
catch { useReasoningBank() } // SQLite fallback  
catch { useInMemory() }      // Last resort
```

### 3. Optional Native Dependencies
```json
"optionalDependencies": {
  "better-sqlite3": "^12.2.0",
  "onnxruntime-node": "^1.23.0"
}
```

### 4. Hooks for Automation
```
Pre-operation: Validate, prepare, assign
Post-operation: Format, update memory, train
Session: Save/restore context
```

### 5. Batch Operations
```javascript
// ✅ DO: Single message, all operations
Task("Agent 1"), Task("Agent 2"), Task("Agent 3")
Write(file1), Write(file2), Write(file3)

// ❌ DON'T: Multiple sequential messages
Message 1: Task("Agent 1")
Message 2: Task("Agent 2")
```

---

## 🚫 SKIP THESE

1. **SPARC** - Claude Code has native planning now
2. **Hive-Mind** - Over-engineered consensus
3. **Skills** - Just markdown files, not real code
4. **Flow Nexus** - Commercial distraction
5. **Performance Claims** - Unverified marketing

---

## ⚠️ RED FLAGS

- 🚩 Unverified performance claims (84.8% solve rate? No evidence)
- 🚩 Multiple deprecated systems in codebase
- 🚩 3,245-line files (complexity creep)
- 🚩 Documentation ≠ implementation
- 🚩 Marketing-first approach

---

## ✅ GREEN FLAGS

- ✅ High commit velocity (active development)
- ✅ Pragmatic learning (5 stdio fixes = iteration works)
- ✅ Resilient architecture (fallbacks everywhere)
- ✅ MIT license
- ✅ MCP integration is production-ready

---

## 📁 KEY FILES

**Study these:**
- `src/mcp/mcp-server.js` - Clean MCP implementation
- `CLAUDE.md` - Coordination vs execution model
- `src/memory/fallback-store.js` - Graceful degradation
- `package.json` - Optional dependencies pattern

**Avoid these:**
- `src/swarm/coordinator.ts` - 3,245 lines of complexity
- `src/hive-mind/` - Over-engineered consensus
- `.claude/skills/` - Just markdown

---

## 💡 ONE-SENTENCE SUMMARY

**Claude-Flow is a research lab with good architectural patterns buried in complexity - extract the validated patterns, skip the abandoned experiments.**

---

## 🎬 NEXT STEPS

1. Copy hybrid fallback pattern for resilience
2. Use optional dependencies for native modules
3. Implement event-driven hooks for automation
4. Keep MCP coordination separate from execution
5. Batch all operations in single calls
6. **DON'T** use claude-flow as a dependency
7. **DON'T** copy unverified performance claims

---

## 📊 REALITY CHECK

| Claim | Reality |
|-------|---------|
| 84.8% SWE-Bench solve | No evidence found |
| 96x-164x faster search | AgentDB marketing, not verified |
| 2-3ms ReasoningBank queries | ✅ Believable (simple SQLite) |
| 100+ MCP tools | ✅ Confirmed in code |
| Hybrid memory system | ✅ Exists and works |
| Production-ready | ⚠️ Mixed - MCP yes, swarm experimental |

---

**Use as:** Learning resource and pattern library  
**Don't use as:** Production dependency or source of truth for performance claims

