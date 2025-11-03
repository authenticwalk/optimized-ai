# Claude Triggers Quick Start Guide

**Get up and running with Claude Code triggers in 10 minutes.**

## What Are Triggers?

Claude Code provides three main trigger mechanisms:

1. **Hooks** - Automated scripts at lifecycle events (8 types)
2. **Subagents** - Specialized AI assistants with isolated context
3. **Skills** - On-demand knowledge injection (3-level progressive loading)

## 5-Minute Setup

### 1. Create Hook Directory

```bash
mkdir -p ~/.claude/hooks
mkdir -p .claude/agents
mkdir -p .claude/skills
```

### 2. Your First Hook (Desktop Notification)

Create `~/.claude/hooks/stop_notify.sh`:

```bash
#!/bin/bash
# Notify when Claude finishes

osascript -e 'display notification "Task complete!" with title "Claude Finished"'
exit 0
```

Make it executable:
```bash
chmod +x ~/.claude/hooks/stop_notify.sh
```

Configure in `~/.claude/settings.json`:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/Users/you/.claude/hooks/stop_notify.sh"
          }
        ]
      }
    ]
  }
}
```

### 3. Your First Subagent

Create `.claude/agents/code-reviewer.md`:

```yaml
---
name: code-reviewer
description: Use PROACTIVELY to review code before creating PRs
tools: Read,Glob,Grep
model: sonnet
---

# Code Reviewer

Review code for:
- Quality issues
- Security vulnerabilities
- Performance problems
- Test coverage gaps

Provide specific feedback with file:line references.
```

### 4. Your First Skill

Create `.claude/skills/testing.md`:

```yaml
---
name: testing
description: Use when writing tests or ensuring test coverage
tools: Read,Write,Edit,Bash
---

# Testing Skill

## Test Structure
Use AAA pattern:
- Arrange: Set up test data
- Act: Execute function
- Assert: Verify results

## Coverage
Ensure:
- Happy path tests
- Edge cases
- Error handling
```

### 5. Test It

```bash
# Start Claude Code
claude

# Test the subagent
> "Use the code-reviewer subagent to review src/"

# Test the skill (happens automatically if you ask for tests)
> "Add tests for auth.ts"

# Hook will notify when you finish
> "I'm done for now"
# Should see desktop notification
```

## Common Patterns

### Pattern 1: Quality Gate Hook

**Block completion until tests pass**

Create `~/.claude/hooks/stop_quality_gate.py`:

```python
#!/usr/bin/env python3
import json
import sys
import subprocess

# Run tests
result = subprocess.run(['npm', 'test'], capture_output=True)

if result.returncode != 0:
    output = {
        "decision": "block",
        "reason": "Tests failing. Please fix before completing."
    }
    print(json.dumps(output))
    sys.exit(0)

# Tests passed
print(json.dumps({"continue": True}))
```

Configure:
```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "python3 /path/to/stop_quality_gate.py",
        "timeout": 120
      }]
    }]
  }
}
```

### Pattern 2: Security Validation Hook

**Block dangerous commands**

Create `~/.claude/hooks/pre_tool_use_security.py`:

```python
#!/usr/bin/env python3
import json
import sys
import re

hook_input = json.loads(sys.stdin.read())
tool_name = hook_input.get('tool_name')

if tool_name == 'Bash':
    command = hook_input.get('tool_input', {}).get('command', '')

    # Dangerous patterns
    if re.search(r'rm\s+-rf\s+/', command):
        output = {
            "decision": "deny",
            "reason": "Blocked: recursive delete from root"
        }
        print(json.dumps(output))
        sys.exit(0)

print(json.dumps({"continue": True}))
```

Configure:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "python3 /path/to/pre_tool_use_security.py"
      }]
    }]
  }
}
```

### Pattern 3: Auto-Format Hook

**Format files after editing**

Create `~/.claude/hooks/post_tool_use_format.sh`:

```bash
#!/bin/bash
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

if [[ "$tool_name" =~ ^(Edit|Write)$ ]]; then
    if [[ "$file_path" =~ \\.(ts|js)$ ]]; then
        npx prettier --write "$file_path" 2>&1
    fi
fi

exit 0
```

Configure:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "/path/to/post_tool_use_format.sh"
      }]
    }]
  }
}
```

### Pattern 4: Context Injection Hook

**Add project context on session start**

Create `~/.claude/hooks/session_start.py`:

```python
#!/usr/bin/env python3
import json
import subprocess

# Get git status
git_status = subprocess.check_output(['git', 'status', '--short']).decode()

context = f"""
Project: My App
Tech Stack: TypeScript, Firebase, Svelte
Recent changes:
{git_status}

Preferences:
- No React/Tailwind
- Use Vitest for tests
- Minimal boilerplate
"""

output = {
    "hookSpecificOutput": {
        "additionalContext": context
    }
}

print(json.dumps(output))
```

Configure:
```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "python3 /path/to/session_start.py"
      }]
    }]
  }
}
```

## Quick Reference

### Hook Events

| Event | When | Use For |
|-------|------|---------|
| **UserPromptSubmit** | Before processing prompt | Context injection, validation |
| **PreToolUse** | Before tool execution | Security blocking, approval |
| **PostToolUse** | After tool completes | Testing, formatting, logging |
| **Stop** | When finishing | Quality gates, notifications |
| **SubagentStop** | When subagent finishes | Subagent validation |
| **SessionStart** | At session start | Initialization, context loading |
| **SessionEnd** | At session end | Cleanup, archiving |
| **PreCompact** | Before context compression | Backup transcripts |

### Hook Control

**Exit Codes**:
- `0` - Success, continue
- `2` - Block operation, show stderr to Claude

**JSON Response**:
```json
{
  "decision": "block|approve|deny|ask",
  "reason": "Why",
  "hookSpecificOutput": {
    "additionalContext": "Extra context"
  }
}
```

### Subagent Invocation

**Automatic**:
```
Description: "Use PROACTIVELY when..."
```

**Explicit**:
```
"Use the code-reviewer subagent to review my changes"
```

**Chained**:
```
"Use planner subagent to create plan, then implementer subagent to code"
```

### Skills Activation

**Automatic** (LLM-based routing):
```
User: "Implement Firebase authentication"
→ Claude activates firebase-auth skill automatically
```

**Manual mention**:
```
"Use the firebase-auth skill to implement login"
```

## Common Use Cases

### Use Case 1: Spin Detection

**Problem**: Claude edits the same file repeatedly without progress

**Solution**: PreToolUse hook that tracks edit history

```python
# Check if file edited 3+ times in 2 minutes
if recent_edit_count >= 3:
    return {"decision": "deny", "reason": "Spin detected"}
```

### Use Case 2: Always-Passing Tests

**Problem**: Want to ensure tests always pass before finishing

**Solution**: Stop hook that runs test suite

```python
if subprocess.run(['npm', 'test']).returncode != 0:
    return {"decision": "block", "reason": "Tests failing"}
```

### Use Case 3: Code Review Requirement

**Problem**: Want independent review before PRs

**Solution**: Subagent for code review

```yaml
name: code-reviewer
description: Use PROACTIVELY before creating PRs
```

### Use Case 4: Domain-Specific Guidance

**Problem**: Team needs Firebase best practices consistently

**Solution**: firebase-auth skill with patterns

```yaml
name: firebase-auth
description: Use when implementing Firebase Authentication
# Include patterns, pitfalls, security notes
```

### Use Case 5: Auto-Documentation

**Problem**: Want changes documented automatically

**Solution**: PostToolUse hook that updates docs

```python
if tool_name == "Write" and "src/" in file_path:
    # Extract function signatures, update docs
```

## Debugging

### Check Hook Execution

```bash
claude --debug
```

Shows:
- Which hooks matched
- Commands executed
- Output from hooks
- Timing

### Test Hook Manually

```bash
echo '{"tool_name":"Edit"}' | python3 your_hook.py
```

### Check Subagent Activation

Look for:
```
[Using subagent: code-reviewer]
```

### Check Skill Loading

Look for:
```
[Loaded skill: firebase-auth]
```

## Next Steps

### Expand Hooks
1. Add more security patterns to PreToolUse
2. Add linting to PostToolUse
3. Add metrics collection
4. Add error recovery patterns

### Create Subagents
1. Planner for complex tasks
2. Tester for test creation
3. Security auditor for sensitive code
4. Documentation writer

### Build Skills
1. Your tech stack (Firebase, Supabase, etc.)
2. Testing frameworks
3. Design patterns
4. Security practices

### Study Examples
- [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- [Claude Command Suite](https://github.com/qdhenry/Claude-Command-Suite)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)

## Integration with Optimized AI Project

### For Phase 0 (Experimental Framework)

**Use hooks to collect metrics**:
```python
# post_tool_use_metrics.py
# Log: tool_name, time, tokens, file_path
# Save to: experiments/metrics/session_metrics.jsonl
```

**Use subagents in experiments**:
```bash
# Control: No subagents
# Treatment: With planner + implementer + reviewer subagents
# Measure: Quality, time, tokens
```

**Test skills effectiveness**:
```bash
# Control: No skills
# Treatment: firebase-auth skill active
# Measure: Code quality, errors, time
```

### For Phase 1 (Minimal Core)

**Use hooks for validation**:
- PreToolUse: Enforce .cursorrules compliance
- PostToolUse: Auto-test after changes
- Stop: Ensure minimal .cursorrules followed

**Use subagents for separation**:
- Planner: Strategic decisions
- Implementer: Code writing
- Reviewer: Quality checks

### For Phase 2+ (Skills & Subagents)

**Build skill library**:
- firebase-auth
- supabase-rls
- testing-patterns
- refactoring-patterns

**Create agent coordination**:
- Orchestrator pattern
- Sequential pipeline
- Parallel processing

## Resources

- **Full Docs**: See other files in this directory
- **Examples**: [repositories.md](./repositories.md)
- **Best Practices**: [best-practices.md](./best-practices.md)
- **Hooks Reference**: [hooks-reference.md](./hooks-reference.md)
- **Subagents**: [subagent-patterns.md](./subagent-patterns.md)
- **Skills**: [skills-architecture.md](./skills-architecture.md)

---

**Ready to implement?** Start with one hook, one subagent, and one skill. Test thoroughly. Then expand based on what works for your workflow.
