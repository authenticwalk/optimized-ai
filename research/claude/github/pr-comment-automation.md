# GitHub PR Comment Triggers & Automation

**Research Quality**: ⭐⭐⭐⭐ (GitHub MCP research, CLI patterns)
**Status**: ✅ IMPLEMENTATION-READY
**Approach**: GitHub Actions + `gh` CLI (no MCP middleware)

## Use Case: Fix Issues from PR Comments

**Scenario**: Reviewer comments "Fix the typo in README", trigger AI to fix and push

## Architecture

```
PR Comment → GitHub Webhook → GitHub Action → AI Agent → Fix → Push → Comment Update
```

**Why GitHub Actions** (not MCP server):
- Runs on GitHub infrastructure
- Direct access to repo
- Secure secrets management
- Free for public repos, generous limits for private

## Implementation

### Step 1: GitHub Action Workflow

**File**: `.github/workflows/pr-comment-trigger.yml`

```yaml
name: AI PR Comment Response

on:
  issue_comment:
    types: [created]

jobs:
  respond-to-comment:
    # Only run on PR comments (not regular issues)
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/ai-fix')

    runs-on: ubuntu-latest

    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Checkout PR branch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.issue.pull_request.head.ref }}
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Extract fix request
        id: extract
        run: |
          COMMENT="${{ github.event.comment.body }}"
          # Remove /ai-fix prefix, get the actual request
          REQUEST=$(echo "$COMMENT" | sed 's|^/ai-fix ||')
          echo "request=$REQUEST" >> $GITHUB_OUTPUT

      - name: Call AI to fix issue
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          FIX_REQUEST: ${{ steps.extract.outputs.request }}
          PR_NUMBER: ${{ github.event.issue.number }}
        run: |
          node scripts/ai-fix-from-comment.js

      - name: Commit and push fix
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

          if git diff --quiet; then
            echo "No changes made"
          else
            git add -A
            git commit -m "fix: address review comment - ${{ steps.extract.outputs.request }}"
            git push
          fi

      - name: Reply to comment
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh pr comment ${{ github.event.issue.number }} \
            --body "✅ Fixed! See latest commit."
```

### Step 2: AI Fix Script

**File**: `scripts/ai-fix-from-comment.js`

```javascript
#!/usr/bin/env node

const Anthropic = require('@anthropic-ai/sdk')
const { execSync } = require('child_process')
const fs = require('fs')

async function fixFromComment() {
    const anthropic = new Anthropic({
        apiKey: process.env.ANTHROPIC_API_KEY
    })

    const request = process.env.FIX_REQUEST
    const prNumber = process.env.PR_NUMBER

    console.log(`Fixing: ${request}`)

    // Get PR context
    const prDiff = execSync(`gh pr diff ${prNumber}`).toString()
    const changedFiles = execSync(`gh pr view ${prNumber} --json files -q '.files[].path'`)
        .toString()
        .split('\n')
        .filter(Boolean)

    // Build prompt for Claude
    const prompt = `You are an AI assistant helping with a pull request.

A reviewer commented: "${request}"

Your task: Make the necessary changes to address this comment.

Changed files in this PR:
${changedFiles.join('\n')}

Recent diff:
${prDiff}

Make ONLY the changes needed to address the comment. Be precise and minimal.

For each file you need to modify, respond in this format:
FILE: path/to/file.ts
CHANGE: <description of change>
<file contents after change>
---
`

    // Call Claude
    const response = await anthropic.messages.create({
        model: 'claude-sonnet-4-5-20250929',
        max_tokens: 4096,
        messages: [{
            role: 'user',
            content: prompt
        }]
    })

    // Parse response and apply changes
    const content = response.content[0].text
    const fileBlocks = content.split('FILE: ').slice(1)

    fileBlocks.forEach(block => {
        const [filePath, ...rest] = block.split('\n')
        const fileContent = rest.join('\n').split('---')[0].trim()

        console.log(`Writing ${filePath}...`)
        fs.writeFileSync(filePath, fileContent, 'utf8')
    })

    console.log('✅ Changes applied')
}

fixFromComment().catch(console.error)
```

### Step 3: Usage

**Reviewer comments on PR**:
```
/ai-fix Fix the typo in README line 42: "recieve" should be "receive"
```

**What happens**:
1. GitHub Action triggered
2. Checks out PR branch
3. Calls AI with fix request
4. AI makes the change
5. Commits and pushes to PR branch
6. Replies to comment: "✅ Fixed! See latest commit."

## Command Variations

### Simple Fix
```
/ai-fix Fix typo in README
```

### Refactor Request
```
/ai-fix Extract the validation logic in src/auth.ts into a separate function
```

### Test Addition
```
/ai-fix Add test for edge case when user is null
```

### Documentation
```
/ai-fix Add JSDoc comments to the createUser function
```

## Advanced: AI Review & Auto-Fix

**Trigger**: When PR is created, AI reviews and posts comments

```yaml
name: AI Auto Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Get PR diff
        id: diff
        run: |
          gh pr diff ${{ github.event.pull_request.number }} > pr.diff
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: AI Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          node scripts/ai-review-pr.js ${{ github.event.pull_request.number }}

      - name: Post review comments
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh pr review ${{ github.event.pull_request.number }} \
            --comment \
            --body-file review-comments.md
```

## Safety & Permissions

### Required Secrets

In GitHub repo settings → Secrets:
- `ANTHROPIC_API_KEY`: For Claude API access

### Branch Protection Rules

**Prevent AI from breaking things**:
1. Settings → Branches → Add rule for `main`
2. Enable:
   - Require pull request reviews
   - Require status checks to pass
   - Require branches to be up to date

**AI can push to PR branches but not merge to main**

### Cost Control

**Limit AI usage**:
```yaml
# Only allow certain users to trigger
if: contains(fromJson('["username1", "username2"]'), github.event.comment.user.login)
```

**Rate limiting**:
```javascript
// Check rate limits before calling API
const DAILY_LIMIT = 100
const count = await getUsageCount()

if (count > DAILY_LIMIT) {
    console.log('Daily limit reached')
    process.exit(0)
}
```

## Error Handling

```yaml
- name: Handle AI errors
  if: failure()
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    gh pr comment ${{ github.event.issue.number }} \
      --body "❌ Failed to apply fix. Please make the change manually."
```

## Testing Locally

**Test the fix script without GitHub Actions**:

```bash
# Set environment variables
export ANTHROPIC_API_KEY="your-key"
export FIX_REQUEST="Fix typo in README"
export PR_NUMBER="123"

# Run script
node scripts/ai-fix-from-comment.js
```

## Learning from Fixes (AgentDB Integration)

**Store successful fixes**:
```sql
-- After successful automated fix
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'typo fix via /ai-fix command',
    'github_automation',
    0.85,
    'success'
);

-- Track failures
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'complex refactor via /ai-fix',
    'github_automation',
    'AI made incorrect changes, manual fix required'
);
```

## Alternative: Simpler Webhook Approach

**If you don't want GitHub Actions**, use webhook to external server:

```javascript
// server.js - Express server
app.post('/github-webhook', async (req, res) => {
    const { action, comment, issue } = req.body

    if (action === 'created' && comment.body.startsWith('/ai-fix')) {
        const request = comment.body.replace('/ai-fix ', '')

        // Clone repo
        execSync(`git clone ${issue.pull_request.html_url}`)

        // Make AI fix
        await makeAIFix(request)

        // Push changes
        execSync(`git push`)

        // Reply to comment
        await octokit.issues.createComment({
            owner,
            repo,
            issue_number: issue.number,
            body: '✅ Fixed!'
        })
    }

    res.sendStatus(200)
})
```

**Setup webhook**: Repo → Settings → Webhooks → Add webhook
- Payload URL: `https://your-server.com/github-webhook`
- Content type: `application/json`
- Events: Issue comments

## Summary: Quick Reference

### Setup
1. Create GitHub Action workflow (`.github/workflows/pr-comment-trigger.yml`)
2. Add AI fix script (`scripts/ai-fix-from-comment.js`)
3. Add `ANTHROPIC_API_KEY` secret to repo
4. Test with `/ai-fix` command

### Usage
```
/ai-fix <description of what to fix>
```

### What It Does
1. Triggers GitHub Action
2. AI reads PR context
3. Makes requested changes
4. Commits and pushes
5. Replies to comment

### Safety
- Only runs on PRs (not issues)
- Requires specific command prefix
- Branch protection prevents direct merge
- Rate limiting prevents abuse

---

**Status**: ✅ Ready for implementation
**Next**: Create GitHub Action workflow and test
