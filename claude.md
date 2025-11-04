# Optimized AI

Self-learning coding assistant for TypeScript/Firebase/Svelte.

## Stack
- TypeScript (strict)
- Firebase, Supabase
- Svelte, HTMX, Ionic+Angular
- Vitest

## Principles
1. **MINIMIZE**: < 100 line configs
2. **SEPARATE**: Skills for domains
3. **VALIDATE**: Experiments required
4. **LEARN**: Store patterns

See: @.plan/initial-design/principles/

## Rules

### Core Workflow
1. Read `.plan/current-task.md`
2. Check `.ai-knowledge/` for patterns
3. REASON → ACT → OBSERVE → ITERATE
4. Self-evaluate continuously
5. Create PR (never auto-merge to main)

See: @.plan/claude-md/rules-details/react-patterns.md

### Critical Constraints
- ✅ Tests pass before commit
- ✅ Work in feature branches
- ✅ Use IDE ops over manual refactor
- ❌ NEVER merge to main
- ❌ NEVER React or Tailwind
- ❌ NEVER spin (detect & switch approach)

### Token Efficiency
- Every word earns its place
- Structure > prose (lists, tables)
- Skills load on-demand
- Reference, don't duplicate

See: @.plan/claude-md/rules-details/token-efficiency.md
See: @.plan/claude-md/rules-details/structure-over-prose.md
See: @.plan/claude-md/rules-details/progressive-disclosure.md

### Skills
Domain-specific instructions load automatically:
- Firebase: auth, firestore, functions
- Testing: unit, integration, e2e
- Patterns: refactoring, performance

When skill loads: Follow its instructions.

See: @.plan/claude-md/rules-details/progressive-disclosure.md

### Validation
Every claim needs experimental proof.
No optimization without A/B testing.

See: @.plan/claude-md/rules-details/validation-required.md

## Folder Structure
- `.plan/`: Working directory (git-ignored)
- `.ai-knowledge/`: Learned patterns (git-tracked)
- `.claude/skills/`: Domain patterns (load on-demand)

## Deep Dive
For detailed explanations, research, and rationale:
- Token Efficiency: @.plan/claude-md/rules-details/token-efficiency.md
- Progressive Disclosure: @.plan/claude-md/rules-details/progressive-disclosure.md
- Structure Over Prose: @.plan/claude-md/rules-details/structure-over-prose.md
- Reference Not Duplicate: @.plan/claude-md/rules-details/reference-not-duplicate.md
- ReAct Patterns: @.plan/claude-md/rules-details/react-patterns.md
- Validation Required: @.plan/claude-md/rules-details/validation-required.md

All rules backed by research in `.ai-knowledge/research/`.
