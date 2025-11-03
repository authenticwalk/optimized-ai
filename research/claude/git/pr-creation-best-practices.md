# Pull Request Creation Best Practices

**Research Quality**: ⭐⭐⭐⭐⭐ (Spec-Kit, Roo Code, quality-workflow-meta)
**Status**: ✅ IMPLEMENTATION-READY

## When to Create PR

### Timing Rules

✅ **Create PR when**:
- Feature is complete (not WIP)
- All tests pass locally
- Linting passes
- Self-review completed
- Ready for team review

❌ **Don't create PR when**:
- Tests are failing ("I'll fix them later")
- Incomplete feature ("just want feedback")
- Debugging in progress
- No tests written yet

**Exception**: Draft PRs for early feedback (mark as draft)

## Pre-PR Checklist (Self-Evaluation)

**From AI Testing research + Spec-Kit validation patterns**:

### Automated Checks

```bash
#!/bin/bash
# scripts/pre-pr-check.sh

echo "🔍 Running pre-PR validation..."

# 1. Run all tests
echo "Running tests..."
npm test || {
    echo "❌ Tests failed - fix before creating PR"
    exit 1
}

# 2. Run linter
echo "Running linter..."
npm run lint || {
    echo "❌ Linting failed - run 'npm run lint:fix'"
    exit 1
}

# 3. Type check
echo "Type checking..."
npm run typecheck || {
    echo "❌ Type errors found"
    exit 1
}

# 4. Check commit messages
echo "Checking commit format..."
git log origin/main..HEAD --pretty=%s | while read msg; do
    if ! echo "$msg" | grep -qE '^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: '; then
        echo "❌ Non-conventional commit: $msg"
        exit 1
    fi
done

# 5. Check for secrets (from Pheromind research)
echo "Scanning for secrets..."
npm run scan-secrets || {
    echo "❌ Potential secrets found"
    exit 1
}

echo "✅ All pre-PR checks passed!"
```

### Manual Checklist

```markdown
## Pre-PR Checklist

### Code Quality
- [ ] All tests pass (`npm test`)
- [ ] Linting passes (`npm run lint`)
- [ ] Type checking passes (`npm run typecheck`)
- [ ] No console.log or debug code left
- [ ] No commented-out code
- [ ] Functions <50 lines (Pheromind standard)
- [ ] Files <500 lines (Pheromind standard)
- [ ] Cyclomatic complexity ≤10 (quality-workflow-meta)

### Testing
- [ ] New code has tests
- [ ] Test coverage didn't decrease
- [ ] Integration tests for new features
- [ ] Edge cases covered
- [ ] Tests follow FIRST principles (Fast, Independent, Repeatable, Self-validating, Timely)

### Documentation
- [ ] README updated if needed
- [ ] API docs updated if needed
- [ ] Inline comments for complex logic
- [ ] CHANGELOG.md updated

### Security
- [ ] No hardcoded credentials
- [ ] No API keys in code
- [ ] Dependencies scanned (`npm audit`)
- [ ] Input validation added

### Git
- [ ] Commits follow conventional format
- [ ] No merge commits (rebased on main)
- [ ] Logical commit grouping
- [ ] File moves in separate commits
```

## PR Title & Description Format

### Title Format

**Use conventional commit style**:
```
<type>(<scope>): <description>
```

**Examples**:
```
feat(auth): Add JWT token refresh mechanism
fix(api): Handle null response in user endpoint
refactor(utils): Extract date formatting to helpers
```

### Description Template (Spec-Kit Pattern)

```markdown
## Summary
<!-- 1-3 sentences: What does this PR do? -->

Adds JWT token refresh mechanism to prevent session timeouts. Users will
automatically get new tokens before expiry without re-logging in.

## Changes
<!-- Bulleted list of specific changes -->

- Added `refreshToken` endpoint to auth controller
- Implemented token rotation with 15-minute refresh window
- Updated client to auto-refresh on 401 responses
- Added integration tests for token refresh flow

## Testing
<!-- How was this tested? -->

- [ ] Unit tests added for token refresh logic (100% coverage)
- [ ] Integration tests for refresh flow (happy path + edge cases)
- [ ] Manual testing in dev environment
- [ ] Tested token expiry scenarios

## Screenshots/Demos
<!-- If UI changes, include screenshots or GIFs -->

_N/A - Backend only changes_

## Breaking Changes
<!-- Any breaking changes? How to migrate? -->

No breaking changes. Existing tokens continue to work.

## Deployment Notes
<!-- Anything special needed for deployment? -->

- Environment variable `JWT_REFRESH_WINDOW` defaults to 900s (15 min)
- No database migrations required

## Checklist
- [x] Tests added/updated
- [x] Documentation updated
- [x] Linting passes
- [x] Type checking passes
- [x] No secrets in code
- [x] Follows conventional commits
```

## Branch Naming

**Pattern** (from Spec-Kit research):
```
<type>/<scope>/<description>
OR
<type>/<ticket-id>-<description>
```

**Examples**:
```
feat/auth/jwt-refresh
fix/api/null-handling
refactor/utils/date-helpers
feat/PROJ-123-add-login
```

**Automated branch creation** (Spec-Kit pattern):
```bash
#!/bin/bash
# scripts/create-feature-branch.sh

TYPE=$1  # feat, fix, refactor, etc.
SCOPE=$2  # auth, api, ui, etc.
DESC=$3   # Short description

BRANCH_NAME="${TYPE}/${SCOPE}/${DESC}"

git checkout main
git pull origin main
git checkout -b "$BRANCH_NAME"

echo "✅ Created branch: $BRANCH_NAME"
echo "📝 First commit should be: ${TYPE}(${SCOPE}): ${DESC}"
```

## Creating PR via CLI

**Using `gh` CLI** (preferred over MCP):

```bash
#!/bin/bash
# scripts/create-pr.sh

# 1. Run pre-PR checks
./scripts/pre-pr-check.sh || exit 1

# 2. Get branch name and derive info
BRANCH=$(git rev-parse --abbrev-ref HEAD)
MAIN_BRANCH="main"  # Or detect from git

# 3. Ensure pushed to remote
git push -u origin "$BRANCH" || exit 1

# 4. Generate PR title from first commit
PR_TITLE=$(git log "${MAIN_BRANCH}..HEAD" --pretty=%s | tail -1)

# 5. Generate PR body
PR_BODY=$(cat <<'EOF'
## Summary
[Auto-generated from commits - EDIT THIS]

## Changes
$(git log ${MAIN_BRANCH}..HEAD --pretty="- %s")

## Testing
- [ ] Tests added/updated
- [ ] Manual testing completed

## Checklist
- [x] All tests pass
- [x] Linting passes
- [x] No secrets in code
EOF
)

# 6. Create PR using gh CLI
gh pr create \
    --title "$PR_TITLE" \
    --body "$PR_BODY" \
    --base "$MAIN_BRANCH" \
    --head "$BRANCH"

echo "✅ PR created successfully"
echo "🔗 View at: $(gh pr view --web)"
```

## Learning from PR Outcomes (AgentDB Integration)

**Store PR patterns**:

```sql
-- After PR approved quickly
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'conventional title + comprehensive description + tests',
    'pr_creation',
    0.92,
    'approved_first_review'
);

-- After PR needs rework
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'minimal description + no test plan',
    'pr_creation',
    'Reviewer requested more context and test coverage details'
);

-- Causal link
INSERT INTO causal_links (cause, effect, confidence)
VALUES (
    'comprehensive_pr_description',
    'fast_approval',
    0.85
);
```

**Query before creating PR**:
```sql
-- What leads to fast PR approval?
SELECT cause, confidence
FROM causal_links
WHERE effect IN ('fast_approval', 'approved_first_review')
ORDER BY confidence DESC;

-- Expected results after learning:
-- comprehensive_description → fast_approval (0.89)
-- all_tests_passing → fast_approval (0.92)
-- includes_screenshots → fast_approval (0.81)
```

## PR Size Guidelines (Roo Code Research)

**Optimal PR size**:
- 200-400 lines changed (ideal)
- <800 lines (acceptable)
- >1000 lines (split into multiple PRs)

**Why**:
- Reviewers can't properly review >1000 lines
- Smaller PRs get faster feedback
- Easier to understand and test
- Lower risk of introducing bugs

**How to split large PRs**:
```bash
# Instead of one 2000-line PR:
feat/big-feature

# Create sequence of smaller PRs:
feat/big-feature-part1-models
feat/big-feature-part2-api
feat/big-feature-part3-ui
feat/big-feature-part4-tests
```

## Draft PRs for Early Feedback

**When to use**:
- Want feedback on approach before full implementation
- Complex feature with uncertain design
- Breaking changes that need team buy-in

**How to create**:
```bash
gh pr create --draft \
    --title "feat(auth): [WIP] JWT refresh mechanism" \
    --body "Early draft for feedback on approach. Not ready for merge."
```

**Convert to ready when done**:
```bash
gh pr ready
```

## Common Mistakes to Avoid

### ❌ Mistake 1: "PR as backup"
```
Creating PR just to save work to remote.
Use branches and push, don't create PR until ready.
```

### ❌ Mistake 2: "The everything PR"
```
feat: add login, update ui, refactor utils, fix bugs, update deps
(10 files changed, 2,431 insertions, 892 deletions)

Split into logical PRs!
```

### ❌ Mistake 3: "No description"
```
Title: "updates"
Body: (empty)

Reviewers have no context!
```

### ❌ Mistake 4: "Ignoring CI failures"
```
"CI is failing but that's a known issue"

Fix it before creating PR!
```

## Summary: Quick Reference

### Pre-PR
1. Run `./scripts/pre-pr-check.sh`
2. All tests pass
3. Linting passes
4. Self-review completed

### PR Creation
1. Push branch to remote
2. Run `gh pr create` with template
3. Fill in summary, changes, testing
4. Add screenshots if UI changes
5. Mark as draft if not ready

### Title & Description
- Title: Conventional commit format
- Description: Use template (summary, changes, testing, checklist)
- Link to tickets if applicable

### Learning
- Store successful PR patterns in AgentDB
- Track what leads to fast approval
- Build project-specific PR style

---

**Status**: ✅ Ready for implementation
**Next**: Create `scripts/create-pr.sh` and PR template
