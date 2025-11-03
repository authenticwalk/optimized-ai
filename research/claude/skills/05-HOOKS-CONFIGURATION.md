# Hooks Configuration & Automation

**Status**: ✅ Validated by official documentation and community implementations

---

## Table of Contents
1. [What Are Hooks?](#what-are-hooks)
2. [Hook Types & Events](#hook-types--events)
3. [Configuration Format](#configuration-format)
4. [Practical Examples](#practical-examples)
5. [Best Practices](#best-practices)
6. [Humans-in-the-Loop Pattern](#humans-in-the-loop-pattern)
7. [Safety & Validation](#safety--validation)

---

## What Are Hooks?

### Definition
**Hooks are shell commands that execute in response to Claude Code events like tool calls, agent completion, or session events.**

### Purpose
- ✅ Automate workflows
- ✅ Validate operations
- ✅ Suggest next steps
- ✅ Integrate with external tools
- ✅ Log and track activities
- ✅ Enforce security policies

### Key Principle
**Hooks suggest, humans approve**
```
❌ Hooks chain agents automatically (runaway risk)
✅ Hooks print suggested commands, humans execute
```

---

## Hook Types & Events

### Event Types

**1. PreToolUse** (before tool execution)
- Triggered: Before Claude executes a tool
- Use for: Validation, modification, security checks
- Since: v2.0.10 can modify tool inputs

**2. PostToolUse** (after tool completion)
- Triggered: After tool completes successfully
- Use for: Logging, notifications, cleanup

**3. UserPromptSubmit** (when user submits prompt)
- Triggered: When user sends a message
- Use for: Context setup, validation

**4. SubagentStop** (when subagent completes)
- Triggered: Subagent finishes and returns
- Use for: Suggesting next steps, status updates

**5. Stop** (when Claude finishes responding)
- Triggered: Main agent completes response
- Use for: Session logging, notifications

**6. SessionEnd** (when session terminates)
- Triggered: Claude Code exits
- Use for: Cleanup, final reports

---

## Configuration Format

### Location

**Project-level** (recommended)
```
.claude/settings.json
```

**User-level**
```
~/.claude/settings.json
```

**Local overrides**
```
.claude/settings.local.json
```

### Basic Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

### Matcher Patterns

**Exact match**
```json
{
  "matcher": "Write"  // Matches only Write tool
}
```

**Regex pattern**
```json
{
  "matcher": "Edit|Write"  // Matches Edit OR Write
}
```

**MCP tools**
```json
{
  "matcher": "mcp__github__.*"  // All GitHub MCP tools
}
```

**Pattern format**: `mcp__<server>__<tool>`
```
mcp__memory__create_entities
mcp__filesystem__read_file
mcp__github__create_pr
```

---

## Practical Examples

### Example 1: Desktop Notifications

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Task completed'"
          }
        ]
      }
    ]
  }
}
```

### Example 2: Git Commit Validation

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'if [[ \"$CLAUDE_TOOL_INPUT\" =~ \"git commit\" ]]; then echo \"⚠️  Review commit message carefully\"; fi'"
          }
        ]
      }
    ]
  }
}
```

### Example 3: Security Validation

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/validate-security.py"
          }
        ]
      }
    ]
  }
}
```

```python
# .claude/hooks/validate-security.py
import os
import sys
import re

# Get tool input from environment
tool_input = os.environ.get('CLAUDE_TOOL_INPUT', '')

# Dangerous patterns
dangerous_patterns = [
    r'rm\s+-rf\s+/',
    r'sudo\s+rm',
    r'chmod\s+777',
    r'>(dev/null|/)',
]

for pattern in dangerous_patterns:
    if re.search(pattern, tool_input):
        print(f"🛑 BLOCKED: Dangerous pattern detected: {pattern}")
        sys.exit(1)

print("✅ Security check passed")
```

### Example 4: Subagent Workflow Suggestion

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "pm-spec",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/suggest-next-step.sh pm-spec"
          }
        ]
      },
      {
        "matcher": "architect-review",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/suggest-next-step.sh architect-review"
          }
        ]
      }
    ]
  }
}
```

```bash
# .claude/hooks/suggest-next-step.sh
#!/bin/bash

AGENT=$1

case $AGENT in
  "pm-spec")
    echo ""
    echo "✅ PM-Spec complete"
    echo "📋 Spec written to: docs/specs/"
    echo ""
    echo "Next step (copy-paste):"
    echo "  claude --agents architect-review 'Review spec and create ADR'"
    ;;
  "architect-review")
    echo ""
    echo "✅ Architecture review complete"
    echo "📋 ADR written to: docs/decisions/"
    echo ""
    echo "Next step (copy-paste):"
    echo "  claude --agents implementer-tester 'Implement per ADR'"
    ;;
  "implementer-tester")
    echo ""
    echo "✅ Implementation complete"
    echo "📋 Code and tests written"
    echo ""
    echo "Next step (copy-paste):"
    echo "  claude --agents code-reviewer 'Review implementation'"
    ;;
esac
```

### Example 5: Queue Status Management

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "pm-spec",
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/hooks/update-queue-status.js READY_FOR_ARCH"
          }
        ]
      }
    ]
  }
}
```

```javascript
// .claude/hooks/update-queue-status.js
const fs = require('fs');

const newStatus = process.argv[2];
const queueFile = 'enhancements/_queue.json';

const queue = JSON.parse(fs.readFileSync(queueFile, 'utf8'));

// Update current item status
const currentItem = queue.items.find(item => item.status === 'IN_PROGRESS');
if (currentItem) {
  currentItem.status = newStatus;
  currentItem.updated_at = new Date().toISOString();
}

fs.writeFileSync(queueFile, JSON.stringify(queue, null, 2));

console.log(`✅ Queue updated: ${currentItem.id} → ${newStatus}`);
```

### Example 6: Logging & Audit Trail

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "mcp__.*__write.*",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/log-write-operation.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
# .claude/hooks/log-write-operation.sh
#!/bin/bash

timestamp=$(date -Iseconds)
tool=$CLAUDE_TOOL_NAME
input=$CLAUDE_TOOL_INPUT

echo "[$timestamp] $tool - $input" >> .claude/hooks.log
```

### Example 7: Test Coverage Validation

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'if [[ \"$CLAUDE_TOOL_INPUT\" =~ \"npm test\" ]]; then .claude/hooks/check-coverage.sh; fi'"
          }
        ]
      }
    ]
  }
}
```

```bash
# .claude/hooks/check-coverage.sh
#!/bin/bash

# Extract coverage from last test run
coverage=$(grep "All files" coverage/lcov-report/index.html | grep -oP '\d+\.\d+(?=%)')

if (( $(echo "$coverage < 90" | bc -l) )); then
  echo "⚠️  Coverage is $coverage% (target: 90%)"
else
  echo "✅ Coverage: $coverage%"
fi
```

---

## Best Practices

### 1. Output to STDOUT

```bash
# ✅ Good: Output visible to Claude
echo "✅ Hook executed successfully"

# ❌ Bad: Output goes nowhere
echo "Message" > /dev/tty
```

### 2. Register Both SubagentStop and Stop

```json
{
  "hooks": {
    "SubagentStop": [{...}],  // For subagent completion
    "Stop": [{...}]           // Fallback if not subagent
  }
}
```

### 3. Make Hooks Idempotent

```javascript
// ✅ Good: Check before modifying
if (!queue.items.find(item => item.id === newItem.id)) {
  queue.items.push(newItem);
}

// ❌ Bad: Always append
queue.items.push(newItem);  // Duplicates!
```

### 4. Version Control Hook Scripts

```
.claude/
├── hooks/
│   ├── validate-security.py
│   ├── suggest-next-step.sh
│   ├── update-queue-status.js
│   └── check-coverage.sh
└── settings.json
```

**Commit hooks to git**
- Team shares automation
- Auditable changes
- Versioned alongside code

### 5. Treat Hooks as Production Code

```
✅ Test hooks independently
✅ Handle errors gracefully
✅ Validate JSON in config
✅ Document hook purpose
✅ Code review hook changes
```

### 6. Use Environment Variables

**Available variables**:
```bash
$CLAUDE_TOOL_NAME      # Name of tool being called
$CLAUDE_TOOL_INPUT     # Input to the tool
$CLAUDE_AGENT_NAME     # Current agent name (if subagent)
```

**Example**:
```bash
#!/bin/bash

if [[ $CLAUDE_AGENT_NAME == "pm-spec" ]]; then
  echo "PM agent completed"
elif [[ $CLAUDE_AGENT_NAME == "implementer-tester" ]]; then
  echo "Implementation agent completed"
fi
```

### 7. Graceful Failures

```bash
#!/bin/bash

# Don't block on non-critical failures
coverage_check || echo "⚠️  Coverage check failed (non-blocking)"

# Exit with success even if check fails
exit 0
```

---

## Humans-in-the-Loop Pattern

### The Problem

**Runaway agent chains**:
```
Agent A completes
  → Hook automatically calls Agent B
    → Agent B completes
      → Hook automatically calls Agent C
        → Agent C completes
          → Hook automatically calls Agent D
            → [No human oversight, potential for errors]
```

### The Solution

**Hooks suggest, humans approve**:
```
Agent A completes
  → Hook prints: "Next: claude --agents agent-b ..."
    → HUMAN reviews and decides
      → HUMAN copy-pastes if approved
        → Agent B executes
          → [Human in loop, controlled progression]
```

### Implementation

```bash
# ❌ Bad: Automatic chaining
claude --agents architect-review "..."  # Hook executes automatically

# ✅ Good: Suggest next step
echo ""
echo "Next step (review and copy-paste if approved):"
echo "  claude --agents architect-review \"Review spec: $SLUG\""
echo ""
```

### Benefits

1. **Human review gate**: Quick check before proceeding
2. **Error prevention**: Catch issues before propagating
3. **Cost control**: No runaway token usage
4. **Flexibility**: Humans can modify suggested command
5. **Audit trail**: Humans consciously approve each step

---

## Safety & Validation

### Pattern 1: Whitelist Commands

```python
# validate-command.py
import os
import sys

allowed_commands = [
    'npm test',
    'npm run build',
    'git status',
    'git diff',
]

command = os.environ.get('CLAUDE_TOOL_INPUT', '')

if not any(cmd in command for cmd in allowed_commands):
    print(f"🛑 BLOCKED: Command not in whitelist")
    sys.exit(1)

print("✅ Command approved")
```

### Pattern 2: Dry-Run Mode

```bash
#!/bin/bash

DRY_RUN=${DRY_RUN:-false}

if [[ $DRY_RUN == "true" ]]; then
  echo "[DRY RUN] Would execute: $CLAUDE_TOOL_INPUT"
  exit 0
fi

# Actual execution
```

### Pattern 3: Confirmation Prompts

```bash
#!/bin/bash

if [[ $CLAUDE_TOOL_INPUT =~ "rm" ]]; then
  echo "⚠️  Deletion detected:"
  echo "  $CLAUDE_TOOL_INPUT"
  echo ""
  echo "Proceed? (yes/no)"
  # In hook context, this blocks execution for review
  exit 1  # Block by default
fi
```

### Pattern 4: Rate Limiting

```bash
#!/bin/bash

RATE_LIMIT_FILE=".claude/rate-limit"
CURRENT_TIME=$(date +%s)

if [[ -f $RATE_LIMIT_FILE ]]; then
  LAST_EXEC=$(cat $RATE_LIMIT_FILE)
  DIFF=$((CURRENT_TIME - LAST_EXEC))

  if [[ $DIFF -lt 60 ]]; then
    echo "⚠️  Rate limit: Wait $((60 - DIFF))s"
    exit 1
  fi
fi

echo $CURRENT_TIME > $RATE_LIMIT_FILE
```

---

## Key Takeaways for Optimized AI

### 1. Use Hooks for Workflow Suggestions

```bash
# After each subagent completes:
SubagentStop → suggest next step
Human → reviews and approves
Human → copy-pastes command
Next agent → executes
```

### 2. Maintain Queue State

```bash
pm-spec completes → update queue to READY_FOR_ARCH
architect-review completes → update to READY_FOR_BUILD
implementer completes → update to READY_FOR_REVIEW
```

### 3. Enforce Quality Gates

```bash
PostToolUse (test) → check coverage >90%
PostToolUse (build) → verify no errors
PreToolUse (commit) → lint checks pass
```

### 4. Log Everything

```bash
All tool usage → append to .claude/hooks.log
Include: timestamp, agent, tool, input
Enables: debugging, auditing, metrics
```

### 5. Security Validation

```bash
PreToolUse → validate dangerous patterns
Block: rm -rf, sudo rm, chmod 777
Allow: safe operations only
```

### 6. Never Auto-Chain Agents

```bash
✅ Print suggested command
❌ Execute command automatically
Reason: Prevents runaway chains, maintains control
```

---

## Complete Example: Multi-Stage Pipeline

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "pm-spec",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/workflow.sh pm-spec-complete"
          }
        ]
      },
      {
        "matcher": "architect-review",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/workflow.sh architect-complete"
          }
        ]
      },
      {
        "matcher": "implementer-tester",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/workflow.sh implementer-complete"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/log-tool-use.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
# .claude/hooks/workflow.sh
#!/bin/bash

EVENT=$1
QUEUE_FILE="enhancements/_queue.json"
LOG_FILE=".claude/hooks.log"

log_event() {
  echo "[$(date -Iseconds)] $1" >> $LOG_FILE
}

case $EVENT in
  "pm-spec-complete")
    log_event "PM-Spec completed"
    node .claude/hooks/update-queue.js READY_FOR_ARCH

    echo ""
    echo "✅ PM-Spec Complete"
    echo "📋 Spec: docs/specs/"
    echo ""
    echo "Next (copy-paste if approved):"
    echo "  claude --agents architect-review 'Review spec and create ADR'"
    echo ""
    ;;

  "architect-complete")
    log_event "Architect-Review completed"
    node .claude/hooks/update-queue.js READY_FOR_BUILD

    echo ""
    echo "✅ Architecture Review Complete"
    echo "📋 ADR: docs/decisions/"
    echo ""
    echo "Next (copy-paste if approved):"
    echo "  claude --agents implementer-tester 'Implement per ADR'"
    echo ""
    ;;

  "implementer-complete")
    log_event "Implementer-Tester completed"
    node .claude/hooks/update-queue.js READY_FOR_REVIEW

    # Run tests
    npm test > /dev/null 2>&1
    if [[ $? -eq 0 ]]; then
      echo "✅ Tests passing"
    else
      echo "⚠️  Tests failing - review before proceeding"
    fi

    # Check coverage
    coverage=$(.claude/hooks/check-coverage.sh)

    echo ""
    echo "✅ Implementation Complete"
    echo "📋 Code: src/"
    echo "📋 Tests: $coverage coverage"
    echo ""
    echo "Next (copy-paste if approved):"
    echo "  claude --agents code-reviewer 'Review implementation'"
    echo ""
    ;;
esac
```

---

## Resources

### Official
- [Claude Code Hooks Reference](https://docs.claude.com/en/docs/claude-code/hooks)

### Community
- [claude-code-hooks-mastery](https://github.com/disler/claude-code-hooks-mastery)
- [Automate AI Workflows with Hooks](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)

---

**Next**: Read [06-EXAMPLES-SNIPPETS.md](06-EXAMPLES-SNIPPETS.md) for practical code examples
