# Hooks-Based Enforcement

## Core Concept

**Transform "please do X" into "X always happens"**

Prompts are suggestions. Hooks are guarantees.

## Two Types of Hooks

### 1. Git Hooks
**Trigger**: Git operations (commit, push, etc.)
**Purpose**: Block bad code from entering version control
**Tools**: Husky (JS/TS), pre-commit (Python), custom scripts

### 2. Claude Code Hooks
**Trigger**: Tool use events (Edit, Write, etc.)
**Purpose**: Automate quality checks during development
**Tools**: Built into Claude Code

## Git Hooks Implementation

### Pre-Commit Hook Pattern

```bash
#!/bin/sh
# .husky/pre-commit or .git/hooks/pre-commit

echo "Running pre-commit quality gates..."

# 1. Check tests exist
TEST_COUNT=$(find src -name "*.test.ts" -o -name "*.spec.ts" | wc -l)
if [ "$TEST_COUNT" -eq 0 ]; then
  echo "ERROR: No test files found (e.g., src/**/*.test.ts)"
  echo "Add at least one test before committing."
  exit 1
fi

# 2. Run linter
echo "Linting..."
npm run lint || exit 1

# 3. Type check
echo "Type checking..."
npm run typecheck || exit 1

# 4. Run tests
echo "Running tests..."
npm test || exit 1

# 5. Check complexity
echo "Checking complexity..."
npm run complexity:check || exit 1

echo "✓ All pre-commit checks passed"
```

### Pre-Push Hook Pattern

```bash
#!/bin/sh
# .husky/pre-push

echo "Running pre-push checks..."

# Full repo linting and type checking
npm run lint || exit 1
npm run typecheck || exit 1

# Optional: Run full test suite
# (might be slow; consider running in CI instead)
npm test || exit 1

echo "✓ All pre-push checks passed"
```

### Python Pre-Commit Pattern

`.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.0
    hooks:
      - id: mypy

  - repo: local
    hooks:
      - id: pytest-check
        name: pytest-check
        entry: bash -c 'pytest --co -q | wc -l > /dev/null || exit 1'
        language: system
        pass_filenames: false
        always_run: true

      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
```

## Claude Code Hooks Implementation

### Hook Events

1. **PreToolUse**: Before AI calls a tool
2. **PostToolUse**: After AI successfully calls a tool
3. **Notification**: When AI sends a notification
4. **Stop**: When AI finishes responding

### PostToolUse Hook for Testing

**Config**: `.claude/hooks.json`
```json
{
  "hooks": [
    {
      "event": "PostToolUse",
      "tool": "Edit",
      "command": "npm test 2>&1 | tee .test-output.txt && grep -q 'PASS' .test-output.txt"
    },
    {
      "event": "PostToolUse",
      "tool": "Write",
      "command": "npm test 2>&1 | tee .test-output.txt && grep -q 'PASS' .test-output.txt"
    }
  ]
}
```

**Effect**: Every time AI edits or writes a file, tests automatically run. AI cannot proceed if tests fail.

### PostToolUse Hook for Complexity

```json
{
  "hooks": [
    {
      "event": "PostToolUse",
      "tool": "Edit",
      "command": "npm run complexity:check || echo 'WARNING: Complexity threshold exceeded'"
    }
  ]
}
```

### PreToolUse Hook for Safety

```json
{
  "hooks": [
    {
      "event": "PreToolUse",
      "tool": "Bash",
      "command": "echo 'About to run bash command. Verifying safety...' && sleep 1"
    }
  ]
}
```

## Hook Design Principles

### 1. Keep Hooks Fast
**Target**: < 5 seconds per hook
**Why**: Slow hooks get bypassed with `--no-verify`

**Optimization Strategies**:
- Run only on changed files (git hooks)
- Use incremental type checking
- Parallel execution where possible
- Cache results when safe

**Example** (only lint staged files):
```bash
# .husky/pre-commit
npx lint-staged
```

`.lintstagedrc.json`:
```json
{
  "*.ts": ["eslint --fix", "prettier --write"],
  "*.test.ts": ["npm test -- --findRelatedTests"]
}
```

### 2. Provide Clear Feedback

**Bad**:
```
Error: Tests failed
```

**Good**:
```
ERROR: 3 tests failed in src/auth.test.ts

  ✗ should validate email format
    Expected: false
    Received: true

  ✗ should reject invalid passwords
    AssertionError: expected 'valid' to equal 'invalid'

Fix these tests before committing.
To bypass (not recommended): git commit --no-verify
```

### 3. Make Bypassing Visible

**Pattern**: Don't hide the bypass option, but make it explicit and logged.

```bash
# .husky/pre-commit
if [ -n "$HUSKY_SKIP_HOOKS" ]; then
  echo "⚠️  WARNING: Hooks bypassed via HUSKY_SKIP_HOOKS"
  echo "⚠️  This commit may not meet quality standards"
  git log -1 --pretty=%B > .bypass-reason.txt
  exit 0
fi
```

### 4. Fail Fast, Fail Clearly

```bash
# Run checks in order of speed and likelihood of failure
# Fastest/most likely to fail first

npm run lint || exit 1          # Fast, often fails
npm run typecheck || exit 1     # Medium speed
npm test || exit 1               # Slower
npm run complexity:check || exit 1  # Fast but less likely to fail
```

## Hook Integration Patterns

### Pattern 1: Graduated Enforcement

**Commit**: Basic checks (lint, type, tests exist)
**Push**: Full checks (all tests, coverage, complexity)
**CI**: Comprehensive (security, performance, integration tests)

**Why**: Balance speed vs thoroughness at each stage.

### Pattern 2: Warning vs Blocking

**Warning** (non-zero exit but continues):
```bash
npm run complexity:check || echo "⚠️  Complexity warning"
# Continue even if complexity exceeded
```

**Blocking** (exits on failure):
```bash
npm run complexity:check || exit 1
# Cannot proceed if complexity exceeded
```

**Use warnings for**:
- New checks being phased in
- Non-critical quality metrics
- Informational feedback

**Use blocking for**:
- Tests must pass
- Linter must pass
- Critical security/type errors

### Pattern 3: Conditional Hooks

**Run different checks based on context**:
```bash
# .husky/pre-commit

# Check if this is a merge commit (skip extensive checks)
if git rev-parse -q --verify MERGE_HEAD; then
  echo "Merge commit detected, running minimal checks..."
  npm run lint || exit 1
  exit 0
fi

# Regular commit: full checks
npm run lint || exit 1
npm test || exit 1
```

### Pattern 4: Incremental Adoption

**Phase 1**: Warnings only
```bash
npm test || echo "⚠️  Tests failed (warning only)"
```

**Phase 2**: Blocking for new files only
```bash
if git diff --cached --name-only | grep -q "src/.*\.ts$"; then
  npm test || exit 1
fi
```

**Phase 3**: Blocking for all files
```bash
npm test || exit 1
```

## Hook Maintenance

### Testing Hooks

```bash
# Test pre-commit hook manually
.husky/pre-commit
echo $?  # Should be 0 on success, non-zero on failure

# Test with staged changes
git add <file>
.husky/pre-commit
```

### Debugging Hooks

```bash
# Add verbose output
set -x  # Print commands as they execute
set -e  # Exit on any error

# Add timing
time npm test
```

### Updating Hooks

```bash
# Hooks are just scripts - edit and test
vim .husky/pre-commit

# Commit hook changes
git add .husky/pre-commit
git commit -m "Update pre-commit hook to check coverage"
```

## Anti-Patterns to Avoid

### ❌ Hook Runs Too Long
**Problem**: > 30 seconds, developers bypass
**Solution**: Optimize or move to CI

### ❌ Hook Fails Mysteriously
**Problem**: "Tests failed" with no details
**Solution**: Show actual error output

### ❌ Hook Depends on External Services
**Problem**: Can't commit if service is down
**Solution**: Cache results, fail gracefully

### ❌ Hook Modifies Code
**Problem**: Git state changes during commit
**Solution**: Only validate, don't modify (or stage changes if you do)

### ❌ Hook Has Implicit Dependencies
**Problem**: Assumes specific tools installed
**Solution**: Check for dependencies, provide clear errors

## Hook Configuration for Our Project

### Recommended Setup

**Git Hooks** (.husky/):
- `pre-commit`: Tests exist, tests pass, lint passes, type check passes
- `pre-push`: Full lint, full type check
- `post-checkout`: Load relevant knowledge for branch

**Claude Code Hooks** (.claude/hooks.json):
- `PostToolUse` (Edit): Run tests, parse output
- `PostToolUse` (Write): Run tests if production code
- `Stop`: Update progress tracking

**Verification** (custom scripts):
- `scripts/verify-tests-ran.sh`: Parse test output
- `scripts/check-coverage-delta.sh`: Ensure coverage didn't drop
- `scripts/check-complexity.sh`: Complexity budget checks

## Success Metrics

- ✅ 0 commits with failing tests (100% enforcement)
- ✅ < 5 second hook execution time (fast enough not to bypass)
- ✅ 0 bypass attempts (or minimal and logged)
- ✅ Clear error messages (developers know what failed and why)
- ✅ Hooks enforced in CI (can't bypass by pushing directly)

## References

- quality-workflow-meta: Production-ready hook implementations
- Claude Code Hooks Guide: https://docs.claude.com/en/docs/claude-code/hooks-guide
- Husky: https://typicode.github.io/husky/
- pre-commit framework: https://pre-commit.com/
