# Git Commits Best Practices

**Research Quality**: ⭐⭐⭐⭐⭐ (AgentDB, Roo Code, quality-workflow-meta)
**Status**: ✅ IMPLEMENTATION-READY

## Conventional Commits (Gitflow Syntax)

### Format (ENFORCE THIS)

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, missing semicolons, etc.
- `refactor`: Code restructuring without behavior change
- `test`: Adding or fixing tests
- `chore`: Build process, dependencies, tooling

**Examples**:
```bash
feat(auth): add JWT token refresh mechanism
fix(api): handle null response in user endpoint
docs(readme): update installation instructions
refactor(utils): extract date formatting to helper
test(auth): add integration tests for login flow
chore(deps): update axios to v1.6.0
```

### How Often to Commit

**Atomic Commits Principle** (from Roo Code research):

✅ **DO commit**:
- After each logical unit of work
- When tests pass
- Before switching context/tasks
- After fixing a bug (one commit per fix)
- After completing a feature component

❌ **DON'T commit**:
- Mid-refactoring (broken code)
- With failing tests
- Multiple unrelated changes together
- "WIP" or "temp" commits (squash these)

**Frequency guideline**: 3-10 commits per feature, not 50+

### Grouping Files in Commits

**Principle**: Group by LOGICAL CHANGE, not by file type

✅ **GOOD grouping**:
```bash
# Single commit for authentication feature
git add src/auth/login.ts
git add src/auth/login.test.ts
git add src/routes/auth.ts
git commit -m "feat(auth): add login endpoint with JWT"
```

❌ **BAD grouping**:
```bash
# Commit 1: All TypeScript files
git add src/**/*.ts
git commit -m "update typescript files"

# Commit 2: All tests
git add src/**/*.test.ts
git commit -m "update tests"
```

**File Grouping Rules**:
1. Implementation + tests together
2. Related routes/controllers/models together
3. Config changes separate from code changes
4. Documentation updates separate

### Moving Files (Preserve Git History)

**CRITICAL**: Do `git mv` immediately, commit separately

✅ **CORRECT sequence**:
```bash
# 1. Move file with git (preserves history)
git mv src/old-name.ts src/new-name.ts

# 2. Commit the move IMMEDIATELY (before editing)
git commit -m "refactor: rename old-name to new-name"

# 3. Then make changes to the file
vim src/new-name.ts

# 4. Commit changes separately
git commit -am "refactor(new-name): update logic for new structure"
```

❌ **WRONG sequence**:
```bash
# Move and edit in same commit = LOSES HISTORY
mv src/old-name.ts src/new-name.ts
vim src/new-name.ts
git add src/new-name.ts
git commit -m "renamed and updated file"  # Git sees as DELETE + ADD
```

**Why this matters**: Git tracks files by similarity. Moving + editing = too much change = lost history.

**Pattern from Roo Code**: `safeWriteFile` always moves to temp first:
```typescript
// Write to temp location
fs.writeFileSync(tempPath, content)
// Atomic move (preserves history)
fs.renameSync(tempPath, finalPath)
```

### Learning from Past Commits (AgentDB Integration)

**Store commit patterns**:
```sql
-- After successful commit
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'feat(auth): conventional format with scope',
    'git_commit',
    0.85,
    'success'
);

-- After rejected commit
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'fixed stuff',  -- Too vague
    'git_commit',
    'Code review rejected - unclear description'
);
```

**Query before committing**:
```sql
-- Get high-confidence commit patterns
SELECT pattern, confidence
FROM patterns
WHERE context='git_commit'
AND confidence > 0.7
ORDER BY confidence DESC LIMIT 5;
```

**Expected learning over time**:
- Week 1: Agent learns your project uses conventional commits
- Week 2: Agent learns common scopes (auth, api, ui, etc.)
- Week 3: Agent learns description patterns that get approved
- Month 2: Agent knows which commit formats correlate with fast PR approval

## Commit Message Quality

### Description Guidelines

**Length**: 50-72 characters (enforced by hooks)

**Tense**: Imperative ("add feature" not "added feature")

**Focus**: WHAT and WHY, not HOW (code shows how)

✅ **GOOD descriptions**:
```
feat(auth): add JWT refresh to prevent session timeout
fix(api): handle race condition in concurrent requests
refactor(db): extract connection pool for reuse
```

❌ **BAD descriptions**:
```
updated files
fixed bug
changes
WIP
asdf
```

### Body (When to Use)

**Use body when**:
- Change is complex and needs explanation
- Breaking changes
- Migration steps required
- Context for future maintainers

**Format**:
```
feat(api): add GraphQL endpoint for user queries

Previous REST endpoints were inefficient for complex user data.
New GraphQL endpoint reduces payload size by 60% and allows
clients to request exactly what they need.

Breaking change: /api/users endpoint deprecated, migrate to
/graphql with userQuery. See migration guide in docs/migration.md
```

### Footers (References)

**Common footers**:
```
Fixes #123
Closes #456
Related to #789
Breaking change: API version 2.0 required
```

## Pre-Commit Hooks (Enforcement)

**From quality-workflow-meta research**: Hooks > Prompts

### Hook 1: Conventional Commit Format Check

```bash
#!/bin/bash
# .husky/commit-msg

COMMIT_MSG=$(cat "$1")
PATTERN='^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,72}'

if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
    echo "❌ Commit message doesn't follow conventional commits format"
    echo ""
    echo "Expected: <type>(<scope>): <description>"
    echo "Example: feat(auth): add login endpoint"
    echo ""
    echo "Types: feat, fix, docs, style, refactor, test, chore"
    exit 1
fi

echo "✅ Commit message format valid"
```

### Hook 2: No WIP or Temp Commits

```bash
#!/bin/bash
# .husky/commit-msg (additional check)

COMMIT_MSG=$(cat "$1")

if echo "$COMMIT_MSG" | grep -qiE '(WIP|temp|temporary|asdf|test commit)'; then
    echo "❌ WIP or temporary commit messages not allowed"
    echo ""
    echo "Either:"
    echo "  1. Write a proper commit message"
    echo "  2. Use 'git stash' for temporary work"
    exit 1
fi
```

### Hook 3: File Move Detection

```bash
#!/bin/bash
# .husky/pre-commit

# Check for moved files
MOVED_FILES=$(git diff --cached --name-status | grep '^R')

if [ -n "$MOVED_FILES" ]; then
    # Check if any moved files also have content changes
    while IFS=$'\t' read -r status old_path new_path; do
        if [ -n "$(git diff --cached "$new_path")" ]; then
            echo "⚠️  WARNING: File moved AND modified in same commit"
            echo "   $old_path → $new_path"
            echo ""
            echo "Best practice: Commit move separately from changes"
            echo ""
            read -p "Continue anyway? [y/N] " -n 1 -r
            echo
            if [[ ! $REPLY =~ ^[Yy]$ ]]; then
                exit 1
            fi
        fi
    done <<< "$MOVED_FILES"
fi
```

## Commit Frequency Anti-Patterns

### Anti-Pattern 1: The "Friday Dump"

❌ **BAD**:
```bash
# Friday afternoon
git add .
git commit -m "week's work"
# 47 files changed, 2,304 insertions, 891 deletions
```

✅ **GOOD**:
```bash
# Throughout the week, atomic commits
git commit -m "feat(auth): add login"      # 3 files, Tuesday
git commit -m "test(auth): add login tests" # 2 files, Wednesday
git commit -m "feat(auth): add logout"      # 2 files, Thursday
git commit -m "docs(auth): update API docs" # 1 file, Friday
```

### Anti-Pattern 2: The "Commit Spree"

❌ **BAD**:
```bash
git commit -m "add file"
git commit -m "fix typo"
git commit -m "fix another typo"
git commit -m "actually fix it"
git commit -m "for real this time"
# 5 commits in 2 minutes
```

✅ **GOOD**:
```bash
# Work on feature
# Use 'git commit --amend' for small fixes
# OR squash before pushing:
git rebase -i HEAD~5  # Squash into single commit
```

### Anti-Pattern 3: Mixed Concerns

❌ **BAD**:
```bash
git commit -m "feat: add login, fix button styling, update deps, refactor utils"
# Multiple unrelated changes
```

✅ **GOOD**:
```bash
git commit -m "feat(auth): add login endpoint"
git commit -m "style(ui): fix button alignment"
git commit -m "chore(deps): update axios to v1.6"
git commit -m "refactor(utils): extract date helpers"
```

## Atomic Writes Pattern (Roo Code)

**For critical files** (config, database, knowledge base):

```typescript
import fs from 'fs'
import path from 'path'

async function safeWriteFile(filePath: string, content: string) {
    const tempPath = `${filePath}.tmp.${Date.now()}`

    try {
        // 1. Write to temp file first
        await fs.promises.writeFile(tempPath, content, 'utf8')

        // 2. Atomic rename (preserves history)
        await fs.promises.rename(tempPath, filePath)

        // 3. Git add immediately
        exec(`git add "${filePath}"`)

    } catch (error) {
        // Cleanup temp file on failure
        if (fs.existsSync(tempPath)) {
            fs.unlinkSync(tempPath)
        }
        throw error
    }
}
```

**When to use**:
- `.ai-knowledge/` JSON files (critical data)
- `.swarm/memory.db` (learning database)
- Configuration files
- Any file where corruption = disaster

## Summary: Quick Reference

### Commit Format
```
<type>(<scope>): <description>

[body]

[footer]
```

### Commit Frequency
- After each logical unit
- When tests pass
- 3-10 commits per feature

### File Grouping
- Related changes together
- Implementation + tests together
- Config separate from code

### File Moves
1. `git mv old new`
2. Commit move immediately
3. Then edit
4. Commit changes separately

### Enforcement
- Pre-commit hooks check format
- Block WIP commits
- Warn on move + edit

### Learning
- Store successful patterns in AgentDB
- Track failures to avoid
- Build project-specific commit style

---

**Status**: ✅ Ready for implementation
**Next**: Install commit-msg and pre-commit hooks
