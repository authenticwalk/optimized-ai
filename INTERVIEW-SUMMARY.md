# Interview Summary - Project Requirements

**Date**: November 3, 2025  
**Interviewee**: Chris Priebe - Principal Engineer, 30 years experience, ex-Microsoft

## Core Problem Statement

"I want to work as a PM, not touch code. I create task lists, AI executes them, I review PRs and comment. AI learns from everything and gets better over time."

## Key Pain Points with Current AI Coding Tools

1. **AI Gets Confused & Spins**
   - Goes down wrong paths repeatedly
   - Doesn't detect it's stuck
   - Wastes time on approaches that won't work
   - Forgets what I taught it minutes ago

2. **No Learning Mechanism**
   - I correct the same mistakes repeatedly
   - Doesn't remember my preferences
   - Can't learn from past successes/failures
   - Each conversation starts from zero

3. **Workspace Pollution**
   - Artifacts dumped in home directory
   - Abandoned files left everywhere
   - AI starts one approach, pivots, leaves old files
   - Confusing, bloated projects

4. **Inefficient Operations**
   - AI manually renames files instead of using IDE refactoring
   - Doesn't organize imports properly
   - Reinvents what IDE already does
   - Wastes tokens on trivial operations

## Technical Background & Preferences

### Languages (in order of preference):
1. **TypeScript** - Primary language, loves strong types
2. **Golang** - When performance matters
3. **Python** - Data analysis, scripts, one-offs

**BUT**: Hates fighting with node_modules, tsconfig issues, and TypeScript setup complexity

### Frameworks & Infrastructure:
- **Infrastructure**: Firebase, Supabase
- **Web**: Svelte, HTMX, Ionic+Angular
- **NOT**: React (dislikes), Tailwind (actively hates)

### Tailwind Objection:
"Completely blows away the idea that you should separate design from structure. Makes messy code everywhere. It's the antithesis of beautiful code."

## Coding Philosophy

### Testing:
- **No** to 100% test coverage ("misleading, creates tech debt")
- Tests should cover **edge cases**, not what the compiler catches
- **Integration tests** are key
- **Browser testing** is critical - "seeing it actually work"

### Code Quality:
- **Hates** copy-paste and boilerplate
- **Don't** optimize prematurely - refactor when actually reused
- **Less code is better**
- **Clean code** above all

### Development Approach:
- Pragmatic over dogmatic
- Real value over hype
- Working code over perfect code
- Learn from actual use, not assumptions

## Solution Requirements

### 1. Self-Learning Loops
**The Core Innovation**

AI should:
- Check: "Have I done something similar before?"
- Use: Learned patterns and approaches
- Do: Implement using best known method
- Learn: Record what worked/failed for next time

**What to Learn:**
- Coding patterns and preferences
- Successful vs failed approaches
- User corrections and feedback
- Edge cases encountered
- Refactoring patterns

**Storage:**
- Local with codebase (can sync with git)
- Optional global learnings across projects
- Accessible by AI on every task

### 2. Workspace Management
**Keep It Clean**

- Use organized folders like `.plan/` for work-in-progress
- Never dump files in home directory
- Clean up after task completion
- Archive old plans (keep last 5)
- Only commit actual source code

### 3. Spin Detection
**Auto-Correct When Stuck**

Detect spinning via:
- Repetitive file edits (3+ times in 2 min)
- Same error 3+ times
- No progress for 5 minutes
- Token usage spike

When detected:
- Pause automatically
- Check knowledge base for similar failures
- Try completely different approach
- If still stuck after 2 attempts, ask for help

### 4. IDE Integration
**Use IDE, Don't Recreate It**

VSCode/Cursor plugin that exposes:
- Rename symbol (proper refactoring)
- Organize imports
- Format document
- Show references
- Find definition
- Run tests
- Get linter errors
- Apply quick fixes

AI calls these via MCP instead of manual edits.

### 5. Self-Evaluation
**Only Commit Working Code**

Before creating PR, AI must:
1. Run all tests (must pass)
2. Check linter (no errors)
3. Validate against requirements
4. Review against learned patterns
5. Self-critique: "What could go wrong?"
6. If issues found: Fix and repeat
7. Only create PR when everything passes

### 6. GitHub PR Workflow
**PM Review Cycle**

AI creates PRs:
- Feature branch: `ai/task-description-YYYYMMDD`
- Descriptive commit messages
- PR includes: task, changes, tests, self-eval
- Assigns to user for review

User reviews on GitHub:
- Comments on specific lines
- AI reads comments
- AI makes fixes
- AI updates PR
- Repeat until user merges

**Never**: Auto-merge to main

### 7. Infrastructure Helpers
**Firebase/Supabase MCPs**

For debugging and validation:
- Read data from database
- Check auth state
- Verify security rules/RLS policies
- Test cloud functions/RPCs
- Validate queries actually work

## Project Setup Flow

### Initial Setup:
```bash
npx @optimized-ai/init
```

Wizard asks:
1. Primary language? (TS/Go/Python/Multi)
2. Infrastructure? (Firebase/Supabase/Other)
3. Web framework? (None/Svelte/HTMX/Ionic)
4. Test framework? (Jest/Vitest/Go test/pytest)
5. Import global learnings? (Y/N)

Creates:
- `.cursorrules` - Project-specific Claude rules
- `claude.md` - Configuration and patterns
- `.plan/` folder (git-ignored) for work
- `.ai-knowledge/` folder (git-tracked) for learnings
- `optimized-ai.config.json` - Settings
- GitHub workflow for PR validation

### Doesn't Create:
- Any artifacts in home directory
- Unnecessary boilerplate
- Complex tsconfig (keeps it simple)

## Success Metrics

**You know it works when:**
1. Chris spends 90% of time as PM, 10% reviewing PRs
2. AI rarely spins (< 5% of tasks)
3. PRs need < 2 rounds of feedback
4. System improves over time
5. No abandoned files
6. All committed code works

**Ultimate Goal:**
"I could switch to a role of project manager and never have to touch code again. I create lists of work, it does it. I occasionally review on GitHub PR page and add comments for it to resolve."

## Key Quotes

> "100% test coverage is misleading, often creating technical debt to maintain tests that test very little."

> "I hate it when AI dumps files in my home directory that are artifacts."

> "I also hate how AI makes files then goes another direction then leaves files hanging around."

> "What I really want is self learning loops so when AI does something it checks if it has learnt from the past how to do that better."

> "I also don't understand why AI will do things that the IDE should be doing, things like renaming files, cleaning up imports."

> "There is also the reality that given a few weeks Claude builds it into their base so much of it is obsolete." (Re: hype about agents/skills)

> "I hate Tailwind because it completely blows the whole idea away of you should separate your design from your structure."

> "Tests should never cover things that the compiler would cover like sending the wrong type."

> "Less code is better."

## Scope & Distribution

**Primary User**: Chris himself
**Distribution**: Open source, others can use if helpful, but not the driving factor
**Platform**: Cursor/Claude Code specifically
**Delivery**: NPM package (`npx @optimized-ai/init`)

**Similar To**: How Claude Code's `npx init` sets up a project - but for AI optimization

## Next Steps

1. ✅ **Interview Complete** - Requirements captured
2. ✅ **Spec Written** - Full system specification in `SPEC.md`
3. ✅ **Roadmap Created** - 12-week implementation plan in `ROADMAP.md`
4. → **Phase 1 Implementation** - Begin foundation work

## Open Questions (Resolved)

1. ~~Where does learning database live?~~ → Local with codebase, optional global sync
2. ~~How aggressive should cleanup be?~~ → Archive last 5 tasks, clean rest
3. ~~Should AI auto-use IDE features or ask?~~ → Auto-use via MCP
4. ~~Git-ignore .plan folder?~~ → Yes, ignored
5. ~~Git-track .ai-knowledge?~~ → Yes, tracked
6. ~~What about Reuv's claude-flow?~~ → Reference for .plan structure patterns
7. ~~Auto-detect spinning triggers?~~ → Yes, multiple triggers (repetition, time, tokens, errors)
8. ~~Should AI merge to main?~~ → Never, user decision only
9. ~~Architecture decisions?~~ → AI is self-maintaining, suggests improvements on branches, never merges
10. ~~Commit workflow?~~ → Only commit working code that passes all checks

## Notes

- Chris is experienced enough to know what he wants
- Not interested in hype or trends that become obsolete
- Values pragmatism and real productivity gains
- Willing to share but building primarily for himself
- Understands the difference between valuable abstractions and premature optimization
- Strong opinions based on decades of experience
- Wants to focus on PM/architecture, not implementation details

