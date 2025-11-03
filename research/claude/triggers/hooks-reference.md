# Claude Code Hooks: Complete Reference

## Overview

Claude Code hooks are automated scripts that execute at specific points in Claude's workflow, enabling custom validation, approval flows, context injection, and deterministic control over AI behavior.

**Key Insight**: Hooks provide deterministic control through structured rules rather than relying on LLM reasoning. This is critical for reliability.

## Hook Lifecycle Events

### 1. UserPromptSubmit

**When**: Fires when user submits a prompt, before Claude processes it

**Use Cases**:
- Inject additional context based on prompt content
- Validate prompts for dangerous patterns
- Block certain types of requests
- Log all prompts for audit trails
- Add security guardrails

**Control Options**:
```json
{
  "decision": "block",  // Prevents prompt processing
  "reason": "Explanation shown to user",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "Extra context Claude will see"
  }
}
```

**Example - Context Injection**:
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/inject_context.py"
          }
        ]
      }
    ]
  }
}
```

```python
# inject_context.py
import json
import sys
import subprocess

def get_git_status():
    return subprocess.check_output(['git', 'status', '--short']).decode()

def get_recent_errors():
    # Read from project error log
    try:
        with open('.logs/recent_errors.log', 'r') as f:
            return f.read()
    except:
        return ""

hook_input = json.loads(sys.stdin.read())
prompt = hook_input.get('user_prompt', '')

context = f"""
Git Status:
{get_git_status()}

Recent Errors:
{get_recent_errors()}
"""

output = {
    "hookSpecificOutput": {
        "hookEventName": "UserPromptSubmit",
        "additionalContext": context
    }
}

print(json.dumps(output))
```

**Example - Prompt Validation**:
```python
# validate_prompt.py
import json
import sys
import re

DANGEROUS_PATTERNS = [
    r'rm\s+-rf\s+/',
    r'drop\s+database',
    r'delete\s+from\s+users',
    r'chmod\s+777',
    r'sudo\s+rm'
]

hook_input = json.loads(sys.stdin.read())
prompt = hook_input.get('user_prompt', '').lower()

for pattern in DANGEROUS_PATTERNS:
    if re.search(pattern, prompt):
        output = {
            "decision": "block",
            "reason": f"Dangerous pattern detected: {pattern}"
        }
        print(json.dumps(output))
        sys.exit(0)

# Allow prompt to proceed
print(json.dumps({"continue": True}))
```

### 2. PreToolUse

**When**: Fires after Claude creates tool parameters, before executing the tool

**Use Cases**:
- Validate commands before execution
- Block dangerous file operations
- Modify tool inputs (v2.0.10+)
- Enforce security policies
- Add approval workflows

**Control Options**:
```json
{
  "decision": "approve",   // Bypass permission system
  "decision": "deny",      // Block with feedback to Claude
  "decision": "ask",       // Prompt user confirmation
  "updatedInput": {...},   // Modify tool parameters (v2.0.10+)
  "reason": "Explanation for decision"
}
```

**Matcher Support**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|MultiEdit|Write",  // Only file modifications
        "hooks": [...]
      },
      {
        "matcher": "Bash",  // Only shell commands
        "hooks": [...]
      },
      {
        "matcher": "*",  // All tools
        "hooks": [...]
      }
    ]
  }
}
```

**Example - Block Dangerous Commands**:
```python
# pre_tool_use.py
import json
import sys
import re

hook_input = json.loads(sys.stdin.read())
tool_name = hook_input.get('tool_name')
tool_input = hook_input.get('tool_input', {})

# Only check Bash commands
if tool_name == 'Bash':
    command = tool_input.get('command', '')

    DANGEROUS_PATTERNS = [
        (r'rm\s+-rf\s+/', 'Recursive delete from root'),
        (r'chmod\s+777', 'Insecure permissions'),
        (r'sudo\s+rm', 'Elevated delete'),
        (r'>\s*/dev/sd[a-z]', 'Direct disk write'),
        (r'dd\s+if=', 'Direct disk operation'),
    ]

    for pattern, description in DANGEROUS_PATTERNS:
        if re.search(pattern, command):
            output = {
                "decision": "deny",
                "reason": f"Blocked dangerous command: {description}\\nCommand: {command}"
            }
            print(json.dumps(output))
            sys.exit(0)

# Check file operations
elif tool_name in ['Edit', 'Write', 'MultiEdit']:
    file_path = tool_input.get('file_path', '')

    PROTECTED_FILES = [
        '.env',
        'credentials.json',
        '.secret',
        'id_rsa',
        '.aws/credentials'
    ]

    if any(protected in file_path for protected in PROTECTED_FILES):
        output = {
            "decision": "ask",
            "reason": f"Modifying sensitive file: {file_path}\\nConfirm?"
        }
        print(json.dumps(output))
        sys.exit(0)

# Allow operation
print(json.dumps({"continue": True}))
```

**Example - Modify Tool Input (v2.0.10+)**:
```python
# pre_tool_use_modify.py
import json
import sys

hook_input = json.loads(sys.stdin.read())
tool_name = hook_input.get('tool_name')
tool_input = hook_input.get('tool_input', {})

if tool_name == 'Bash':
    command = tool_input.get('command', '')

    # Auto-add error handling to commands
    if 'set -e' not in command and '&&' not in command:
        modified_command = f"set -e\\n{command}"
        output = {
            "updatedInput": {
                "command": modified_command
            }
        }
        print(json.dumps(output))
        sys.exit(0)

print(json.dumps({"continue": True}))
```

### 3. PostToolUse

**When**: Fires immediately after a tool completes successfully

**Use Cases**:
- Validate tool outputs
- Run tests after code changes
- Format/lint after file edits
- Capture metrics
- Log successful patterns
- Provide feedback to Claude

**Input Data**:
```json
{
  "tool_name": "Edit",
  "tool_input": {...},
  "tool_response": {...}  // Results from tool execution
}
```

**Example - Run Tests After Code Changes**:
```python
# post_tool_use.py
import json
import sys
import subprocess

hook_input = json.loads(sys.stdin.read())
tool_name = hook_input.get('tool_name')
tool_input = hook_input.get('tool_input', {})

# Run tests after file modifications
if tool_name in ['Edit', 'Write', 'MultiEdit']:
    file_path = tool_input.get('file_path', '')

    # Only run tests for source files
    if file_path.endswith('.ts') or file_path.endswith('.js'):
        try:
            # Run relevant tests
            result = subprocess.run(
                ['npm', 'test', '--', '--related', file_path],
                capture_output=True,
                text=True,
                timeout=30
            )

            if result.returncode != 0:
                # Tests failed - inform Claude
                sys.stderr.write(f"Tests failed after modifying {file_path}:\\n{result.stdout}\\n{result.stderr}")
                sys.exit(2)  # Exit code 2 = blocking error

        except subprocess.TimeoutExpired:
            sys.stderr.write(f"Tests timed out for {file_path}")
            sys.exit(2)

sys.exit(0)  # Success
```

**Example - Auto-Format After Edits**:
```bash
#!/bin/bash
# post_tool_use_format.sh

# Read input
input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Format after file edits
if [[ "$tool_name" =~ ^(Edit|Write|MultiEdit)$ ]]; then
    if [[ "$file_path" =~ \\.ts$ || "$file_path" =~ \\.js$ ]]; then
        npx prettier --write "$file_path" 2>&1
    fi
fi

exit 0
```

**Example - Capture Successful Patterns**:
```python
# post_tool_use_learn.py
import json
import sys
import os

hook_input = json.loads(sys.stdin.read())
tool_name = hook_input.get('tool_name')
tool_input = hook_input.get('tool_input', {})
tool_response = hook_input.get('tool_response', {})

# Capture successful code patterns
if tool_name in ['Edit', 'Write']:
    file_path = tool_input.get('file_path', '')
    content = tool_input.get('content', '') or tool_input.get('new_string', '')

    # Extract patterns (simplified example)
    if 'firebase' in content.lower():
        pattern = {
            "type": "firebase",
            "file": file_path,
            "snippet": content[:200],
            "timestamp": hook_input.get('timestamp')
        }

        # Save to knowledge base
        knowledge_file = '.ai-knowledge/patterns.jsonl'
        os.makedirs('.ai-knowledge', exist_ok=True)
        with open(knowledge_file, 'a') as f:
            f.write(json.dumps(pattern) + '\\n')

sys.exit(0)
```

### 4. Stop

**When**: Fires when Claude finishes responding to a user prompt

**Use Cases**:
- Validate completion criteria
- Ensure tests pass before finishing
- Check requirements met
- Force continuation if incomplete
- Send notifications
- Commit changes

**Control Options**:
```json
{
  "decision": "block",  // Forces Claude to continue
  "reason": "Why Claude cannot stop yet"
}
```

**Example - Quality Gate**:
```python
# stop_quality_gate.py
import json
import sys
import subprocess

def run_tests():
    result = subprocess.run(['npm', 'test'], capture_output=True)
    return result.returncode == 0

def run_linter():
    result = subprocess.run(['npm', 'run', 'lint'], capture_output=True)
    return result.returncode == 0

def check_requirements():
    # Check if .plan/current-task.md requirements are met
    # (Simplified example)
    return True

hook_input = json.loads(sys.stdin.read())

# Run quality checks
if not run_tests():
    output = {
        "decision": "block",
        "reason": "Tests are failing. Please fix test failures before completing the task."
    }
    print(json.dumps(output))
    sys.exit(0)

if not run_linter():
    output = {
        "decision": "block",
        "reason": "Linter errors present. Please fix linting issues."
    }
    print(json.dumps(output))
    sys.exit(0)

if not check_requirements():
    output = {
        "decision": "block",
        "reason": "Task requirements not fully met. Please review .plan/current-task.md"
    }
    print(json.dumps(output))
    sys.exit(0)

# All checks passed
print(json.dumps({"continue": True}))
```

**Example - Desktop Notification**:
```bash
#!/bin/bash
# stop_notify.sh (macOS)

osascript -e 'display notification "Claude finished the task" with title "✅ Done" sound name "Glass"'
exit 0
```

### 5. SubagentStop

**When**: Fires when a subagent finishes its task

**Use Cases**:
- Validate subagent output
- Log subagent performance
- Force subagent continuation
- Coordinate multi-agent workflows

**Example - Subagent Validation**:
```python
# subagent_stop.py
import json
import sys

hook_input = json.loads(sys.stdin.read())
subagent_name = hook_input.get('subagent_name', 'unknown')

# Different validation for different subagents
if subagent_name == 'code-reviewer':
    # Ensure review was thorough
    # Check if review output contains required sections
    pass

elif subagent_name == 'tester':
    # Ensure tests were created and pass
    pass

print(json.dumps({"continue": True}))
```

### 6. SessionStart

**When**: Fires at the beginning of a Claude Code session

**Use Cases**:
- Initialize environment
- Load project context
- Set session variables
- Configure project-specific settings
- Inject knowledge base

**Environment Persistence**:
Write to `CLAUDE_ENV_FILE` to persist environment variables:

```python
# session_start.py
import json
import sys
import os

hook_input = json.loads(sys.stdin.read())
env_file = os.environ.get('CLAUDE_ENV_FILE')

# Load project preferences
preferences = {
    'PREFER_VITEST': 'true',
    'PREFER_FIREBASE': 'true',
    'NO_REACT': 'true',
    'NO_TAILWIND': 'true'
}

# Write to env file
if env_file:
    with open(env_file, 'w') as f:
        for key, value in preferences.items():
            f.write(f"{key}={value}\\n")

# Inject initial context
context = """
Project: Optimized AI System
Tech Stack: TypeScript, Firebase, Svelte, Vitest
Preferences: No React, No Tailwind, Minimal boilerplate

Recent Work:
- Phase 0: Experimental framework in progress
- Focus: Token optimization through skills and subagents
"""

output = {
    "hookSpecificOutput": {
        "additionalContext": context
    }
}

print(json.dumps(output))
```

### 7. SessionEnd

**When**: Fires when Claude Code session terminates

**Use Cases**:
- Cleanup temporary files
- Archive session transcripts
- Update metrics
- Save session learnings

**Example - Cleanup and Archive**:
```python
# session_end.py
import json
import sys
import os
import shutil
from datetime import datetime

hook_input = json.loads(sys.stdin.read())
transcript_path = hook_input.get('transcript_path')

# Archive transcript
if transcript_path and os.path.exists(transcript_path):
    archive_dir = '.claude/transcripts'
    os.makedirs(archive_dir, exist_ok=True)

    timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    archive_path = f"{archive_dir}/session_{timestamp}.jsonl"

    shutil.copy(transcript_path, archive_path)

# Cleanup temp files
temp_dir = '.plan/temp'
if os.path.exists(temp_dir):
    shutil.rmtree(temp_dir)

sys.exit(0)
```

### 8. Notification

**When**: Fires when Claude Code needs to notify the user

**Use Cases**:
- Custom notification handling
- Text-to-speech alerts
- External integrations
- Custom permission flows

**Example - TTS Notification**:
```python
# notification_tts.py
import json
import sys
import subprocess

hook_input = json.loads(sys.stdin.read())
notification_type = hook_input.get('notification_type')
message = hook_input.get('message', '')

# Use TTS for important notifications
if notification_type in ['permission_required', 'error']:
    subprocess.run(['say', message])

sys.exit(0)
```

### 9. PreCompact

**When**: Fires before Claude compacts the conversation context

**Use Cases**:
- Backup full transcript before compression
- Extract and save important information
- Update knowledge base before context loss

**Example - Backup Transcript**:
```python
# pre_compact.py
import json
import sys
import os
import shutil
from datetime import datetime

hook_input = json.loads(sys.stdin.read())
transcript_path = hook_input.get('transcript_path')

if transcript_path and os.path.exists(transcript_path):
    backup_dir = '.claude/backups'
    os.makedirs(backup_dir, exist_ok=True)

    timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    backup_path = f"{backup_dir}/pre_compact_{timestamp}.jsonl"

    shutil.copy(transcript_path, backup_path)

sys.exit(0)
```

## Configuration

### Location Priority

Hooks are checked in this order:
1. `~/.claude/settings.json` - Global (all projects)
2. `.claude/settings.json` - Project-specific (committed)
3. `.claude/settings.local.json` - Local overrides (not committed)

### Configuration Format

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",  // Optional, regex or wildcard
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/script.py",
            "timeout": 60  // Optional, default 60 seconds
          }
        ]
      }
    ]
  }
}
```

### Matcher Patterns

- `"*"` - Match all tools
- `"Edit|Write|MultiEdit"` - File modifications
- `"Bash"` - Shell commands
- `"Read"` - File reads
- `"Grep"` - Search operations
- `"mcp__memory__.*"` - MCP memory operations
- `"mcp__.*__write.*"` - All MCP write operations
- Empty string `""` - All events (for hooks without tool matching)

### Complete Example Configuration

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/inject_context.py",
            "timeout": 5
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/validate_command.py",
            "timeout": 10
          }
        ]
      },
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/protect_sensitive_files.py",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/format_file.sh",
            "timeout": 30
          },
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/run_tests.py",
            "timeout": 60
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/quality_gate.py",
            "timeout": 120
          },
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/notify.sh",
            "timeout": 5
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/initialize_session.py",
            "timeout": 10
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.claude/hooks/backup_transcript.py",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

## Hook Input/Output

### Input Format (stdin)

All hooks receive JSON via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/project/directory",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "src/index.ts",
    "old_string": "...",
    "new_string": "..."
  },
  "user_prompt": "Add error handling",
  "timestamp": "2025-11-03T10:30:00Z"
}
```

### Output Options

**1. Exit Codes**:
- `0` - Success, continue normally
- `2` - Blocking error (stderr message shown to Claude/user, stops operation)
- `Other` - Non-blocking error (stderr shown, execution continues)

**2. JSON Response**:
```json
{
  "continue": true,  // or false
  "decision": "block|approve|deny|ask",  // Hook-specific
  "reason": "Explanation",
  "stopReason": "User-facing message",
  "suppressOutput": false,
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "Extra context for Claude",
    "updatedInput": {...}  // Modified tool input (PreToolUse v2.0.10+)
  }
}
```

## Execution Behavior

### Parallelization

All matching hooks run simultaneously in parallel. Results are aggregated.

### Deduplication

Identical commands execute only once, even if multiple matchers trigger them.

### Timeout

- Default: 60 seconds
- Configurable per command
- Hook kills if exceeded

### Environment Variables

Available in all hooks:
- `CLAUDE_PROJECT_DIR` - Project root directory
- `CLAUDE_CODE_REMOTE` - Execution context (local/ssh)
- `CLAUDE_ENV_FILE` - Path for persisting environment variables (SessionStart only)
- `CLAUDE_PLUGIN_ROOT` - Plugin installation directory (for plugin-provided hooks)

## Security Considerations

### Critical Safety Rules

1. **Always validate and sanitize inputs**
   ```python
   # BAD
   os.system(user_input)

   # GOOD
   import shlex
   safe_input = shlex.quote(user_input)
   subprocess.run(['command', safe_input])
   ```

2. **Quote shell variables**
   ```bash
   # BAD
   rm $file_path

   # GOOD
   rm "$file_path"
   ```

3. **Block path traversal**
   ```python
   if '../' in file_path or file_path.startswith('/'):
       sys.exit(2)  # Block
   ```

4. **Use absolute paths for scripts**
   ```json
   {
     "command": "/home/user/.claude/hooks/script.py"  // GOOD
     "command": "script.py"  // BAD - relative path
   }
   ```

5. **Avoid sensitive file access**
   ```python
   SENSITIVE_PATHS = ['.env', 'credentials.json', '.secret', '.ssh/']
   if any(path in file_path for path in SENSITIVE_PATHS):
       # Block or ask for confirmation
   ```

### Snapshot Protection

Claude Code takes configuration snapshots at session start. External modifications during a session are ignored, preventing malicious injection.

## Debugging

### Enable Debug Mode

```bash
claude --debug
```

Shows:
- Which hooks matched
- Commands executed
- Output from each hook
- Timing information

### Logging Pattern

```python
import json
import sys
import logging

logging.basicConfig(
    filename='/tmp/claude_hook.log',
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

hook_input = json.loads(sys.stdin.read())
logging.debug(f"Hook input: {json.dumps(hook_input, indent=2)}")

# ... hook logic ...

logging.debug(f"Hook output: {json.dumps(output, indent=2)}")
print(json.dumps(output))
```

### Testing Hooks

```bash
# Test hook with mock input
echo '{"tool_name":"Edit","tool_input":{"file_path":"test.ts"}}' | python3 pre_tool_use.py
```

## Best Practices

### 1. Keep Hooks Fast

- Set appropriate timeouts
- Avoid expensive operations
- Cache results when possible
- Run async operations in background

### 2. Fail Gracefully

```python
try:
    # Hook logic
    output = {"continue": True}
except Exception as e:
    # Log error but don't block
    logging.error(f"Hook error: {e}")
    output = {"continue": True}

print(json.dumps(output))
```

### 3. Use Appropriate Exit Codes

- Exit 0 for non-blocking feedback
- Exit 2 only for critical blocking
- Include helpful stderr messages

### 4. Match Specifically

```json
{
  "matcher": "Edit|Write",  // Only file operations
  "hooks": [...]
}
```

Don't use `"*"` unless necessary - reduces unnecessary executions.

### 5. Isolate Hook Logic

Use UV single-file scripts with embedded dependencies:

```python
#!/usr/bin/env -S uv run
# /// script
# dependencies = ["requests", "anthropic"]
# ///

import json
import sys
# ... hook logic ...
```

### 6. Document Hook Behavior

```python
"""
Pre-Tool-Use Hook: Validate Bash Commands

Blocks dangerous commands that could harm the system:
- Recursive deletes from root
- Insecure permissions (chmod 777)
- Elevated dangerous operations

Exit codes:
- 0: Command is safe
- 2: Command is dangerous (blocks execution)
"""
```

## Integration with Optimized AI System

### For MINIMIZE Principle

**Hooks that reduce token usage**:
- SessionStart: Inject only relevant context
- PreCompact: Save critical information before compression
- Stop: Ensure completion before context grows

### For SEPARATE Principle

**Hooks that maintain separation**:
- PreToolUse matchers: Different hooks for different tool types
- SubagentStop: Validate subagent outputs independently
- PostToolUse: Tool-specific validation logic

### For VALIDATE Principle

**Hooks that ensure quality**:
- PostToolUse: Run tests after code changes
- Stop: Quality gates before completion
- PreToolUse: Validate inputs meet standards

### For LEARN Principle

**Hooks that capture learnings**:
- PostToolUse: Capture successful patterns
- SessionEnd: Save session metrics
- Stop: Log completion patterns

## Common Patterns

### Pattern 1: Spin Detection

```python
# Implement in PreToolUse with Edit matcher
import json
import sys
from datetime import datetime, timedelta

EDIT_HISTORY_FILE = '.claude/edit_history.json'

def load_edit_history():
    try:
        with open(EDIT_HISTORY_FILE, 'r') as f:
            return json.load(f)
    except:
        return {}

def save_edit_history(history):
    with open(EDIT_HISTORY_FILE, 'w') as f:
        json.dump(history, f)

hook_input = json.loads(sys.stdin.read())
file_path = hook_input.get('tool_input', {}).get('file_path', '')
now = datetime.now()

history = load_edit_history()
file_history = history.get(file_path, [])

# Add current edit
file_history.append(now.isoformat())

# Keep only recent edits (last 5 minutes)
recent_edits = [
    ts for ts in file_history
    if datetime.fromisoformat(ts) > now - timedelta(minutes=5)
]

# Check for spin (3+ edits in 2 minutes)
two_min_ago = now - timedelta(minutes=2)
recent_edit_count = sum(
    1 for ts in recent_edits
    if datetime.fromisoformat(ts) > two_min_ago
)

if recent_edit_count >= 3:
    output = {
        "decision": "deny",
        "reason": f"Spin detected: {file_path} edited {recent_edit_count} times in 2 minutes. Try a different approach."
    }
    print(json.dumps(output))
    sys.exit(0)

# Save updated history
history[file_path] = recent_edits
save_edit_history(history)

print(json.dumps({"continue": True}))
```

### Pattern 2: Auto-Commit on Stop

```python
# stop_auto_commit.py
import json
import sys
import subprocess

def get_git_status():
    result = subprocess.run(['git', 'status', '--short'], capture_output=True, text=True)
    return result.stdout.strip()

def get_last_prompt():
    # Extract from transcript
    hook_input = json.loads(sys.stdin.read())
    transcript_path = hook_input.get('transcript_path')
    if not transcript_path:
        return "Auto-commit"

    with open(transcript_path, 'r') as f:
        lines = f.readlines()
        for line in reversed(lines):
            data = json.loads(line)
            if data.get('type') == 'user_message':
                return data.get('content', '')[:72]  # First 72 chars

    return "Auto-commit"

status = get_git_status()
if status:  # Has changes
    message = get_last_prompt()
    subprocess.run(['git', 'add', '.'])
    subprocess.run(['git', 'commit', '-m', message])

sys.exit(0)
```

### Pattern 3: Multi-Session Isolation

```python
# pre_tool_use_session_isolation.py
import json
import sys
import os
import subprocess

hook_input = json.loads(sys.stdin.read())
session_id = hook_input.get('session_id')
tool_name = hook_input.get('tool_name')

# Create session-specific git index
if tool_name in ['Edit', 'Write', 'MultiEdit']:
    index_file = f".git/index.{session_id}"

    # Initialize session index if not exists
    if not os.path.exists(index_file):
        subprocess.run(['git', 'read-tree', '--index-output=' + index_file, 'HEAD'])

    # Set GIT_INDEX_FILE for this session
    env_file = os.environ.get('CLAUDE_ENV_FILE')
    if env_file:
        with open(env_file, 'a') as f:
            f.write(f"GIT_INDEX_FILE={index_file}\\n")

sys.exit(0)
```

## References

- [Official Hooks Documentation](https://docs.claude.com/en/docs/claude-code/hooks)
- [claude-code-hooks-mastery Repository](https://github.com/disler/claude-code-hooks-mastery)
- [GitButler Hooks Guide](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)
- [ClaudeLog Hooks Mechanics](https://claudelog.com/mechanics/hooks/)

---

**Last Updated**: 2025-11-03
