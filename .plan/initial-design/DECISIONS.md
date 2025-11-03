# Architectural Decision Records

This document captures key architectural and design decisions made during the project definition phase.

---

## ADR-001: Local Knowledge Base with Git Tracking

**Status**: Accepted  
**Date**: 2025-11-03

### Context
Need to store learned patterns, preferences, and failures. Options:
1. Cloud storage (Firebase/Supabase)
2. Local database (SQLite) in home directory
3. Local JSON files in project, tracked by git
4. Global storage in home directory

### Decision
Store knowledge in `.ai-knowledge/` folder within each project, tracked by git.

### Rationale
- Version controlled with code (patterns evolve with project)
- Shareable with team (everyone benefits from learnings)
- No external dependencies
- Human-readable JSON for debugging
- Can sync with global learnings via script
- Privacy-friendly (stays local unless pushed)

### Consequences
- Project-specific learnings (good for context)
- Need sync mechanism for cross-project patterns
- Some duplication across projects (acceptable)
- Knowledge base grows with project (manageable)

---

## ADR-002: Git-Track `.ai-knowledge/` Folder with Plans and Research

**Status**: Accepted  
**Date**: 2025-11-03  
**Updated**: 2025-11-03

### Context
AI needs structured storage for plans, research, and learnings that can be referenced and learned from over time.

### Decision
Create `.ai-knowledge/` folder with structured subfolders, **tracked by git**.

**Structure**:
```
.ai-knowledge/
├── knowledge.db (SQLite - searchable knowledge base)
├── plans/
│   ├── {plan-name}/
│   │   ├── plan.md
│   │   ├── progress.md
│   │   ├── learnings.md
│   │   └── subtasks/
│   │       └── {subtask-name}/
│   └── ...
└── research/
    ├── {topic}/
    │   ├── overview.md
    │   └── {optional-subtopic}/
    │       └── findings.md
    └── ...
```

### Rationale
- **Plans are learning artifacts** - Should be preserved and referenced
- **Research is reusable** - "Best practices for X" benefits future tasks
- **SQLite for search** - Fast, structured queries across knowledge
- **Git-tracked** - Version controlled, shareable with team
- **Structured folders** - Easy to navigate and organize
- **Supports learning loops** - AI can learn from past plans

### Consequences
- Plans become part of project history (GOOD - valuable context)
- Knowledge base grows with project (manageable, valuable)
- Team can see AI's reasoning process (transparency)
- Can reference past solutions easily
- SQLite needs backup/migration strategy
- Folder structure needs discipline to maintain

---

## ADR-003: Direct Scripts Over MCP Middleware (Revised)

**Status**: Accepted  
**Date**: 2025-11-03  
**Updated**: 2025-11-03

### Context
Need way for AI to interact with external tools (Firebase, Supabase, IDE, etc.). Initially considered MCP servers for everything.

### Decision
**Prefer direct CLI scripts over MCP middleware.** Only use MCP when actually necessary.

**Approach**:
1. Try direct CLI first (firebase, supabase, tsx, etc.)
2. Create simple shell scripts for complex operations
3. Skills call scripts directly
4. Only add MCP if proven necessary (complex state, real-time data, etc.)

### Rationale
- **Simpler**: CLI tools already exist (firebase, supabase, tsx)
- **Less middleware**: No unnecessary abstraction layers
- **Easier to test**: Scripts can be tested independently
- **More maintainable**: Standard tools vs custom MCP servers
- **Faster development**: Don't build what already exists

### Examples

**Firebase**:
```yaml
# Instead of: MCP server wrapper
# Just use: Firebase CLI directly
skill: firebase-debug
scripts:
  check-connection: firebase projects:list
  query-users: firebase firestore:get users --limit 10
```

**TypeScript Scripts**:
```yaml
skill: data-analysis
scripts:
  analyze: tsx scripts/analyze-data.ts
```

### When MCP Might Be Needed
- Complex state management across calls
- Real-time data streaming
- Cross-tool coordination
- IDE integration (VSCode extension)

### Consequences
- **Positive**: Simpler architecture, less code to maintain
- **Positive**: Use battle-tested CLI tools
- **Positive**: Faster to implement
- **Trade-off**: May need MCP later for specific cases
- **Risk**: If CLI changes, scripts need updates (but same for MCP)

---

## ADR-004: No React, No Tailwind

**Status**: Accepted  
**Date**: 2025-11-03

### Context
System needs to support web frameworks for generated projects.

### Decision
Support Svelte, HTMX, Ionic+Angular. Actively discourage React and Tailwind.

### Rationale
- **React**: User preference based on experience
- **Tailwind**: Violates separation of concerns principle
- **Svelte**: Clean, simple, fast
- **HTMX**: Minimal JavaScript, server-driven
- **Ionic+Angular**: Cross-platform (Android/iOS/Web)

### Consequences
- System will have opinionated defaults
- Users who want React/Tailwind need to override
- Learnings focused on preferred frameworks
- Patterns assume clean separation of concerns

---

## ADR-005: TypeScript Primary, Avoid Config Complexity

**Status**: Accepted  
**Date**: 2025-11-03

### Context
Need to choose implementation language and handle project configurations.

### Decision
Build system in TypeScript. Generate minimal `tsconfig.json`. Avoid complex TypeScript configurations.

### Rationale
- TypeScript provides type safety user values
- Strong typing prevents whole classes of bugs
- User experienced with TypeScript
- BUT: User frustrated by tsconfig complexity
- Keep configs simple and standard

### Consequences
- Use common tsconfig presets
- Don't generate complex path mappings
- Avoid experimental TypeScript features
- Focus on clean, simple TypeScript
- May not support every TS feature

---

## ADR-006: Integration Tests Over Unit Tests

**Status**: Accepted  
**Date**: 2025-11-03

### Context
Testing strategy for the system and guidance for generated code.

### Decision
Prioritize integration tests and edge cases. Skip tests for what compiler already validates.

### Rationale
- User philosophy: 100% coverage is misleading
- TypeScript compiler catches type errors
- Integration tests prove it actually works
- Edge cases are where bugs hide
- Browser testing critical for web apps

### Consequences
- Won't have 100% unit test coverage (by design)
- Focus testing effort on complex logic and edge cases
- Always test in real environment (browser, etc.)
- Compiler errors caught at build time

---

## ADR-007: Self-Evaluation Before Commit

**Status**: Accepted  
**Date**: 2025-11-03

### Context
AI might commit broken code without validation.

### Decision
Implement mandatory self-evaluation loop before any commit/PR creation.

### Rationale
- Only working code should reach git
- AI should validate its own work
- Tests must pass
- Linter must be clean
- Self-critique catches logical issues

### Consequences
- Slower commit cycle (good - quality over speed)
- AI attempts multiple fixes if needed
- Max 3 attempts before escalating to user
- Every commit is validated

---

## ADR-008: Never Auto-Merge to Main

**Status**: Accepted  
**Date**: 2025-11-03

### Context
Should AI be able to merge its own PRs?

### Decision
AI creates PRs but never merges them. User always merges to main.

### Rationale
- User wants final control
- PM reviews and approves
- Prevents runaway AI changes
- User can reject entire approach
- Forces human review checkpoint

### Consequences
- AI can't fully automate deployment
- User must check GitHub regularly
- Slower overall workflow (acceptable)
- Higher quality, more control

---

## ADR-009: Spin Detection with Auto-Correction

**Status**: Accepted  
**Date**: 2025-11-03

### Context
AI sometimes gets stuck repeating the same failed approach.

### Decision
Implement automatic spin detection with multiple triggers and auto-correction.

**Triggers:**
- Same file edited 3+ times in 2 minutes
- Same error 3+ times
- No progress for 5 minutes
- Token spike >50k in one response

**Response:**
- Pause automatically
- Check knowledge base for similar failures
- Try different approach
- If still stuck after 2 attempts, escalate

### Rationale
- Current AI assistants don't detect spinning
- Wastes time and tokens
- Frustrating user experience
- Knowledge base has solutions to try

### Consequences
- Need monitoring system
- Need pattern matching for "similar failures"
- Adds complexity but high value
- Reduces frustration significantly

---

## ADR-010: VSCode Extension for IDE Operations

**Status**: Accepted  
**Date**: 2025-11-03

### Context
AI often manually edits code for things IDE can do better (rename symbols, organize imports).

### Decision
Build VSCode/Cursor extension exposing IDE operations via MCP.

**Operations:**
- Rename symbol
- Organize imports
- Format document
- Show references
- Find definition
- Run tests
- Get linter errors
- Apply quick fixes

### Rationale
- IDE refactoring is safer (updates all references)
- IDE knows project structure
- Faster than manual edits
- Proper formatting
- Leverages existing tooling

### Consequences
- Need to build VSCode extension
- Need to learn VSCode API
- Need MCP server for extension
- But: Better UX, fewer errors, proper refactoring

---

## ADR-011: Firebase and Supabase via CLI (Revised)

**Status**: Accepted  
**Date**: 2025-11-03  
**Updated**: 2025-11-03

### Context
User primarily uses Firebase and Supabase for infrastructure. Need AI to debug and validate.

### Decision
Use Firebase CLI and Supabase CLI directly via shell scripts. No MCP middleware initially.

**Approach**:
```yaml
# firebase-debug.skill
scripts:
  list-projects: firebase projects:list
  query-firestore: firebase firestore:get $COLLECTION --limit 10
  check-auth: firebase auth:export auth-dump.json
  validate-rules: firebase deploy --only firestore:rules --dry-run
  test-function: firebase functions:shell

# supabase-debug.skill  
scripts:
  db-dump: supabase db dump
  list-functions: supabase functions list
  check-migrations: supabase db diff
  test-connection: supabase status
```

### Rationale
- **Firebase CLI** and **Supabase CLI** already have all needed features
- No need to wrap them in MCP servers
- Simpler, more maintainable
- AI can call CLI directly
- Skills coordinate which commands to use

### When MCP Might Be Needed
- Real-time listeners
- Complex multi-step workflows requiring state
- If CLI proves insufficient (validate first)

### Consequences
- **Positive**: Much simpler implementation
- **Positive**: Use official, maintained CLIs
- **Positive**: No auth complexity (CLIs handle it)
- **Trade-off**: If CLI insufficient, add MCP later

---

## ADR-012: Opinionated with Escape Hatches

**Status**: Accepted  
**Date**: 2025-11-03

### Context
How opinionated should the system be?

### Decision
Strongly opinionated defaults, but allow overrides via config.

**Opinions:**
- No React/Tailwind
- Separation of concerns
- Minimal boilerplate
- Refactor on reuse
- Clean workspace
- Integration tests

**Overrides:**
- User can edit `.cursorrules`
- User can edit `optimized-ai.config.json`
- User can disable features

### Rationale
- Strong opinions create consistency
- User has clear preferences
- But others might differ
- Config allows customization
- Sane defaults for new users

### Consequences
- System will feel "opinionated"
- Users who disagree can override
- Documentation must explain opinions
- Some users might not like it (acceptable)

---

## ADR-013: Workspace Cleanliness as First-Class Feature

**Status**: Accepted  
**Date**: 2025-11-03  
**Updated**: 2025-11-03

### Context
AI often creates artifacts and leaves them around, polluting projects.

### Decision
Make workspace cleanliness a core system feature, not an afterthought.

**Rules:**
- AI workspace in `.ai-knowledge/` (tracked, structured)
- Plans organized in `.ai-knowledge/plans/{plan-name}/`
- Research organized in `.ai-knowledge/research/{topic}/`
- No artifacts in home directory
- No abandoned files in project root
- Clear folder structure maintained

### Rationale
- User specifically hates messy workspaces
- Professional projects need clean structure
- Abandoned files confusing
- Clear organization aids understanding
- Plans and research are valuable learning artifacts

### Consequences
- Need disciplined folder organization
- Need to track what AI creates
- All AI work products tracked in git (transparency)
- Knowledge base grows with project (valuable)
- But: Much better UX and learning capability

---

## ADR-014: PM-Mode as Primary Workflow

**Status**: Accepted  
**Date**: 2025-11-03

### Context
What is the target user experience?

### Decision
Design entire system around PM-mode workflow: user directs, AI implements, user reviews.

**Workflow:**
1. User writes task description
2. AI plans and implements
3. AI self-evaluates
4. AI creates PR
5. User reviews PR on GitHub
6. User comments for changes
7. AI reads comments and fixes
8. Repeat until user merges

### Rationale
- User's stated goal: never touch code
- Work as project manager
- AI handles implementation details
- User provides direction and approval

### Consequences
- System optimized for this workflow
- Other workflows may not fit well
- Requires robust PR integration
- Requires good self-evaluation
- Requires comment parsing
- But: Exactly what user wants

---

## ADR-015: Progressive Enhancement Over Big Bang

**Status**: Accepted  
**Date**: 2025-11-03

### Context
How to approach implementation?

### Decision
Build in phases with each phase independently useful.

**Phases:**
1. Foundation (init, knowledge, config)
2. Workspace management
3. Spin detection
4. IDE integration
5. Self-evaluation
6. PR workflow
7. Learning system
8. Infrastructure MCPs
9. Agent coordination
10. Polish

### Rationale
- 12-week full implementation is long
- Each phase adds value
- User can use partial system
- Easier to test incrementally
- Can adjust based on learnings

### Consequences
- Longer to full feature set
- But each phase ships value
- Can get feedback between phases
- Less risk of building wrong thing

---

## Future Decisions to Make

### Open Questions:
1. **Pattern Recognition**: Use ML or heuristics for learning?
2. **Global Sync**: How to merge global learnings? Manual or automatic?
3. **Multi-User**: How to handle team learnings?
4. **Cloud Backup**: Optional cloud storage for learnings?
5. **Analytics**: Track metrics? Privacy implications?
6. **Versioning**: How to version knowledge base format?
7. **Migration**: How to upgrade old knowledge bases?

These will be decided as we implement relevant phases.

---

## Change Log

- **2025-11-03**: Initial decision records from requirements interview

