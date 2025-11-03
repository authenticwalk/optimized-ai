# PR Review Best Practices

**Research Quality**: ⭐⭐⭐⭐ (Spec-Kit validation patterns, GitHub research)
**Status**: ✅ IMPLEMENTATION-READY

## AI-Assisted PR Review Pattern

### Using `gh` CLI (Preferred over MCP)

**Why CLI**: Direct, transparent, no middleware overhead (validated by MCP research)

```bash
#!/bin/bash
# scripts/review-pr.sh

PR_NUMBER=$1

if [ -z "$PR_NUMBER" ]; then
    echo "Usage: $0 <pr-number>"
    exit 1
fi

# 1. Fetch PR details
echo "📋 Fetching PR #$PR_NUMBER..."
gh pr view $PR_NUMBER --json title,body,files,commits > .pr-context.json

# 2. Get the diff
echo "📝 Fetching diff..."
gh pr diff $PR_NUMBER > .pr-diff.txt

# 3. Extract changed files
FILES=$(gh pr view $PR_NUMBER --json files -q '.files[].path')

echo "🔍 Reviewing changes in:"
echo "$FILES"
```

## Self-Review Checklist (Before Requesting Review)

### Code Quality

```markdown
## Code Quality Checklist

### Functionality
- [ ] Code does what PR description says
- [ ] Edge cases handled
- [ ] Error handling implemented
- [ ] No hardcoded values (use config/env)

### Readability
- [ ] Variable names are clear
- [ ] Function names describe behavior
- [ ] Complex logic has comments
- [ ] No commented-out code
- [ ] No debug console.logs

### Performance
- [ ] No N+1 queries
- [ ] Large lists are paginated
- [ ] Heavy operations are async
- [ ] No unnecessary re-renders (React)

### Security
- [ ] Input validation
- [ ] No SQL injection risks
- [ ] No XSS vulnerabilities
- [ ] Secrets not in code
- [ ] Authentication/authorization correct

### Testing
- [ ] New code has tests
- [ ] Tests cover edge cases
- [ ] Tests are not tautological
- [ ] Tests follow FIRST principles
- [ ] Coverage didn't decrease

### Dependencies
- [ ] New deps justified
- [ ] No known vulnerabilities
- [ ] License compatible
- [ ] Bundle size acceptable
```

### Code Complexity (Pheromind Standards)

```bash
#!/bin/bash
# scripts/check-complexity.sh

# Check function size (<50 lines per Pheromind)
echo "Checking function sizes..."
find src -name "*.ts" -exec awk '/^function|^const.*=.*\(/ {start=NR} /^}/ && start {lines=NR-start; if(lines>50) print FILENAME":"start" - Function exceeds 50 lines ("lines")"; start=0}' {} \;

# Check file size (<500 lines per Pheromind)
echo "Checking file sizes..."
find src -name "*.ts" -exec wc -l {} \; | awk '$1 > 500 {print $2 " exceeds 500 lines (" $1 ")"}'

# Check cyclomatic complexity (≤10 per quality-workflow-meta)
npx fta src --format json > .fta-report.json
node scripts/check-fta-cap.mjs || {
    echo "❌ Complexity threshold exceeded"
    exit 1
}
```

## Automated Review Comments

### Pattern 1: Required Changes

```typescript
// scripts/auto-review-issues.ts

interface ReviewIssue {
    path: string
    line: number
    message: string
    severity: 'error' | 'warning' | 'info'
}

function detectIssues(diff: string): ReviewIssue[] {
    const issues: ReviewIssue[] = []

    // Detect console.log
    if (diff.includes('console.log(')) {
        issues.push({
            path: extractPath(diff),
            line: extractLine(diff),
            message: 'Remove console.log before merge',
            severity: 'error'
        })
    }

    // Detect hardcoded secrets
    if (diff.match(/api[_-]?key\s*=\s*['"][^'"]+['"]/i)) {
        issues.push({
            path: extractPath(diff),
            line: extractLine(diff),
            message: 'Possible hardcoded secret - use environment variables',
            severity: 'error'
        })
    }

    // Detect TODO comments
    if (diff.includes('// TODO')) {
        issues.push({
            path: extractPath(diff),
            line: extractLine(diff),
            message: 'TODO comment - resolve or create ticket',
            severity: 'warning'
        })
    }

    // Detect missing tests
    if (!diff.includes('.test.ts') && !diff.includes('.spec.ts')) {
        issues.push({
            path: 'N/A',
            line: 0,
            message: 'No test files in this PR',
            severity: 'warning'
        })
    }

    return issues
}

// Post comments via gh CLI
function postReviewComments(prNumber: number, issues: ReviewIssue[]) {
    issues.forEach(issue => {
        if (issue.severity === 'error') {
            const comment = `**${issue.severity.toUpperCase()}** at ${issue.path}:${issue.line}\n\n${issue.message}`

            // Post review comment
            exec(`gh pr review ${prNumber} --comment -b "${comment}"`)
        }
    })
}
```

### Pattern 2: Positive Feedback

**What to look for**:
- Good test coverage
- Clear variable names
- Proper error handling
- Good documentation

```typescript
function detectGoodPractices(diff: string): string[] {
    const positives: string[] = []

    // High test coverage
    if (countLines(diff, '.test.ts') > countLines(diff, '.ts') * 0.5) {
        positives.push('✅ Excellent test coverage')
    }

    // Error handling
    if (diff.includes('try {') && diff.includes('catch (')) {
        positives.push('✅ Good error handling')
    }

    // TypeScript types
    if (!diff.includes(': any')) {
        positives.push('✅ No `any` types - good type safety')
    }

    return positives
}

// Post encouraging comment
function postPositiveFeedback(prNumber: number, positives: string[]) {
    if (positives.length > 0) {
        const comment = `Great work! 🎉\n\n${positives.join('\n')}`
        exec(`gh pr review ${prNumber} --comment -b "${comment}"`)
    }
}
```

## Review Template

```markdown
## Review of PR #{{PR_NUMBER}}

### Summary
<!-- 1-2 sentence summary of what this PR does -->

### Strengths
<!-- What's done well? -->
- Clear commit messages
- Good test coverage
- Well-documented code

### Issues Found

#### Critical (Must Fix)
- [ ] **Security**: Hardcoded API key in `src/config.ts:42`
- [ ] **Tests**: Missing test for edge case in `validateInput`

#### Important (Should Fix)
- [ ] **Performance**: N+1 query in `getUserPosts` - use JOIN
- [ ] **Code Quality**: Function `processData` exceeds 50 lines

#### Minor (Nice to Have)
- [ ] **Style**: Inconsistent variable naming (camelCase vs snake_case)
- [ ] **Documentation**: Add JSDoc for public API

### Suggestions
<!-- Improvements for future PRs -->
- Consider extracting complex logic into separate functions
- Could add integration test for the full flow

### Decision
- [ ] ✅ Approve (no issues or minor only)
- [ ] 💬 Comment (suggestions, no blocking issues)
- [ ] 🔄 Request Changes (critical/important issues found)

### Next Steps
1. Address critical issues
2. Push fixes to branch
3. Re-request review
```

## Learning from Reviews (AgentDB Integration)

### Store Review Patterns

```sql
-- Successful pattern (PR approved quickly)
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'comprehensive tests + clear docs + no console.logs',
    'pr_review',
    0.91,
    'approved_first_review'
);

-- Common issue
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'missing error handling',
    'pr_review',
    'Reviewer requested try/catch blocks for async operations'
);

-- Causal link
INSERT INTO causal_links (cause, effect, confidence)
VALUES (
    'high_test_coverage',
    'fast_pr_approval',
    0.88
);
```

### Query Before Review

```sql
-- What issues commonly appear in reviews?
SELECT pattern, COUNT(*) as frequency
FROM failures
WHERE context = 'pr_review'
GROUP BY pattern
ORDER BY frequency DESC
LIMIT 10;

-- What leads to fast approval?
SELECT cause, confidence
FROM causal_links
WHERE effect = 'fast_pr_approval'
ORDER BY confidence DESC;
```

## Common Review Issues (From Research)

### Issue 1: Missing Tests

**Detection**:
```bash
# Check if PR modifies source but not tests
SRC_CHANGES=$(gh pr diff $PR_NUMBER | grep -c '^\+.*\.ts$')
TEST_CHANGES=$(gh pr diff $PR_NUMBER | grep -c '^\+.*\.test\.ts$')

if [ $SRC_CHANGES -gt 0 ] && [ $TEST_CHANGES -eq 0 ]; then
    echo "⚠️  Source code changed but no test changes"
fi
```

### Issue 2: Large PR (>1000 lines)

**Detection**:
```bash
LINES_CHANGED=$(gh pr view $PR_NUMBER --json additions,deletions -q '.additions + .deletions')

if [ $LINES_CHANGED -gt 1000 ]; then
    echo "⚠️  PR is large ($LINES_CHANGED lines). Consider splitting."
fi
```

### Issue 3: Merge Conflicts

**Detection**:
```bash
# Check if PR has conflicts
CONFLICTS=$(gh pr view $PR_NUMBER --json mergeable -q '.mergeable')

if [ "$CONFLICTS" = "CONFLICTING" ]; then
    echo "❌ PR has merge conflicts - resolve before review"
    exit 1
fi
```

### Issue 4: Failing CI

**Detection**:
```bash
# Check CI status
CI_STATUS=$(gh pr view $PR_NUMBER --json statusCheckRollup -q '.statusCheckRollup[].state')

if echo "$CI_STATUS" | grep -q "FAILURE"; then
    echo "❌ CI checks failing - fix before review"
    exit 1
fi
```

## Automated Review Workflow

```bash
#!/bin/bash
# scripts/automated-pr-review.sh

PR_NUMBER=$1

echo "🤖 Starting automated review of PR #$PR_NUMBER..."

# 1. Basic validation
./scripts/validate-pr.sh $PR_NUMBER || exit 1

# 2. Complexity check
./scripts/check-complexity.sh || exit 1

# 3. Security scan
npm run scan-secrets || exit 1

# 4. Detect review issues
node scripts/auto-review-issues.ts $PR_NUMBER

# 5. Generate review comment
./scripts/generate-review-comment.sh $PR_NUMBER

echo "✅ Automated review complete"
echo "📝 Review posted to PR"
```

## Review Approval Criteria

### Auto-Approve (Low Risk)

Automatically approve if ALL true:
- [ ] All CI checks pass
- [ ] Coverage ≥80% and didn't decrease
- [ ] No security issues found
- [ ] File sizes <500 lines
- [ ] Function complexity ≤10
- [ ] PR size <400 lines
- [ ] All tests pass
- [ ] No console.logs or TODOs

```bash
#!/bin/bash
# scripts/auto-approve-if-safe.sh

if can_auto_approve $PR_NUMBER; then
    gh pr review $PR_NUMBER --approve -b "✅ Auto-approved: All quality checks passed"
    gh pr merge $PR_NUMBER --auto --squash
else
    echo "⚠️  Manual review required"
fi
```

### Request Changes (High Risk)

Automatically request changes if ANY true:
- [ ] Tests failing
- [ ] Security issues found
- [ ] Hardcoded secrets
- [ ] Coverage decreased >5%
- [ ] Complexity >15
- [ ] File >800 lines

## Summary: Quick Reference

### Self-Review Before Requesting
1. Run `./scripts/check-complexity.sh`
2. Run `npm audit`
3. Run `npm test -- --coverage`
4. Review own diff for issues
5. Fill out PR review checklist

### Automated Review Steps
1. Fetch PR with `gh pr view`
2. Run security scan
3. Check complexity
4. Detect common issues
5. Post review comments

### Review Criteria
- Tests present and passing
- No security issues
- Complexity thresholds met
- Clear code and docs
- No debug code left

### Learning
- Store successful review patterns
- Track common issues
- Build project-specific review guidelines

---

**Status**: ✅ Ready for implementation
**Next**: Create automated review scripts
