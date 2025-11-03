# MCP Servers Research

## What We Researched

Explored two major repositories:
1. **awesome-mcp-servers** (74.2k stars) - Comprehensive collection of 25+ categories
2. **claude-flow** (3,058 commits) - Enterprise orchestration platform

Also investigated official MCP servers, community alternatives, and the MCP vs CLI debate.

## Why We're Researching This

Our project (Optimized AI) needs to determine which MCP servers are necessary for our self-learning AI coding assistant. Per **ADR-003**, we prefer direct CLI tools over MCP middleware, only using MCP when actually necessary.

**Alignment with our goals:**
- Self-learning system needs memory/knowledge storage (`.ai-knowledge/`)
- IDE integration for refactoring operations (ADR-010)
- GitHub PR workflow for code review
- Firebase/Supabase infrastructure support
- Minimal overhead, clean separation (Principles 1 & 2)

## Critical Finding: CLI > MCP for Most Cases

**Research consensus from 2025:**
> "There's little benefit of using an MCP compared to telling your coding agent to use its shell tool to run the CLI directly, and most often the MCP will lead to much worse results than just letting the agent run the command line tool directly."

**When MCP Makes Sense:**
- No easy-to-use CLI tools exist
- Stateful operations required (MCP servers are stateful by default)
- Safety constraints needed
- Complex workflows requiring context

**When CLI is Better (our default):**
- Straightforward tasks
- Existing CLI tools available (`git`, `gh`, `firebase`, `supabase`, `npm`)
- Want control and transparency
- Avoid context pollution

**MCP Problems:**
- Can flood context window with unnecessary output
- Many MCPs offer dozens of tools, degrading performance
- Context pollution from irrelevant capabilities

## What We Explored

Based on our project needs, we focused on:
- [Memory/Knowledge Graph Servers](./memory-knowledge-servers.md)
- [GitHub Integration](./github-integration.md)
- [IDE/VSCode Integration](./vscode-integration.md)
- [Database Servers](./database-servers.md)
- [Supabase MCP Server](./supabase-mcp.md)

## What We Skipped

- **claude-flow platform** - Heavy orchestration system (we're building our own)
- **Web scraping/browser automation** - Not core to our use case
- **Creative/multimedia tools** - Not relevant
- **Enterprise cloud services** - Beyond initial scope
- **Hundreds of niche servers** - Not aligned with our minimal approach

---

## Categorization: What We Actually Need

### 🔴 **ABSOLUTELY NEED**

#### 1. **Memory/Knowledge Graph Server**
**Need Level:** CRITICAL

**Why:** Our learning system (Principle 4) requires persistent storage of patterns, failures, preferences, and learnings across sessions.

**Options:**
- **Official Anthropic Memory Server** (`@modelcontextprotocol/server-memory`)
  - ✅ Official, actively maintained (Nov 2024)
  - ✅ Entities, relations, observations model
  - ✅ Local knowledge graph
  - ✅ Works with Claude Code

**Decision:** Use official Anthropic memory server for `.ai-knowledge/` learning storage.

**CLI Alternative:** None suitable for knowledge graph operations.

---

### 🟡 **SHOULD CONSIDER**

#### 2. **GitHub MCP Server**
**Need Level:** MEDIUM

**Why:** PR workflow (ADR-014) for code review and comments.

**Official Server:**
- **github/github-mcp-server** (Go, 438 commits, actively maintained)
  - ✅ Repository management
  - ✅ Issues & PR automation
  - ✅ CI/CD intelligence
  - ✅ Code analysis
  - ✅ Official GitHub support

**CLI Alternative:** `gh` CLI
- ✅ Mature, battle-tested
- ✅ All GitHub operations
- ✅ Scriptable
- ❌ Requires explicit command syntax
- ❌ No multi-step agent workflows

**Recommendation:** **Start with `gh` CLI** (ADR-003), only add MCP if we need complex multi-step PR workflows that benefit from stateful context.

**Trade-off:**
- MCP: Better for conversational, multi-step operations
- CLI: Better for explicit, transparent operations

---

#### 3. **VSCode/IDE Integration**
**Need Level:** MEDIUM-HIGH

**Why:** ADR-010 requires IDE operations (rename symbol, organize imports, format, run tests, etc.)

**Native Support (2025):**
- ✅ VSCode 1.102+ has **native MCP support** (generally available)
- ✅ Full MCP specification support
- ✅ Setup via `MCP: Add Server` command
- ✅ `.vscode/mcp.json` for workspace-specific servers

**Community Server:**
- **vscode-mcp-server** (juehang)
  - Read workspace structure
  - Get linter problems
  - Edit files
  - TypeScript implementation

**Our Approach:**
1. **Phase 1:** Build VSCode extension per ADR-010 using **VSCode API directly**
2. **Phase 2:** If needed, expose operations via MCP using VSCode's native support
3. **Avoid:** Creating custom MCP middleware when VSCode API suffices

**Recommendation:** **VSCode Extension API > MCP** for IDE operations. Use MCP only if cross-IDE support needed.

---

#### 4. **Supabase MCP Server**
**Need Level:** LOW-MEDIUM

**Why:** User uses Supabase for infrastructure (SPEC.md).

**Official Server:**
- **supabase-community/supabase-mcp** (Launched April 2025, actively maintained)
  - ✅ 20+ tools for database management
  - ✅ Natural language schema updates
  - ✅ Secure read-only queries
  - ✅ OAuth 2 flow
  - ✅ Integrates with Claude, Cursor, VS Code
  - ⚠️ Recommended for dev projects, not production

**CLI Alternative:** `supabase` CLI
- ✅ All Supabase operations
- ✅ Database migrations
- ✅ Function deployment
- ✅ Local development

**Recommendation per ADR-011:** **Use Supabase CLI directly** via skills/scripts. Only add MCP if natural language database operations prove valuable during development.

**Example (from ADR-011):**
```yaml
# supabase-debug.skill
scripts:
  db-dump: supabase db dump
  list-functions: supabase functions list
  check-migrations: supabase db diff
  test-connection: supabase status
```

---

#### 5. **Database Servers (SQLite/PostgreSQL)**
**Need Level:** LOW

**Why:** SPEC mentions SQLite for complex queries on knowledge base.

**Status:**
- ❌ **Official servers ARCHIVED** (moved to servers-archived, no longer maintained)

**Community Alternatives:**
- **executeautomation/mcp-database-server** - Multi-DB (SQLite, PostgreSQL, MySQL, SQL Server)
- **panasenco/mcp-sqlite** - SQLite-specific, Datasette-compatible
- **jparkerweb/mcp-sqlite** - Comprehensive SQLite CRUD

**CLI Alternative:** `sqlite3` command-line tool
- ✅ Direct queries
- ✅ Schema inspection
- ✅ Scriptable
- ✅ No dependencies

**Recommendation:** **Use `sqlite3` CLI** directly. Our knowledge base is JSON files (ADR-002), SQLite only for "complex queries" if needed. Don't add MCP unless proven necessary.

---

### 🟢 **PROBABLY DON'T NEED**

#### 6. **Filesystem Server**
**Need Level:** VERY LOW

**Why:** Official server provides "secure file operations with configurable access controls."

**Our Context:**
- ✅ Claude Code has direct file access via Read/Write/Edit tools
- ✅ We manage `.ai-knowledge/` folder directly
- ✅ No need for additional abstraction

**Recommendation:** **Don't add.** Use Claude Code's native file tools.

---

#### 7. **Git Server**
**Need Level:** VERY LOW

**Why:** Official server offers "tools to read, search, and manipulate Git repositories."

**CLI Alternative:** `git` command
- ✅ Universal, standard
- ✅ All git operations
- ✅ Already familiar

**Recommendation:** **Don't add.** Use `git` CLI directly.

---

#### 8. **Firebase MCP Server**
**Need Level:** VERY LOW

**Why:** User uses Firebase for infrastructure.

**Status:**
- ❌ **No official Firebase MCP server found**

**CLI Alternative:** `firebase` CLI
- ✅ All Firebase operations
- ✅ Auth, Firestore, Functions, Hosting
- ✅ Official Google support

**Recommendation per ADR-011:** **Use Firebase CLI directly** via skills/scripts.

**Example (from ADR-011):**
```yaml
# firebase-debug.skill
scripts:
  list-projects: firebase projects:list
  query-firestore: firebase firestore:get $COLLECTION --limit 10
  check-auth: firebase auth:export auth-dump.json
  validate-rules: firebase deploy --only firestore:rules --dry-run
  test-function: firebase functions:shell
```

---

### 🔴 **DEFINITELY DON'T NEED**

#### 9. **claude-flow Platform**
**Need Level:** NONE

**What it is:** Enterprise-grade AI orchestration platform with:
- Multi-agent swarm coordination
- 25 specialized skills
- 100+ MCP tools
- Semantic search (AgentDB)
- Persistent memory (ReasoningBank)

**Why we don't need it:**
- ❌ We're building our own orchestration system
- ❌ Too heavy, conflicts with Principle 1 (MINIMIZE)
- ❌ Duplicates functionality we're implementing
- ❌ Not aligned with our clean, minimal approach

**Recommendation:** **Don't add.** Build our own system per SPEC.md.

---

## Summary: Final Recommendations

### **Install Now**

1. **Memory/Knowledge Graph Server**
   ```bash
   npm install -g @modelcontextprotocol/server-memory
   ```
   **Why:** Critical for learning system, no CLI alternative.

### **Evaluate Later (After Phase 2)**

2. **GitHub MCP Server** - If `gh` CLI proves insufficient for PR workflows
3. **Supabase MCP Server** - If natural language DB operations add value

### **Use CLI Instead**

4. **Firebase** → `firebase` CLI (scripts in skills)
5. **Supabase** → `supabase` CLI (scripts in skills)
6. **Git** → `git` command
7. **Database** → `sqlite3` CLI if needed
8. **Filesystem** → Claude Code native tools

### **Don't Add**

9. **claude-flow** - Building our own system
10. **Filesystem Server** - Native tools sufficient
11. **Git Server** - CLI better

---

## Implementation Strategy

### Phase 0: Foundation
1. Install **Anthropic Memory Server** for knowledge graph
2. Configure in Claude Code settings
3. Test with `.ai-knowledge/` learning storage

### Phase 1: Skills with CLI
1. Create Firebase skill with CLI scripts (ADR-011)
2. Create Supabase skill with CLI scripts (ADR-011)
3. Test effectiveness vs MCP approach

### Phase 2: Evaluate MCP Additions
1. Track pain points with CLI approach
2. If complex workflows emerge, consider:
   - GitHub MCP for PR automation
   - Supabase MCP for dev database operations
3. Make data-driven decision (Principle 3: VALIDATE)

### Phase 3: VSCode Extension
1. Build extension using VSCode API (ADR-010)
2. If cross-tool support needed, expose via MCP
3. Use VSCode's native MCP support

---

## Validation Against Our Principles

### ✅ **Principle 1: MINIMIZE**
- Only one MCP server initially (memory)
- Prefer CLI over middleware
- Avoid context pollution

### ✅ **Principle 2: SEPARATE**
- CLI scripts in skills (loaded on-demand)
- MCP only when no CLI alternative
- Clean boundaries

### ✅ **Principle 3: VALIDATE**
- Will test CLI approach first
- Measure effectiveness before adding MCP
- Evidence-based decisions

### ✅ **Principle 4: LEARN**
- Memory server enables learning system
- Track which approach works better
- Adapt based on data

---

## Key Learnings

### Learning 1: MCP is Often Overkill
**Discovery:** Research shows CLI tools outperform MCP for most operations.

**Evidence:**
> "Most often the MCP will lead to much worse results than just letting the agent run the command line tool directly."

**Application:** Default to CLI, only add MCP when proven necessary.

### Learning 2: Official vs Community Servers
**Discovery:** Official servers (GitHub, Anthropic) are actively maintained. Many community servers are experimental.

**Application:** Prefer official servers when available. Vet community servers carefully.

### Learning 3: VSCode Native MCP Support
**Discovery:** VSCode 1.102+ has native MCP support, making integration seamless.

**Application:** Don't build custom MCP middleware for VSCode. Use native support.

### Learning 4: Database Servers are Archived
**Discovery:** Official SQLite/PostgreSQL MCP servers are no longer maintained.

**Application:** Use CLI tools. Community servers are risky without active maintenance.

### Learning 5: Supabase > Firebase for MCP
**Discovery:** Supabase has official MCP support (April 2025). Firebase has none.

**Application:** If we need database MCP, Supabase has official option. Firebase uses CLI only.

---

## Comparison to Project Goals

**From SPEC.md:**
> "A self-learning AI coding assistant system that enables you to work as a PM while AI handles implementation."

**MCP Alignment:**
- ✅ **Memory Server** - Critical for self-learning
- ⚠️ **GitHub Server** - Useful for PR workflow but `gh` CLI may suffice
- ⚠️ **IDE Integration** - VSCode API better than MCP initially
- ❌ **Database Servers** - CLI tools sufficient per ADR-011
- ❌ **claude-flow** - Conflicts with building our own system

**From ADR-003:**
> "Prefer direct CLI scripts over MCP middleware. Only use MCP when actually necessary."

**Our Recommendations:** ✅ Fully aligned. Only one MCP (memory) required.

---

## Activity & Maintenance Status

### Active & Maintained (2025)
- ✅ **Anthropic Memory Server** (Official, Nov 2024)
- ✅ **GitHub MCP Server** (Official, 438 commits)
- ✅ **Supabase MCP Server** (Official, April 2025)
- ✅ **VSCode Native MCP** (GA in VSCode 1.102)

### Archived / Inactive
- ❌ **Official SQLite/PostgreSQL** (Moved to servers-archived)
- ⚠️ **Community SQLite servers** (Various, check individually)

### Heavy / Not Suitable
- ❌ **claude-flow** (Active but too heavy for our use case)

---

## Next Steps

1. **Install memory server** and integrate with `.ai-knowledge/`
2. **Create CLI-based skills** for Firebase/Supabase (ADR-011)
3. **Track pain points** - Where does CLI fall short?
4. **Validate approach** - Measure token usage, effectiveness (Principle 3)
5. **Re-evaluate** after Phase 2 - Add MCP only if data supports it

---

## References

- [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers) - 74.2k stars
- [Official MCP Servers](https://github.com/modelcontextprotocol/servers)
- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [Supabase MCP Server](https://github.com/supabase-community/supabase-mcp)
- [MCP vs CLI Debate](https://mariozechner.at/posts/2025-08-15-mcp-vs-cli/)
- [VSCode MCP Support](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)

**Research Date:** 2025-11-03
**Researcher:** Claude
**Status:** ✅ Complete
**Decision:** Install memory server, use CLI for everything else, re-evaluate after validation
