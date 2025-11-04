# Rule: ReAct Pattern (Reason → Act → Observe)

## The Rule
Before taking any action: reason about it. After action: observe the result. Iterate based on observations.

## Research Backing

### Source: Anthropic Best Practices (.ai-knowledge/research/similar-projects/anthropic-best-practices.md:140-174)

**Principle 5: ReAct and CoT as System Patterns**

> "Build systems enforcing structured reasoning before action and planning before code generation."

**Why It Matters:**
- **ReAct** (Reason → Act → Observe): Prevents impulsive actions
- **CoT** (Chain of Thought): Forces explicit reasoning
- System-level enforcement > hoping agent follows instructions

**Application:**
Make these patterns systemic, not optional:
- Force reasoning before actions
- Require plan before implementation
- Enforce observation after action
- Document thought process

### Source: Agent Design ≠ Prompting (.ai-knowledge/research/similar-projects/anthropic-best-practices.md:15-42)

**Principle 1: Agent Design ≠ Prompting**

> "Structure workflows enabling reasoning, action, reflection, retry, and escalation—not just clever prompts."

**Our Implementation:**
```
✅ Reasoning: Planner agent creates approach
✅ Action: Implementer agent writes code
✅ Reflection: Reviewer agent self-evaluates
✅ Retry: Spin detection triggers alternative approaches
✅ Escalation: After 2-3 attempts, document blocker and ask user
```

## Why This Rule Exists

### Problem: Impulsive Actions Without Reasoning

**Scenario**: Claude is asked to "fix the auth bug"

**Without ReAct:**
```
Claude: "I'll update the Firebase config"
[Makes changes]
Claude: "Done! I updated the config."

Issues:
- No analysis of what's actually broken
- No consideration of alternatives
- No verification that fix worked
- If it fails, no insight into why
```

**With ReAct:**
```
REASON:
- Auth bug could be: config error, token expiration, network issue
- Need to check: logs, config, network status
- Hypothesis: Token refresh not working

ACT:
- Read error logs
- Check token refresh logic
- Identify missing refresh call

OBSERVE:
- Logs show: "Token expired" errors every 60 minutes
- Refresh function exists but not called
- Root cause: Missing event listener

REASON (iterate):
- Need to add token refresh listener
- Should also add error handling
- Test with expired token scenario

ACT:
- Add refresh listener
- Add error handling
- Create test case

OBSERVE:
- Tests pass
- No more "Token expired" errors
- Auth working correctly
```

### Evidence: Improves Quality

From research (.ai-knowledge/research/similar-projects/anthropic-best-practices.md:423-431):

**Experiment 1: ReAct vs Ad-Hoc**
- **Hypothesis**: "Enforced ReAct pattern reduces errors by 40%"
- **Expected**: Fewer errors, clearer debugging

**Why it works:**
- Explicit reasoning catches logical errors before they're implemented
- Observation step catches implementation errors immediately
- Iteration based on observation prevents spinning
- Clear audit trail for debugging

## How to Apply This Rule

### Pattern: Required Steps

```markdown
For every action, follow this sequence:

## REASON (before action)
- What is the current situation?
- What are we trying to achieve?
- What approach should we take?
- What could go wrong?
- Why is this the right approach?

## ACT (execute)
- Take the action
- Log what you're doing
- Be explicit about tool calls

## OBSERVE (after action)
- What happened?
- Did it work as expected?
- Any errors or warnings?
- What did we learn?

## ITERATE (based on observation)
- Continue with next step? OR
- Adjust approach based on observation? OR
- Escalate if blocked?
```

### Template for CLAUDE.md

```markdown
## Workflow: ReAct Pattern

Every task follows Reason → Act → Observe:

### REASON
Before acting:
- Analyze the problem
- Consider alternatives
- Choose best approach
- Document reasoning

### ACT
Execute the plan:
- Take action
- Log all steps
- Be explicit

### OBSERVE
After acting:
- Check results
- Verify success
- Document learnings

### ITERATE
Based on observations:
- Continue if successful
- Adjust if issues found
- Escalate if blocked (after 2-3 attempts)
```

### Integration with Planner Agent

```markdown
# In .claude/agents/planner.agent

You are a planning agent. Follow ReAct pattern:

## REASON Phase (Your Primary Role)
Analyze task:
1. What is being asked?
2. What are the requirements?
3. What are potential approaches?
4. What are the risks?
5. What's the best approach?

Output: Detailed plan with reasoning

## ACT Phase (Delegated)
You don't execute. Pass plan to implementer agent.

## OBSERVE Phase (Your Role)
When implementer reports results:
1. Did plan work as expected?
2. Any unexpected outcomes?
3. What adjustments needed?

Output: Refined plan or approval to proceed
```

### Integration with Implementer Agent

```markdown
# In .claude/agents/implementer.agent

You are an implementation agent. Follow ReAct pattern:

## REASON Phase (Check Plan)
Before implementing:
1. Read plan from planner
2. Verify you understand each step
3. Identify potential issues
4. Ask for clarification if needed

## ACT Phase (Your Primary Role)
Execute plan:
1. Implement each step
2. Log every action taken
3. Create clear commits
4. Document as you go

## OBSERVE Phase (Your Primary Role)
After each significant action:
1. Run tests
2. Check for errors
3. Verify expected behavior
4. Document what happened

Output: Results with observations

## ITERATE Phase
Based on observations:
- Continue to next step (if success)
- Retry with adjustments (if minor issues)
- Escalate to planner (if major issues)
```

## ReAct in Practice: Examples

### Example 1: Implementing a Feature

**Task**: Add user profile page

**REASON:**
```markdown
Analysis:
- Need route /profile
- Need to fetch user data from Firebase
- Need to display: name, email, avatar
- Need edit capability

Approach:
1. Create profile route
2. Create ProfilePage component
3. Add Firebase query for user data
4. Add edit form
5. Add update mutation

Risks:
- User might not be logged in (handle)
- Data might not exist (handle)
- Update might fail (handle)

Decision: Start with read-only view, add edit in step 2
```

**ACT:**
```markdown
Step 1: Create route
- Added /profile to routes.ts
- Created src/routes/profile/+page.svelte

Step 2: Fetch user data
- Added getUserProfile() query
- Handles loading/error states

Step 3: Display data
- Shows name, email, avatar
- Graceful degradation if data missing
```

**OBSERVE:**
```markdown
Results:
✅ Route accessible at /profile
✅ Data loads correctly
✅ UI displays properly
⚠️  Slow load time (2 seconds)

Learnings:
- Query works but needs optimization
- Should add loading skeleton
- Avatar needs lazy loading
```

**ITERATE:**
```markdown
Adjustments needed:
1. Add loading skeleton
2. Optimize query (add index)
3. Lazy load avatar

Continue? YES, with adjustments
```

### Example 2: Debugging a Bug

**Task**: Fix "undefined is not a function" error

**REASON:**
```markdown
Analysis:
- Error: "undefined is not a function"
- Location: auth.ts:45
- When: After user login

Hypothesis:
- Likely calling method on undefined object
- Could be: Firebase not initialized, user object missing, or method typo

Approach:
1. Check error stack trace
2. Read auth.ts:45
3. Check Firebase initialization
4. Add defensive checks
```

**ACT:**
```markdown
Investigation:
- Read error logs: Full stack trace
- Read auth.ts:45: `user.getIdToken()`
- Issue: `user` is undefined

Root cause:
- Firebase returns null for logged-out users
- Code assumes user exists
```

**OBSERVE:**
```markdown
Findings:
- Code: `const token = user.getIdToken();`
- Problem: Missing null check
- Occurs: When user logged out but page still open

Fix needed:
- Add null check before calling getIdToken()
- Handle logged-out state gracefully
```

**ITERATE:**
```markdown
Implementation:
- Added: `if (!user) return null;`
- Added: UI state for logged-out users
- Added: Test case for logged-out scenario

Verify:
✅ Error gone
✅ Logged-out users see login prompt
✅ Tests pass
```

## What NOT to Do

### Anti-Pattern 1: Skipping REASON

```markdown
Bad:
User: "Fix the auth bug"
Claude: [Immediately starts editing code]

Why bad:
- No analysis of what's actually broken
- Might fix wrong thing
- No consideration of side effects
```

### Anti-Pattern 2: Skipping OBSERVE

```markdown
Bad:
Claude: [Makes changes]
Claude: "Done!"

Why bad:
- Didn't verify it worked
- Might have broken something
- No learning captured
```

### Anti-Pattern 3: Reasoning Without Acting

```markdown
Bad:
Claude: "I think the issue is X. We should do Y."
[No action taken]

Why bad:
- Analysis paralysis
- Reasoning without execution doesn't solve problems
```

### Anti-Pattern 4: Acting Without Observing

```markdown
Bad:
Claude: [Makes 5 changes in a row without testing]

Why bad:
- If something breaks, hard to know which change caused it
- No feedback loop
- Can't iterate effectively
```

## Validation Method

### Experiment: ReAct vs Ad-Hoc

**Hypothesis**: "Enforced ReAct pattern reduces errors by 40%"

**Design:**
- Control: No structured reasoning required
- Treatment: Enforce Reason → Act → Observe cycle
- Tasks: 20 representative scenarios
- Measure: Error rate, retry count, success rate, time

**Success Criteria:**
- ✅ ≥30% error reduction
- ✅ ≥20% fewer retries
- ✅ Equal or better success rate
- ⚠️  May take longer initially (but faster overall with fewer retries)

### Measurement

Track in `.plan/current-task.md`:

```markdown
## ReAct Log

### REASON (2 minutes)
[Document reasoning]

### ACT (10 minutes)
[Document actions]

### OBSERVE (2 minutes)
[Document observations]

### ITERATE (5 minutes)
[Document adjustments]

Total time: 19 minutes
Errors: 0
Retries: 0
Success: ✅
```

Compare to tasks without ReAct structure:

```markdown
## Ad-Hoc Approach

Implemented directly: 12 minutes
Errors found during review: 3
Time to fix errors: 15 minutes
Total time: 27 minutes
Success: ✅ (after 3 retries)
```

**Result**: ReAct took 19min vs 27min ad-hoc (30% faster, including reasoning overhead)

## Integration with Other Rules

### Synergy with Planning Essential
- ReAct: Reason before acting
- Planning: Create explicit plan before coding
- Combined: Plan (reason) → Implement (act) → Review (observe)

### Synergy with Autonomy Boundaries
- ReAct: Observe results, iterate based on observation
- Boundaries: Max 3 retries, then escalate
- Combined: ReAct iterations are bounded, preventing infinite loops

### Synergy with Validation Required
- ReAct: Observe whether action worked
- Validation: Verify claims with experiments
- Combined: Observations are empirical, not assumptions

## Examples from Research

### Anthropic's Workflow

From .ai-knowledge/research/similar-projects/anthropic-best-practices.md:162-174:

```markdown
Enhanced approach:
1. Force explicit reasoning in plan.md
2. Log every action taken
3. Require observation/reflection after each action
4. Iterate based on observations
```

### Our Implementation

```markdown
Current approach:
1. Planner creates plan (implicit CoT)
2. Implementer writes code
3. Reviewer evaluates (implicit ReAct observation)

Enhanced approach:
1. Force explicit reasoning in .plan/reasoning.md
2. Log every action in .plan/actions.log
3. Require observation in .plan/observations.md after each action
4. Iterate based on observations documented in .plan/iterations.md
```

## Continuous Improvement

### After Each Task

Review ReAct execution:
1. Was reasoning clear and complete?
2. Were actions logged explicitly?
3. Were observations documented?
4. Did iterations improve approach?

If any step was weak, improve template or enforcement.

### Monthly Audit

```json
// .ai-knowledge/metrics/react-usage.json
{
  "period": "2025-11",
  "tasks": 47,
  "reactCompliance": {
    "reason": "100%",
    "act": "100%",
    "observe": "87%",
    "iterate": "76%"
  },
  "insights": {
    "weakness": "Observation step sometimes skipped",
    "action": "Make observation step more explicit in templates"
  }
}
```

## References

1. **Anthropic Best Practices**: `.ai-knowledge/research/similar-projects/anthropic-best-practices.md` (lines 140-174)
2. **Master Synthesis**: `.ai-knowledge/research/MASTER-SYNTHESIS.md`
3. **System Over Prompts**: `.ai-knowledge/research/similar-projects/anthropic-best-practices.md` (Principle 1)

## Status
- **Research-Backed**: ✅ Official Anthropic guidance
- **Validated**: ⚠️ Needs experimental validation (40% error reduction hypothesis)
- **Implementation**: Ready (templates created)
- **Priority**: CRITICAL (Foundation for all workflows)
