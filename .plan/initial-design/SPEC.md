# Optimized AI - System Specification

## Vision
A self-learning AI coding assistant system that enables you to work as a PM while AI handles implementation. You create task lists, AI executes on branches, self-evaluates, and creates PRs. You review and comment on PRs, never touching code directly.

## Core Principles
- **Self-Learning**: System remembers patterns, learns from successes/failures
- **Clean Workspace**: Organized folders, no artifacts in home directory
- **Auto-Detection**: Catches when AI is spinning or confused
- **IDE-First**: Use built-in IDE features instead of recreating them
- **Working Code Only**: Only commits code that passes tests
- **No Main Merges**: AI never merges to main - that's your decision

## Architecture

### 1. Initialization System
**Command**: `npx @optimized-ai/init` or `optimized-ai init`

**What it does:**
- Quick wizard to set up project
- Analyzes existing codebase (if any)
- Creates necessary configuration files
- Sets up folder structure
- Configures hooks and agents

**Files Created:**
- `.cursorrules` - Project-specific Claude rules
- `claude.md` - Main configuration and patterns
- `.plan/` - Git-ignored folder for current work
- `.ai-knowledge/` - Local learning database (git-tracked)
- `.github/workflows/ai-check.yml` - Pre-PR validation
- `optimized-ai.config.json` - Project settings

**Wizard Questions:**
1. Primary language? (TypeScript/Golang/Python/Multi)
2. Infrastructure? (Firebase/Supabase/Other)
3. Web framework? (None/Svelte/HTMX/Ionic+Angular)
4. Test framework? (Jest/Vitest/Go test/pytest)
5. Import global learnings? (Y/N)

### 2. Learning Database

**Location**: `.ai-knowledge/`
- `patterns.json` - Successful code patterns
- `failures.json` - What didn't work and why
- `preferences.json` - Your coding preferences
- `corrections.json` - When you corrected AI
- `metrics.json` - Performance data

**Global Sync**:
- `~/.optimized-ai/global-knowledge/` - Cross-project learnings
- Command: `optimized-ai sync-global` - Merges global into project

**What it captures:**
- File/folder organization patterns you prefer
- How you structure TypeScript projects
- Firebase/Supabase patterns that worked
- Failed approaches and why they failed
- Your corrections and feedback
- Refactoring patterns you use

### 3. Workspace Management

**`.plan/` folder** (git-ignored):
- `current-task.md` - Current work description
- `approach.md` - Planned approach
- `progress.md` - What's been done
- `blockers.md` - Current issues
- `tests.md` - Test plan and results

**Cleanup Rules:**
- After successful PR creation: Archive to `.plan/archive/YYYY-MM-DD-task-name/`
- Keep last 5 tasks in archive
- Compress older archives
- Never leave abandoned files in project root

### 4. Spin Detection System

**Triggers** (any of these = spinning):
- Same file edited 3+ times in 2 minutes
- Same error encountered 3+ times
- No progress for 5 minutes with activity
- Token usage spike (>50k in one response)
- Repetitive tool calls (same call 3+ times)

**When Detected:**
1. Auto-pause execution
2. Check `.ai-knowledge/failures.json` for similar situations
3. Generate alternative approach
4. Log spin incident
5. Resume with new approach

**System Prompt Addition:**
```
If you detect you're repeating yourself or stuck:
1. STOP immediately
2. Check your knowledge base for similar failures
3. Try a completely different approach
4. If still stuck after 2 attempts, document the blocker and ask for guidance
```

### 5. IDE Integration Plugin

**VSCode/Cursor Extension**: `optimized-ai-ide`

**Exposed Operations (via MCP):**
- `ide.renameSymbol(oldName, newName)` - Use IDE refactoring
- `ide.organizeImports(file)` - Clean up imports
- `ide.formatDocument(file)` - Format with project settings
- `ide.showReferences(symbol)` - Find all references
- `ide.findDefinition(symbol)` - Go to definition
- `ide.runTests(pattern?)` - Execute test suite
- `ide.getLinterErrors()` - Get current errors
- `ide.applyQuickFix(error)` - Apply suggested fixes

**Why**: Let IDE do what it's good at, AI focuses on logic

### 6. Self-Evaluation Loop

**Before Creating PR:**
1. Run all tests (`ide.runTests()`)
2. Check linter (`ide.getLinterErrors()`)
3. Validate against requirements in `.plan/current-task.md`
4. Review changes against learned patterns
5. Self-critique: "What could go wrong?"
6. If issues found: Fix and repeat
7. Only create PR when all checks pass

**Evaluation Criteria:**
- All tests pass
- No linter errors
- Follows project patterns (from `.ai-knowledge/patterns.json`)
- Doesn't violate preferences (from `.ai-knowledge/preferences.json`)
- Edge cases considered
- No duplicate/boilerplate code

### 7. GitHub PR Workflow

**When Task Complete:**
1. Create feature branch: `ai/task-description-YYYYMMDD`
2. Commit changes with descriptive message
3. Run final self-evaluation
4. Create PR with:
   - Task description
   - Changes made
   - Test results
   - Self-evaluation notes
5. Assign to you for review

**Reading PR Comments:**
- MCP server monitors PR comments
- When you comment, AI:
  - Reads comment
  - Updates `.plan/current-task.md` with feedback
  - Makes changes
  - Re-evaluates
  - Pushes updates
  - Replies to comment confirming fix

**Never**: Auto-merge to main

### 8. MCP Servers

**optimized-ai-core** - Main learning system
- `getPattern(type)` - Retrieve learned patterns
- `savePattern(type, data)` - Save new pattern
- `checkFailure(scenario)` - Check if similar failure exists
- `recordSuccess(task, approach)` - Log success
- `recordFailure(task, approach, reason)` - Log failure

**optimized-ai-ide** - IDE operations
- All IDE operations listed in section 5

**optimized-ai-firebase** - Firebase helpers
- `queryData(collection, query)` - Read Firestore data
- `checkAuth(uid)` - Verify auth state
- `validateRules()` - Check security rules
- `testFunction(name, payload)` - Test cloud function

**optimized-ai-supabase** - Supabase helpers
- `queryTable(table, query)` - Read data
- `checkRLS(table)` - Verify RLS policies
- `testRPC(name, params)` - Test RPC function

### 9. Hooks & Agents

**Hooks:**
- `pre-commit` - Verify no spin incidents unresolved
- `pre-push` - Ensure `.plan/` is clean
- `post-checkout` - Load relevant knowledge for branch

**Agents:**
- `planner` - Breaks down tasks into steps
- `implementer` - Writes code following patterns
- `tester` - Creates and runs tests
- `reviewer` - Self-reviews code
- `cleaner` - Cleans up workspace
- `learner` - Updates knowledge base

**Agent Coordination:**
1. Planner creates plan in `.plan/`
2. Implementer writes code, consulting knowledge
3. Tester validates
4. Reviewer checks quality
5. If issues: Loop back to implementer
6. When passing: Cleaner tidies up
7. Learner updates knowledge
8. Create PR

### 10. Configuration Files

**`.cursorrules`:**
```
# Optimized AI Rules
- Check .ai-knowledge/ before starting any task
- Use IDE operations instead of manual refactoring
- Follow patterns in .ai-knowledge/patterns.json
- Respect preferences in .ai-knowledge/preferences.json
- Work in .plan/ folder, keep main workspace clean
- Auto-detect spinning, switch approaches
- Only commit passing code
- Self-evaluate before creating PRs
- Never merge to main
```

**`claude.md`:**
- Generated by wizard based on project analysis
- Contains project-specific patterns
- Lists tech stack and preferences
- Links to key documentation

**`optimized-ai.config.json`:**
```json
{
  "version": "1.0.0",
  "language": "typescript",
  "infrastructure": ["firebase"],
  "frameworks": ["svelte"],
  "testFramework": "vitest",
  "spinDetection": {
    "enabled": true,
    "repetitionThreshold": 3,
    "timeWindow": 120,
    "tokenThreshold": 50000
  },
  "workspace": {
    "planFolder": ".plan",
    "knowledgeFolder": ".ai-knowledge",
    "archiveCount": 5
  },
  "evaluation": {
    "requireTests": true,
    "requireLinting": true,
    "requireSelfReview": true
  },
  "preferences": {
    "noReact": true,
    "noTailwind": true,
    "separateDesignFromStructure": true,
    "refactorOnReuse": true,
    "minimalBoilerplate": true
  }
}
```

### 11. User Workflow

**Starting a New Task:**
```bash
# You create a task
echo "Add user authentication with Firebase" > .plan/current-task.md

# AI takes over
# - Reads task
# - Checks knowledge base for similar tasks
# - Creates plan in .plan/approach.md
# - Implements following patterns
# - Self-evaluates continuously
# - Creates PR when done
```

**During Implementation:**
- You don't touch code
- AI works autonomously
- Auto-detects and corrects spinning
- Updates `.plan/progress.md` regularly

**Review Phase:**
- You get PR notification
- Review on GitHub
- Add comments on specific lines
- AI reads comments, makes fixes, updates PR
- When satisfied, you merge

**Teaching the System:**
- When you manually fix something, system logs it
- Your corrections become new patterns
- Your preferences are learned and enforced
- Failed approaches are remembered and avoided

## Tech Stack

**Core System:**
- TypeScript (for main tooling)
- Node.js (CLI and MCPs)
- SQLite (optional, for complex queries on knowledge)

**VSCode Extension:**
- TypeScript
- VSCode Extension API

**Supported Project Stacks:**
- **TypeScript** (primary)
- **Golang** (performance-critical)
- **Python** (data/scripts)
- **Web**: Svelte, HTMX, Ionic+Angular
- **Infrastructure**: Firebase, Supabase

## Success Metrics

**You know it's working when:**
1. You spend 90% of time in PM mode, 10% reviewing PRs
2. AI rarely spins (< 5% of tasks)
3. PRs require < 2 rounds of feedback
4. System learns and improves over time
5. No abandoned files in workspace
6. All committed code works

## Anti-Patterns to Avoid

**Don't:**
- Create artifacts in home directory
- Leave abandoned files
- Copy-paste boilerplate
- Test what compiler catches
- Spin without detecting it
- Commit broken code
- Merge to main automatically
- Use AI for IDE operations
- Force React/Tailwind

## Future Enhancements

**Phase 2:**
- Multi-project learning sync
- Team collaboration features
- Custom agent creation
- Advanced pattern recognition
- Performance optimization suggestions
- Automatic dependency updates
- Security vulnerability detection

**Phase 3:**
- Cloud-based learning (optional)
- AI pair programming mode
- Voice command integration
- Architecture decision support
- Code quality trends
- Cost optimization tracking

