# GitHub Integration: MCP Server vs CLI

## The Question

Should we use **GitHub MCP Server** or **`gh` CLI** for our PR workflow (ADR-014)?

---

## GitHub MCP Server

**Repository:** github/github-mcp-server
**Status:** ✅ Official, actively maintained (438 commits)
**Language:** Go (rewritten from TypeScript for performance)
**Maintained by:** GitHub (official)

### Capabilities

**Repository Management:**
- Browse code
- Search files
- Analyze commits
- Understand project structure

**Issue & PR Automation:**
- Create, update, manage issues
- Create, update PRs
- Add comments to PRs
- Read PR comments
- Manage PR reviews

**CI/CD Intelligence:**
- Monitor GitHub Actions workflows
- Analyze build failures
- Check run status

**Code Analysis:**
- Review security findings
- Check Dependabot alerts
- Code scanning results

**Team Collaboration:**
- Access discussions
- Manage notifications
- Team operations

### Recent Updates (2025)

**October 2025:** Server instructions added, better tools, improved usability
**October 2025:** GitHub Projects support added
**April 2025:** Public preview launched

**Active development:** ✅ Continuous improvements

---

## `gh` CLI Alternative

**Tool:** GitHub CLI
**Status:** ✅ Official, mature, widely used
**Maintained by:** GitHub (official)

### Capabilities

All GitHub operations via command-line:

```bash
# Issues
gh issue create --title "..." --body "..."
gh issue list
gh issue view 123

# Pull Requests
gh pr create --title "..." --body "..."
gh pr list
gh pr view 123
gh pr comment 123 --body "..."
gh pr checkout 123
gh pr merge 123

# Repositories
gh repo view
gh repo clone
gh repo create

# Actions/CI
gh run list
gh run view
gh workflow list

# API access
gh api /repos/{owner}/{repo}/pulls
```

**Advantages:**
- ✅ Explicit, transparent commands
- ✅ Scriptable (skills can call directly)
- ✅ Battle-tested, mature
- ✅ No context pollution
- ✅ Fast, direct operations

---

## Use Case Analysis

### Our PR Workflow (from ADR-014)

**PM-Mode Workflow:**
1. AI implements task
2. AI creates PR with:
   - Task description
   - Changes made
   - Test results
   - Self-evaluation notes
3. User reviews PR on GitHub
4. **User comments on specific lines**
5. **AI reads comments, makes fixes**
6. AI pushes updates
7. **AI replies to comments confirming fix**
8. Repeat until user merges

**Critical operations:**
- Create PR
- Read PR comments
- Push code updates
- Reply to PR comments

---

## MCP vs CLI Comparison

### Creating a PR

**MCP Approach:**
```typescript
// Conversational, multi-step
"Create a PR for this branch with title 'Add auth'
and include the test results in the description"

// AI uses MCP tools to:
// 1. Get current branch
// 2. Get commit history
// 3. Format PR body
// 4. Create PR
// 5. Return PR URL
```

**CLI Approach:**
```bash
# Explicit, single command
gh pr create \
  --title "Add Firebase authentication" \
  --body "$(cat <<'EOF'
## Summary
- Implemented Firebase auth
- Added tests
- Self-evaluation: PASS

## Test Results
$(cat test-results.txt)
EOF
)"
```

**Winner:** **CLI** - More transparent, faster, explicit

---

### Reading PR Comments

**MCP Approach:**
```typescript
// "Read all comments on PR #123"
// Returns structured data
{
  comments: [
    { id: 1, author: "user", body: "Fix this typo", line: 42 }
  ]
}
```

**CLI Approach:**
```bash
# Read comments
gh pr view 123 --json comments

# Or use API
gh api /repos/user/repo/pulls/123/comments
```

**Winner:** **Tie** - Both work fine, CLI is more explicit

---

### Complex Workflow: Read Comments, Fix, Reply

**MCP Approach:**
```typescript
// AI can chain operations in conversation:
// 1. "Read comments on PR 123"
// 2. AI analyzes comments
// 3. AI makes code fixes
// 4. AI commits changes
// 5. "Reply to comment 456 saying I fixed it"

// Stateful, conversational, context-aware
```

**CLI Approach:**
```bash
# Requires explicit scripting:
#!/bin/bash

# 1. Read comments
COMMENTS=$(gh api /repos/$REPO/pulls/$PR/comments)

# 2. Parse comments (more work)
# 3. Make fixes (AI does this)
# 4. Commit changes
git commit -m "Fix based on PR feedback"

# 5. Reply to comment
gh api /repos/$REPO/pulls/comments/$COMMENT_ID/replies \
  -f body="Fixed in latest commit"
```

**Winner:** **MCP** - Stateful context, easier multi-step workflows

---

## Analysis: When Each Excels

### CLI Excels When:
- ✅ Single, explicit operations
- ✅ Scripting predetermined workflows
- ✅ Want transparency and control
- ✅ Simple, direct commands

**Examples:**
- Create PR after task complete
- List open PRs
- Merge PR (user does this)
- Check CI status

### MCP Excels When:
- ✅ Multi-step, conversational workflows
- ✅ Context-dependent operations
- ✅ Complex decision trees
- ✅ Natural language interaction

**Examples:**
- "Read all PR comments, identify which need code changes, make the changes, reply to comments"
- "Check if there are any failing tests in the PR's CI, analyze the failures, suggest fixes"
- Complex PR triage and response

---

## Decision for Our Project

### Phase 1: Start with CLI

**Rationale (from ADR-003):**
> "Prefer direct CLI scripts over MCP middleware. Only use MCP when actually necessary."

**Our workflow can be scripted:**

```yaml
# github-pr.skill
scripts:
  create-pr: |
    gh pr create \
      --title "$TITLE" \
      --body "$BODY" \
      --assignee @me

  read-comments: |
    gh api /repos/$REPO/pulls/$PR/comments

  reply-comment: |
    gh api /repos/$REPO/pulls/comments/$COMMENT_ID/replies \
      -f body="$REPLY"

  check-ci: |
    gh pr checks $PR
```

**Advantages:**
- ✅ Aligned with ADR-003
- ✅ Simpler, more transparent
- ✅ No MCP context overhead
- ✅ Easier to debug

**Implementation:**
```typescript
// In our PR workflow
async function createPR(title: string, body: string) {
  // Call skill script directly
  await executeSkillScript('github-pr', 'create-pr', {
    TITLE: title,
    BODY: body
  });
}

async function readPRComments(prNumber: number) {
  const result = await executeSkillScript('github-pr', 'read-comments', {
    PR: prNumber
  });
  return JSON.parse(result);
}
```

### Phase 2: Evaluate MCP (After 10+ PRs)

**Metrics to track:**
1. How often do multi-step PR workflows occur?
2. How much manual scripting overhead?
3. Do we hit CLI limitations?
4. Would MCP's stateful context help?

**Decision criteria:**
- **If CLI is sufficient:** ✅ Keep using CLI
- **If complex workflows emerge:** Consider MCP

**Example scenario that might need MCP:**
```
User: "Review all open PRs, check if CI passed,
read comments, identify which ones are ready to merge,
and summarize the status of each."

// This requires:
// 1. List PRs
// 2. For each PR:
//    - Check CI status
//    - Read comments
//    - Analyze readiness
// 3. Generate summary

// CLI: Lots of scripting
// MCP: Natural conversation
```

---

## Recommendation

### ✅ Start with `gh` CLI

**Phase:** 0-2

**Why:**
1. **Aligns with ADR-003** - Prefer CLI
2. **Simpler** - No MCP overhead
3. **Transparent** - Explicit commands
4. **Sufficient** - Our workflow is scriptable
5. **Proven** - Mature, battle-tested

**Implementation:**
- Create `github-pr.skill` with CLI scripts
- Skills call `gh` commands directly
- AI uses skills for PR operations

---

### ⏸️ Consider MCP Later

**Phase:** 3+ (if needed)

**When to add:**
1. Complex, multi-step PR workflows become common
2. CLI scripting overhead becomes burdensome
3. Natural language PR operations add value
4. Data shows MCP would improve quality

**Validation (Principle 3):**
- Track pain points with CLI approach
- Measure scripting overhead
- Compare token usage: CLI scripts vs MCP conversation
- Make evidence-based decision

---

## Integration with Our System

### PR Workflow (from SPEC.md)

**7. GitHub PR Workflow:**

```typescript
// When Task Complete:
async function createPRFromTask(task: CompletedTask) {
  // 1. Create feature branch: ai/task-description-YYYYMMDD
  await git.createBranch(`ai/${task.name}-${date}`);

  // 2. Commit changes
  await git.commit(task.description);

  // 3. Run final self-evaluation
  const evaluation = await selfEvaluate(task);

  // 4. Create PR using gh CLI
  const prBody = `
## Task
${task.description}

## Changes Made
${task.changes}

## Test Results
${task.testResults}

## Self-Evaluation
${evaluation}
  `;

  await executeSkillScript('github-pr', 'create-pr', {
    TITLE: task.description,
    BODY: prBody
  });

  // 5. Assign to user (done in script)
}

// Reading PR Comments:
async function readPRFeedback(prNumber: number) {
  // AI reads comments using gh CLI
  const comments = await executeSkillScript('github-pr', 'read-comments', {
    PR: prNumber
  });

  // Parse comments
  const feedback = parseComments(comments);

  // Update .plan/current-task.md with feedback
  await updatePlan(feedback);

  // Make changes...

  // Reply to comments
  for (const comment of feedback) {
    await executeSkillScript('github-pr', 'reply-comment', {
      COMMENT_ID: comment.id,
      REPLY: `Fixed in commit ${commitHash}`
    });
  }
}
```

**This works fine with CLI!**

---

## Experiments to Run

### Experiment: CLI vs MCP for PR Workflow

**Hypothesis:** "CLI approach is sufficient for our PR workflow with no quality degradation vs MCP."

**Design:**
- **Phase 1:** Implement with CLI (10 PRs)
- **Phase 2:** Implement same workflow with MCP (10 PRs)
- **Compare:** Token usage, time, quality, user satisfaction

**Metrics:**
- Token usage for PR operations
- Time from task complete → PR created
- Quality of PR descriptions
- Success rate of comment-based fixes
- User satisfaction (manual review)

**Expected:**
- CLI: Lower token usage (explicit commands)
- CLI: Faster (direct operations)
- MCP: Better for complex multi-step workflows (if they exist)

**Decision criteria:**
- If CLI ≥ MCP: Keep CLI
- If MCP significantly better: Switch to MCP

**Status:** Phase 2 (after implementing PR workflow)

---

## Comparison Summary

| Aspect | `gh` CLI | GitHub MCP | Winner |
|--------|----------|------------|--------|
| **Simplicity** | ✅ Simple commands | ❌ Protocol overhead | CLI |
| **Transparency** | ✅ Explicit | ⚠️ Abstracted | CLI |
| **Speed** | ✅ Direct | ⚠️ Protocol overhead | CLI |
| **Scriptability** | ✅ Easy | ⚠️ Complex | CLI |
| **Multi-step workflows** | ⚠️ Requires scripting | ✅ Stateful context | MCP |
| **Natural language** | ❌ Explicit commands | ✅ Conversational | MCP |
| **Context pollution** | ✅ None | ⚠️ Potential | CLI |
| **Maturity** | ✅ Mature | ⚠️ Newer | CLI |
| **Official support** | ✅ Yes | ✅ Yes | Tie |
| **Alignment with ADR-003** | ✅ Perfect | ❌ Middleware | CLI |

**Overall Winner for Phase 1:** **`gh` CLI**

---

## Decision

**START WITH:** `gh` CLI

**RE-EVALUATE:** After Phase 2 (10+ PRs)

**SWITCH TO MCP IF:** Complex multi-step workflows prove valuable

**Rationale:**
1. ✅ Aligns with ADR-003 (prefer CLI)
2. ✅ Simpler implementation
3. ✅ Our workflow is scriptable
4. ✅ Less overhead
5. ✅ Can add MCP later if needed

---

## Next Steps

1. ✅ Use `gh` CLI for PR workflow
2. Create `github-pr.skill` with scripts
3. Implement PR creation, comment reading, replies
4. Track pain points and limitations
5. After 10+ PRs: Re-evaluate vs MCP
6. Make data-driven decision (Principle 3: VALIDATE)

---

**Research Date:** 2025-11-03
**Status:** ✅ Decision - Start with CLI, re-evaluate later
