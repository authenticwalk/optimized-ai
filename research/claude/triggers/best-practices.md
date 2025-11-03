# Claude Code Best Practices

Compiled from official Anthropic guidance, community patterns, and real-world usage.

## Setup & Configuration

### CLAUDE.md Files

**Purpose**: Document project-specific guidelines, commands, and conventions

**Location Priority**:
1. Repo root (`./ CLAUDE.md`) - Project-specific
2. Parent directory - Monorepo shared
3. Child directories - Component-specific
4. Home folder (`~/.claude/CLAUDE.md`) - Personal defaults

**Content**:
```markdown
# Project Name

## Quick Commands
\`\`\`bash
npm test              # Run tests
npm run lint          # Check code style
npm run dev           # Start dev server
\`\`\`

## Code Style
- TypeScript strict mode
- No React or Tailwind (use Svelte/vanilla CSS)
- Separate structure from styling
- Minimal boilerplate

## Testing
- Use Vitest
- Test behavior, not implementation
- AAA pattern (Arrange-Act-Assert)
- Edge cases required

## Architecture
- Feature-based organization
- Services for business logic
- Components for UI only
- Stores for state management

## Firebase Conventions
- Use environment variables
- Error handling required
- Security rules for all collections
- Offline persistence enabled
```

**Best Practice**: Keep concise (< 300 lines) and iterate based on what actually helps.

### Tool Permissions

**Configure once, use forever**:

1. **During session**: Select "Always allow"
2. **Via command**: Use `/permissions`
3. **Manual config**: Edit `.claude/settings.json`
4. **CLI flag**: `--allowedTools Read,Write,Edit,Bash`

**Recommended allowlist**:
```json
{
  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash",
    "Grep",
    "Glob",
    "Task"
  ]
}
```

### Environment Tuning

**Custom bash tools**:
```bash
# ~/.bashrc or project .bashrc

# Quick project commands
alias ptest="npm test"
alias plint="npm run lint"
alias pdev="npm run dev"

# Git shortcuts
alias gs="git status"
alias gd="git diff"
alias glog="git log --oneline --graph --all"

# Project-specific functions
check_types() {
  npm run type-check
}

run_firebase_emulators() {
  firebase emulators:start
}
```

### MCP Servers

**Connect via**:
1. **Project config**: `.claude/mcp.json` (committed)
2. **Global config**: `~/.claude/settings.json`
3. **Checked-in config**: `.mcp.json` (legacy)

**Example `.claude/mcp.json`**:
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {
        "ALLOWED_DIRECTORIES": "/home/user/projects"
      }
    }
  }
}
```

### Slash Commands

**Store repeated workflows**:

```
.claude/commands/
├── code-review.md
├── create-feature.md
├── fix-tests.md
└── refactor.md
```

**Example: `code-review.md`**:
```markdown
---
description: Comprehensive code review with security focus
---

Please perform a thorough code review of the recent changes:

1. Read all modified files
2. Check for:
   - Security vulnerabilities
   - Performance issues
   - Code quality problems
   - Test coverage gaps
   - Documentation needs

3. Use the code-reviewer subagent for detailed analysis

4. Provide a summary with:
   - Critical issues (must fix)
   - Suggestions (should consider)
   - Positive observations

$ARGUMENTS
```

**Use with**: `/code-review` or `/code-review --focus=security`

## Common Workflows

### 1. Explore, Plan, Code, Commit

```
┌─────────────────────────────────────────────┐
│ 1. EXPLORE (No Code Yet)                    │
│                                             │
│ "Please read and understand:                │
│  - src/auth/ directory                      │
│  - existing patterns                        │
│  - test structure                           │
│  Don't write code yet."                     │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 2. PLAN (With Thinking)                     │
│                                             │
│ "Think hard about how to implement          │
│  password reset. Create a plan document     │
│  in .plan/password-reset.md"                │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 3. CODE (Implementation)                    │
│                                             │
│ "Implement according to the plan.           │
│  Run tests as you go. Ask before            │
│  making major architecture changes."        │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 4. COMMIT (Documentation)                   │
│                                             │
│ "Commit changes with descriptive message.   │
│  Update CLAUDE.md with any new patterns."   │
└─────────────────────────────────────────────┘
```

**Key**: Separate exploration from implementation to prevent premature code.

### 2. Test-Driven Development

```typescript
// 1. Write tests first
it('should reset password with valid token', async () => {
  const token = generateResetToken(user);
  const newPassword = 'newSecurePassword123';

  const result = await resetPassword(token, newPassword);

  expect(result.success).toBe(true);
  expect(await signIn(user.email, newPassword)).toSucceed();
});

// 2. Confirm tests fail
npm test  // Expected: FAIL

// 3. Commit failing tests
git add tests/
git commit -m "Add password reset tests (currently failing)"

// 4. Implement until tests pass
// ... implementation ...

// 5. Use subagent to verify
"Use the code-reviewer subagent to verify the implementation
isn't overfitting to the tests"
```

**Benefits**:
- Clear requirements
- Implementation guidance
- Prevents over-engineering
- Built-in validation

### 3. Visual Iteration

```
┌────────────────┐
│  Screenshot    │  → "Implement this design"
│  or Mock       │
└────────────────┘
         ↓
┌────────────────┐
│ Implementation │
└────────────────┘
         ↓
┌────────────────┐
│  Screenshot    │  → "Polish: fix spacing, colors, alignment"
│  of Result     │
└────────────────┘
         ↓
┌────────────────┐
│ Final Polish   │
└────────────────┘
         ↓
      Commit
```

**How to provide screenshots**:
- **macOS**: `cmd+ctrl+shift+4` → `ctrl+v` in Claude
- **Drag & drop**: Images directly into chat
- **File path**: "See design at /path/to/mockup.png"

**Iterate 2-3 times** for polish before committing.

### 4. Safe YOLO Mode

**Use case**: Uninterrupted flow for trusted environments

```bash
claude --dangerously-skip-permissions
```

**⚠️ Only in**:
- Containerized environments
- No internet access
- Sandboxed projects
- Development VMs

**Never in**:
- Production environments
- Projects with sensitive data
- Shared systems
- Unsandboxed local machines

### 5. Codebase Q&A

```
"Where is authentication state stored?"
"How does the app handle offline mode?"
"What's the pattern for API error handling?"
"Who wrote the original implementation of the Cart class?"
```

Claude searches agentically - no special prompting needed.

### 6. Git & GitHub Interactions

```bash
# Query history
"Show recent changes to auth.ts and explain why they were made"

# Contextual commits
"Create a commit with a message based on the changes"

# Complex operations
"Rebase the last 5 commits and fix merge conflicts"

# PR management
"Create a PR for this branch with detailed description"
"Read PR #42 comments and address the feedback"

# Issue triage
"Read open issues tagged 'bug' and prioritize them"
```

**Requires**: `gh` CLI installed

### 7. Jupyter Notebooks

**Setup**:
1. Open Claude Code
2. Open `.ipynb` file in VS Code side-by-side
3. Ask Claude to interpret outputs

**Use cases**:
- "Explain what this data visualization shows"
- "Clean up this notebook for presentation"
- "Add markdown documentation between cells"
- "Optimize this data processing code"

## Optimization Techniques

### Be Specific in Instructions

**❌ Vague**:
```
"Add tests for foo.py"
```

**✅ Specific**:
```
"Write new test cases for foo.py covering the edge case where:
- User is logged out
- Database connection fails
- Input contains unicode characters

Avoid mocks - use real implementations where possible.
Use the AAA pattern and descriptive test names."
```

**Result**: Fewer course corrections, faster completion.

### Visual Communication

**Screenshots > Descriptions**:

```
# Instead of:
"The button should be 8px from the top, 16px from left,
blue background (#3B82F6), white text, 12px padding,
rounded corners (4px radius)..."

# Do:
[Paste screenshot]
"Implement this design"
```

**When to use**:
- UI implementation
- Bug reports ("Here's what it looks like")
- Design iteration
- Visual debugging

### File & URL References

**Tab completion**:
```
@src/auth/   # Autocomplete file paths
```

**URL fetching**:
```
"Read https://docs.firebase.com/auth and summarize key concepts"
```

**Allowlist domains** (avoid repeated prompts):
```
/permissions
# Add docs.firebase.com to allowlist
```

### Active Collaboration

**Plan before coding**:
```
"Create a plan before implementing"
"Think hard about the architecture"
"Use 'ultrathink' for this complex problem"
```

**Interrupt freely**:
- **Escape**: Stop current operation
- **Double Escape**: Edit previous prompt
- **New prompt**: Change direction mid-execution

**Pivot when needed**:
```
"Actually, undo that. Let's try a different approach..."
```

### Context Management

**Use `/clear` frequently**:
```
After completing Feature A: /clear
Before starting Feature B: /clear
```

**Why**: Fresh context prevents:
- Token bloat
- Confused state
- Irrelevant information pollution
- Unexpected behavior carryover

**Scratchpads for multi-step tasks**:
```markdown
# .plan/current-task.md

## Feature: User Profile Edit

### Todo
- [ ] Create EditProfile component
- [ ] Add form validation
- [ ] Implement save handler
- [ ] Add loading states
- [ ] Write tests
- [ ] Update CLAUDE.md

### Current Status
Working on form validation...

### Blockers
None
```

**Fix incrementally**:
```
# Instead of:
"Fix all 47 lint errors"

# Do:
"Fix the first lint error in auth.ts"
# Verify
"Fix the next error"
# Verify
# Repeat
```

### Data Input Methods

**Option 1: Copy-paste**:
```
"Here's the user data:
[paste JSON]

Parse and summarize..."
```

**Option 2: Pipe**:
```bash
cat users.json | claude "Analyze this user data"
```

**Option 3: Let Claude fetch**:
```
"Use bash to read data.csv and analyze it"
"Fetch the latest issues from GitHub and summarize"
```

**Option 4: File/URL reference**:
```
"Read data from ./exports/users.csv"
"Fetch and analyze https://api.example.com/stats"
```

## Multi-Claude Workflows

### Parallel Reviews

```
Terminal 1 (Writer):
$ claude
> "Implement payment processing"

Terminal 2 (Reviewer):
$ claude
> "Review the payment processing code in src/payment/"

Terminal 3 (Integrator):
$ claude
> "Read the implementation and review feedback.
   Address concerns and finalize."
```

**Benefits**:
- Independent perspectives
- Fresh context for review
- Faster iteration

### Multiple Checkouts

**Using separate checkouts**:
```bash
# Clone project 3 times
~/project-1/  # Task 1: Auth
~/project-2/  # Task 2: Profile
~/project-3/  # Task 3: Settings

# Run Claude in each
cd ~/project-1 && claude  # Implement auth
cd ~/project-2 && claude  # Implement profile
cd ~/project-3 && claude  # Implement settings
```

**Using git worktrees** (lighter weight):
```bash
# Main worktree
cd ~/project

# Create additional worktrees
git worktree add ../project-auth feature-auth
git worktree add ../project-profile feature-profile
git worktree add ../project-settings feature-settings

# Run Claude in each
cd ../project-auth && claude
cd ../project-profile && claude
cd ../project-settings && claude

# Cleanup when done
git worktree remove ../project-auth
```

**When to use**:
- Independent features
- Parallel development
- Large refactors with multiple phases
- A/B testing different approaches

### Headless Automation

**Non-interactive mode**:
```bash
claude -p "Prompt here"
```

**Streaming JSON output**:
```bash
claude -p "Analyze code" --output-format stream-json
```

**Fan-out pattern** (large migrations):
```bash
# Generate task list
claude -p "List all files that need migration" > tasks.txt

# Process each task
while read file; do
  claude -p "Migrate $file to new API" --json
done < tasks.txt
```

**Pipeline into workflows**:
```bash
# Generate code
claude -p "Create user model" --json > output.json

# Extract generated code
jq -r '.code' output.json > models/user.ts

# Run tests
npm test
```

**Pre-commit hooks**:
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Let Claude check for issues
result=$(claude -p "Review staged changes for issues" --json)

if echo "$result" | jq -e '.hasIssues' > /dev/null; then
  echo "Issues found. Aborting commit."
  echo "$result" | jq -r '.issues[]'
  exit 1
fi
```

## Extended Thinking

**Trigger deeper reasoning**:
```
"think about this problem"              # Basic thinking
"think hard about this"                 # More thorough
"think harder about this"               # Deep analysis
"ultrathink about this"                 # Maximum reasoning
```

**When to use**:
- Complex architecture decisions
- Tricky bugs
- Performance optimization
- Security considerations
- Ambiguous requirements

**Example**:
```
"Think harder about the best way to implement real-time
sync between Firebase and local state. Consider:
- Network reliability
- Conflict resolution
- Offline support
- Performance impact
- Battery usage"
```

## Prompt Engineering

### Progressive Refinement

**Start broad, refine as needed**:

```
# Round 1 (Broad)
"Implement user authentication"

# Round 2 (More specific)
"Use Firebase Auth with email/password. Include error handling
and loading states."

# Round 3 (Very specific)
"Add the following error cases:
- Email already in use
- Weak password
- Network error
- Invalid email format"
```

### Constraints First

**State constraints upfront**:

```
"Implement a shopping cart with these constraints:
- No React (use Svelte)
- No Tailwind (use vanilla CSS)
- Must work offline
- Maximum 200 lines per file
- Test coverage required"
```

### Examples Over Descriptions

```
# Instead of:
"Use a functional programming style with pure functions
and immutability..."

# Show an example:
"Follow this pattern:

```typescript
// Pure function
function addToCart(cart: Cart, item: Item): Cart {
  return {
    ...cart,
    items: [...cart.items, item]
  };
}

// Usage
const newCart = addToCart(cart, item);
```

Use this style throughout."
```

### Reference Existing Code

```
"Implement user profile editing following the same patterns
used in src/auth/signup.ts - same error handling, same form
structure, same validation approach."
```

## Quality Assurance

### Built-in Checks

**Let Claude verify**:

```
"After implementing:
1. Run tests and fix failures
2. Run linter and fix issues
3. Check for console errors
4. Verify all requirements met
5. Review code yourself for issues

Only finish when all checks pass."
```

### Subagent Reviews

```
"When done, use the code-reviewer subagent to review your
work. Address any issues found before finishing."
```

### Checkpoint Before Committing

```
"Before committing:
1. Run full test suite
2. Check test coverage (should be > 80%)
3. Run linter
4. Check for TODOs or FIXMEs
5. Verify requirements from .plan/current-task.md
6. Self-review for obvious issues

Create commit only when everything passes."
```

## Performance Optimization

### Token Efficiency

**Minimize loaded context**:
- Use `/clear` between tasks
- Load only relevant files
- Use subagents for isolated tasks
- Progressive skill loading

**Avoid redundancy**:
- Don't repeat information
- Reference files instead of pasting content
- Use CLAUDE.md for persistent info

**Batch operations**:
```
# Instead of:
"Fix error 1"
"Fix error 2"
"Fix error 3"

# Do:
"Fix these three related errors in auth.ts:
1. ...
2. ...
3. ..."
```

### Speed Optimization

**Parallel operations**:
```
"In parallel:
1. Use one subagent to implement feature
2. Use another to write tests
3. Use third to update documentation"
```

**Skip unnecessary steps**:
```
"Implement authentication. I already researched Firebase Auth,
so skip research phase and implement directly based on
src/docs/firebase-auth-plan.md"
```

**Pre-plan complex tasks**:
```
# Planning session (with ultrathink)
"Create detailed implementation plan"

# Implementation session (fast, follows plan)
"Implement according to plan in .plan/feature.md"
```

## Debugging Techniques

### Systematic Debugging

```
"Debug this issue systematically:

1. Reproduce the bug
2. Add logging to understand state
3. Identify root cause
4. Create a test case that fails
5. Fix the bug
6. Verify test passes
7. Remove debug logging
8. Check for similar issues elsewhere"
```

### Rubber Duck Debugging

```
"Explain step-by-step what this code does and why the
bug might be happening. Walk through it line by line."
```

### Bisect Approach

```
"This feature worked in commit abc123 but broke in def456.
Read both versions, identify the differences, and
determine what caused the regression."
```

## Common Pitfalls

### ❌ Don't

1. **Skip planning for complex tasks**
   - Results in meandering, inefficient implementation

2. **Load entire codebase into context**
   - Wastes tokens, slows performance
   - Use targeted file loading instead

3. **Give vague instructions**
   - "Make it better" → Claude guesses
   - Be specific about what you want

4. **Ignore errors**
   - "Implement auth" (doesn't check if it works)
   - Always verify with tests/manual testing

5. **Commit broken code**
   - Use quality gates (Stop hook or manual checks)

6. **Force one-shot completion**
   - Complex tasks need iteration
   - Use checkpoints and incremental progress

7. **Forget to clear context**
   - Old context pollutes new tasks
   - `/clear` frequently

8. **Over-mock in tests**
   - Mocks hide real issues
   - Use real implementations where feasible

### ✅ Do

1. **Plan before implementing**
   - Especially for complex features
   - Use planner subagent or manual planning

2. **Be specific and constraining**
   - State preferences, anti-patterns, requirements upfront

3. **Verify continuously**
   - Run tests as you go
   - Check output after each step

4. **Use visual communication**
   - Screenshots, mockups, examples

5. **Leverage subagents**
   - For specialized tasks
   - For independent reviews

6. **Clear context regularly**
   - Between tasks
   - Before major changes

7. **Commit incrementally**
   - Small, focused commits
   - Working code only

8. **Document patterns**
   - Update CLAUDE.md with learnings
   - Share successful approaches

## Security Best Practices

### Sensitive Data

**Never commit**:
- API keys
- Passwords
- Tokens
- Credentials
- Private keys

**Always use**:
- Environment variables
- `.env` files (gitignored)
- Secret management services

### Code Review Focus

```
"Review this authentication code with security focus:
- Input validation present?
- SQL injection prevented?
- XSS prevention?
- CSRF tokens used?
- Passwords hashed properly?
- Session management secure?"
```

### Pre-deploy Checklist

```
"Before deploying, verify:
1. No secrets in code
2. All inputs validated
3. All outputs escaped
4. Authentication on all protected routes
5. Authorization checks present
6. HTTPS enforced
7. Security headers configured
8. Dependencies up to date
9. Known vulnerabilities patched"
```

## Integration with Project

### For Experimental Framework

**Use Claude Code CLI in experiments**:
```bash
# Experiment runner
claude -p "Implement auth using minimal .cursorrules" \
  --output-format stream-json \
  > results/run-1.json
```

**Capture metrics**:
```bash
# Extract token usage
jq '.tokenUsage' results/run-1.json

# Extract time
jq '.duration' results/run-1.json

# Extract tool calls
jq '.toolCalls | length' results/run-1.json
```

### For Skill Development

**Test skills**:
```
"Use the firebase-auth skill to implement authentication"
# Verify skill activated
# Measure quality vs without skill
```

### For Subagent Orchestration

**Chain subagents**:
```
"Use planner subagent to create plan,
then implementer subagent to code,
then code-reviewer subagent to review,
then report results"
```

## References

- [Official Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Common Workflows](https://docs.claude.com/en/docs/claude-code/common-workflows)
- [Context Management Guide](https://supatest.ai/blog/claude-context-management-guide)
- [GitButler Hooks Guide](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)

---

**Last Updated**: 2025-11-03
