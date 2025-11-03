# CLI vs MCP: Decision Framework

## Purpose

A practical decision framework for choosing between CLI tools and MCP servers, based on our research and ADR-003.

---

## The Default Answer

**From ADR-003:**
> "Prefer direct CLI scripts over MCP middleware. Only use MCP when actually necessary."

**Default:** ✅ **Try CLI first**

**Only add MCP if:** Evidence shows it's necessary

---

## Decision Tree

```
┌─────────────────────────────────┐
│ Need to integrate external tool │
└────────────┬────────────────────┘
             │
             ▼
   ┌─────────────────────┐
   │ Does CLI tool exist? │
   └─────┬───────────┬───┘
         │ Yes       │ No
         ▼           ▼
   ┌─────────┐   ┌──────────────┐
   │ Use CLI │   │ Consider MCP │
   └─────────┘   └──────────────┘
         │
         ▼
   ┌────────────────────────────┐
   │ Is operation stateful?     │
   │ (needs context across      │
   │  multiple steps?)          │
   └─────┬──────────────────┬───┘
         │ No               │ Yes
         ▼                  ▼
   ┌─────────┐      ┌──────────────────┐
   │ Use CLI │      │ Can you script    │
   │         │      │ the stateful flow?│
   └─────────┘      └─────┬────────┬───┘
                          │ Yes    │ No
                          ▼        ▼
                    ┌─────────┐  ┌──────────┐
                    │ Use CLI │  │ Use MCP  │
                    └─────────┘  └──────────┘
```

---

## Quick Reference Table

| Factor | CLI | MCP | Notes |
|--------|-----|-----|-------|
| **Operation is simple, single-step** | ✅ | ❌ | CLI is faster, clearer |
| **Operation requires multiple steps** | ⚠️ | ✅ | Can CLI be scripted? |
| **Need stateful context** | ❌ | ✅ | MCP maintains state |
| **Want explicit control** | ✅ | ❌ | CLI shows exactly what runs |
| **Want natural language interface** | ❌ | ✅ | MCP is conversational |
| **Tool has mature CLI** | ✅ | ❌ | Use existing tooling |
| **No CLI exists** | ❌ | ✅ | MCP is only option |
| **Minimize context pollution** | ✅ | ❌ | CLI doesn't add to context |
| **Need graph operations** | ❌ | ✅ | Knowledge graphs need MCP |
| **Want transparency** | ✅ | ❌ | CLI commands are explicit |

---

## Research-Backed Guidance

### When CLI is Better (Evidence)

**From 2025 research:**
> "There's little benefit of using an MCP compared to telling your coding agent to use its shell tool to run the CLI directly, and most often the MCP will lead to much worse results than just letting the agent run the command line tool directly."

**Scenarios:**
1. **Simple operations** - Single command achieves goal
2. **Existing tooling** - Mature CLI already exists
3. **Scripted workflows** - Can script multi-step operations
4. **Explicit control** - Want to see exact commands
5. **Transparency** - Need to debug operations

**Examples:**
- `git` operations
- `firebase` operations
- `supabase` operations
- `npm` / `pnpm` operations
- File operations

---

### When MCP is Better (Evidence)

**From 2025 research:**
> "MCPs are the only option for LLM clients without a built-in shell tool. Stateful tools are easier to implement as MCPs than CLIs since MCP servers are stateful by default."

**Scenarios:**
1. **No suitable CLI exists**
2. **Stateful operations** - Need context across steps
3. **Knowledge graphs** - Semantic relationships, graph queries
4. **Complex decision trees** - Multi-step with conditional logic
5. **Safety constraints** - Need controlled, validated operations

**Examples:**
- Knowledge graph operations (memory)
- Complex multi-step workflows with branching logic
- Operations requiring semantic understanding
- When no CLI tool exists for the operation

---

## Common Tools: CLI vs MCP

### ✅ Use CLI

| Tool | CLI Command | Why CLI is Better |
|------|-------------|-------------------|
| **Git** | `git` | Universal, standard, transparent |
| **GitHub** | `gh` | Official, mature, scriptable |
| **Firebase** | `firebase` | Official Google CLI, full-featured |
| **Supabase** | `supabase` | Official CLI, all operations |
| **NPM** | `npm` / `pnpm` | Standard, fast, explicit |
| **TypeScript** | `tsc`, `tsx` | Direct compilation, execution |
| **Testing** | `vitest`, `jest` | Direct test execution |
| **Linting** | `eslint`, `prettier` | Standard tooling |
| **Docker** | `docker` | Industry standard |
| **Kubernetes** | `kubectl` | Standard tooling |

---

### ✅ Use MCP

| Operation | MCP Server | Why MCP is Better |
|-----------|------------|-------------------|
| **Knowledge graph** | `@modelcontextprotocol/server-memory` | No CLI for graph operations |
| **Multi-step PR workflows** | `github-mcp-server` | Stateful context, conditional logic |
| **Complex database operations** | Database MCP | Contextual queries, safety |

**Note:** Even these should be validated - can CLI scripts work?

---

## Evaluation Process

### Step 1: Identify the Need

**Question:** What operation do we need to perform?

**Example:** "Need to create GitHub PRs after completing tasks"

---

### Step 2: Check for CLI

**Question:** Does a CLI tool exist for this operation?

**How to check:**
```bash
# Search for official CLI
which gh           # GitHub CLI
which firebase     # Firebase CLI
which supabase     # Supabase CLI

# Check command capabilities
gh --help
firebase --help
```

**If CLI exists:** ✅ Proceed to Step 3
**If no CLI:** ⚠️ Consider MCP (Step 5)

---

### Step 3: Assess Complexity

**Question:** Is the operation simple or complex?

**Simple operation** (single command):
```bash
# Examples:
gh pr create --title "..." --body "..."
firebase deploy --only functions
git push origin main
```
**Decision:** ✅ **Use CLI** - No need for MCP

**Complex operation** (multiple steps, conditional):
```bash
# Example:
# 1. List all PRs
# 2. For each PR, check CI status
# 3. If CI passed, read comments
# 4. If comments are resolved, mark ready to merge
# 5. Summarize results
```
**Decision:** ⚠️ Proceed to Step 4

---

### Step 4: Can It Be Scripted?

**Question:** Can we script the complex operation reliably?

**Test:**
```bash
#!/bin/bash
# Try to script the workflow

# Example: PR workflow
PRS=$(gh pr list --json number,title)
for PR in $PRS; do
  STATUS=$(gh pr checks $PR)
  if [ "$STATUS" = "pass" ]; then
    COMMENTS=$(gh api /repos/$REPO/pulls/$PR/comments)
    # Process comments...
  fi
done
```

**If scriptable:** ✅ **Use CLI script** in skill
**If too complex/fragile:** ⚠️ Consider MCP (Step 5)

---

### Step 5: Evaluate MCP

**Question:** Would MCP add value over CLI?

**Evaluate:**
1. **Context benefits** - Does stateful context help?
2. **Natural language** - Is conversational interface better?
3. **Complexity reduction** - Does MCP simplify the workflow?
4. **Overhead** - Is MCP overhead worth the benefits?

**If yes to multiple:** ⚠️ **Try MCP** (but validate!)
**If no:** ✅ **Stick with CLI**

---

### Step 6: Validate (Principle 3)

**Question:** Does MCP actually improve things?

**Experiment design:**
```
Control: CLI-based approach (10 tasks)
Treatment: MCP-based approach (10 tasks)

Metrics:
- Token usage
- Time to completion
- Quality of results
- Error rate
- User satisfaction

Decision:
- If MCP > CLI by >20%: Adopt MCP
- If CLI ≥ MCP: Keep CLI
```

**Make evidence-based decision!**

---

## Examples Applied

### Example 1: GitHub PR Creation

**Need:** Create PRs after tasks complete

**CLI exists?** ✅ Yes - `gh pr create`

**Simple or complex?** Simple - Single command

**Decision:** ✅ **Use CLI**

**Implementation:**
```yaml
# github-pr.skill
scripts:
  create-pr: gh pr create --title "$TITLE" --body "$BODY"
```

---

### Example 2: Knowledge Graph Operations

**Need:** Store and query learned patterns, relationships

**CLI exists?** ❌ No suitable graph database CLI

**Complex?** ✅ Graph queries, relationships, semantic search

**Can it be scripted?** ❌ Would need to build entire graph system

**Decision:** ✅ **Use MCP** (official memory server)

**Rationale:** No CLI alternative, complex graph operations, MCP designed for this

---

### Example 3: Firebase Deployment

**Need:** Deploy Firebase functions, rules, hosting

**CLI exists?** ✅ Yes - `firebase deploy`

**Simple or complex?** Simple - Single command

**Decision:** ✅ **Use CLI**

**Implementation:**
```yaml
# firebase-deploy.skill
scripts:
  deploy-all: firebase deploy
  deploy-functions: firebase deploy --only functions
  deploy-rules: firebase deploy --only firestore:rules
```

---

### Example 4: Complex PR Analysis

**Need:** "Analyze all open PRs, check CI, read comments, identify ready-to-merge, summarize"

**CLI exists?** ✅ Yes - `gh` CLI

**Simple or complex?** ✅ Complex - Multi-step, conditional

**Can it be scripted?** ⚠️ Yes, but complex script

**Try scripting first:**
```bash
#!/bin/bash
# pr-analysis.sh

gh pr list --json number,title,statusCheckRollup | \
jq -r '.[] | select(.statusCheckRollup.state == "SUCCESS") | .number' | \
while read PR; do
  COMMENTS=$(gh api /repos/$REPO/pulls/$PR/comments | jq 'length')
  if [ "$COMMENTS" -eq 0 ]; then
    echo "PR $PR is ready to merge"
  fi
done
```

**Decision:** ✅ **Try CLI script first**

**If script becomes too complex/fragile:** Re-evaluate MCP

---

## Red Flags: When MCP is Wrong

### 🚩 Red Flag 1: Reinventing Existing CLI

**Example:** Building Firebase MCP when `firebase` CLI exists

**Why wrong:**
- ❌ Duplicates functionality
- ❌ Maintenance burden
- ❌ Likely worse than official CLI
- ❌ Context pollution

**Decision:** ✅ Use official CLI

---

### 🚩 Red Flag 2: "Easier" Than Scripting

**Example:** "MCP is easier than writing a bash script"

**Why wrong:**
- ❌ Short-term thinking
- ❌ MCP has its own complexity
- ❌ Scripts are explicit, debuggable
- ❌ MCP context pollution

**Better:** Learn to script properly, use skills system

---

### 🚩 Red Flag 3: Context Pollution

**Example:** MCP server offers 50+ tools, most irrelevant

**From research:**
> "Many MCPs flood the context window with unnecessary output or offer dozens of tools, degrading your agent's performance by poisoning the context."

**Why wrong:**
- ❌ Wastes tokens on tool descriptions
- ❌ Confuses AI with irrelevant options
- ❌ Degrades performance

**Decision:** ✅ Use focused CLI scripts instead

---

### 🚩 Red Flag 4: Unmaintained Community Server

**Example:** Community MCP server, last commit 1 year ago

**Why wrong:**
- ❌ Security risks
- ❌ No bug fixes
- ❌ Compatibility issues
- ❌ Unreliable

**Decision:** ✅ Use official CLI or build our own

---

## Green Flags: When MCP is Right

### ✅ Green Flag 1: No CLI Alternative

**Example:** Knowledge graph operations

**Why right:**
- ✅ No suitable CLI for graph queries
- ✅ Building own would recreate MCP
- ✅ Official MCP server exists

**Decision:** ✅ Use MCP

---

### ✅ Green Flag 2: Official Support

**Example:** GitHub MCP server (by GitHub), Anthropic memory server

**Why right:**
- ✅ Actively maintained
- ✅ Trusted source
- ✅ Well-tested
- ✅ Proper documentation

**Decision:** ⚠️ Still prefer CLI first, but MCP is viable option

---

### ✅ Green Flag 3: Stateful Operations

**Example:** Multi-turn conversation with branching logic

**Why right:**
- ✅ MCP maintains state across calls
- ✅ CLI scripts become very complex
- ✅ Natural language interface adds value

**Decision:** ⚠️ Try scripting first, use MCP if proven valuable

---

## Documentation Template

When making CLI vs MCP decision, document it:

```markdown
## Decision: [Tool/Operation Name]

**Need:** [What operation we need to perform]

**CLI Exists:** [Yes/No] - [CLI tool name if yes]

**Complexity:** [Simple/Complex] - [Description]

**Decision:** [CLI / MCP]

**Rationale:**
- [Key reason 1]
- [Key reason 2]
- [Key reason 3]

**Implementation:**
[Code example or approach]

**Re-evaluate:** [When/if to reconsider]
```

**Store in:** `.ai-knowledge/research/[tool-name]/cli-vs-mcp.md`

---

## Final Checklist

Before adding any MCP server, verify:

- [ ] **CLI doesn't exist or is insufficient** - Tried CLI first?
- [ ] **Operation is complex** - Can't be scripted reliably?
- [ ] **Stateful context helps** - Multi-step with dependencies?
- [ ] **Official or well-maintained** - Trusted source?
- [ ] **Won't pollute context** - Focused, relevant tools only?
- [ ] **Aligns with ADR-003** - Actually necessary, not convenience?
- [ ] **Will validate** - Plan to measure effectiveness?

**If all checked:** ✅ OK to consider MCP

**If any unchecked:** ⚠️ Use CLI instead

---

## Summary

**Default:** ✅ **CLI first, always**

**Add MCP only when:**
1. No CLI exists
2. OR stateful operations that can't be scripted
3. OR knowledge graph / semantic operations
4. AND MCP is official or well-maintained
5. AND we'll validate it actually helps

**Never add MCP for:**
- Convenience (lazy to script)
- "Seems easier"
- Existing CLI works fine
- Would pollute context

**Always validate:** Measure token usage, quality, time vs CLI approach

---

**Research Date:** 2025-11-03
**Based on:** 2025 MCP vs CLI research, ADR-003
**Status:** ✅ Reference framework for all future decisions
