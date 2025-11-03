# Advanced Features

**Topics Covered**: Skills, Hooks, MCP Integration

## Overview

Advanced features extend Claude Code's capabilities through custom instructions (Skills), automated workflows (Hooks), and external integrations (MCP servers).

## Skills

### What Are Skills?

**Skills**: Focused instruction sets loaded on-demand for specific tasks

**Key characteristics**:
- Stored in `.claude/skills/` directory
- Simple YAML frontmatter + Markdown instructions
- Auto-loaded based on triggers (keywords, file patterns)
- Model-invoked (Claude decides when to use)
- ~100 lines each (keep focused)

### Skill Structure

**File format**: SKILL.md

```markdown
---
name: firebase-auth
description: Firebase Authentication patterns and best practices. Use when implementing login, signup, or auth flows with Firebase.
triggers:
  keywords:
    - firebase
    - authentication
    - login
    - signup
    - auth
  files:
    - "firebase.config.*"
    - "auth/*.ts"
    - "**/auth.ts"
---

# Firebase Authentication Skill

## Setup Pattern

[Detailed instructions on Firebase auth setup...]

## Login Flow

[Best practices for login implementation...]

## Error Handling

[Common errors and how to handle them...]

## Security Best Practices

[Security guidelines...]
```

### How Skills Work

**Automatic Detection**:
1. User provides task: "Implement Firebase authentication"
2. Claude analyzes keywords: "Firebase", "authentication"
3. Matches skill triggers
4. Loads `firebase-auth` skill into context
5. Follows skill instructions

**Context size**:
- Core .cursorrules: 80 lines
- firebase-auth skill: 100 lines
- **Total**: 180 lines

vs. monolithic config: 500+ lines

**Savings**: 64% fewer tokens

### Creating Skills

**Location**: `.claude/skills/<category>/<skill-name>/SKILL.md`

**Example structure**:
```
.claude/
└── skills/
    ├── firebase/
    │   ├── auth/
    │   │   └── SKILL.md
    │   └── firestore/
    │       └── SKILL.md
    ├── testing/
    │   └── vitest/
    │       └── SKILL.md
    └── patterns/
        └── refactoring/
            └── SKILL.md
```

### Skill Best Practices

**DO**:
- ✅ Keep skills ~100 lines (focused)
- ✅ Use descriptive triggers (keywords + file patterns)
- ✅ Include specific, actionable instructions
- ✅ Provide examples and patterns
- ✅ Test with actual tasks

**DON'T**:
- ❌ Create 500-line skills (defeats purpose)
- ❌ Overlap with core .cursorrules
- ❌ Use vague triggers that match everything
- ❌ Include unnecessary background information

### Skill Granularity

**Too broad** (anti-pattern):
```markdown
---
name: firebase
description: Everything Firebase-related
---
# Firebase Skill (500 lines)
[Auth, Firestore, Functions, Hosting, Storage, etc.]
```
**Problem**: Loads unnecessary context

**Optimal** (split into focused skills):
```markdown
firebase/auth/SKILL.md        (~100 lines)
firebase/firestore/SKILL.md   (~100 lines)
firebase/functions/SKILL.md   (~100 lines)
firebase/hosting/SKILL.md     (~80 lines)
```

**Too narrow** (anti-pattern):
```markdown
firebase/email-auth/SKILL.md
firebase/google-auth/SKILL.md
firebase/facebook-auth/SKILL.md
```
**Problem**: Too many skills, management overhead

**Optimal** (combine related):
```markdown
firebase/auth/SKILL.md
[Sections: Email, Social providers, Custom auth]
```

### Manual Skill Loading

**Command-line**:
```bash
# Load specific skills
claude --skills firebase-auth,testing "Implement login feature"

# Exclude all skills
claude --no-skills "Fix typo in README"
```

**In conversation**:
```
User: "Use the firebase-auth and testing skills to implement login"
```

### Skill Combinations

**Track which skills work well together**:
```json
{
  "skill_combinations": {
    "firebase-auth + testing": {
      "frequency": 45,
      "avg_success": 0.92
    },
    "refactoring + testing": {
      "frequency": 60,
      "avg_success": 0.95
    }
  }
}
```

**Learning**: When firebase-auth loaded, suggest testing skill

### Skills vs Hooks vs MCP

**Skills**:
- Instructions for Claude to follow
- Loaded into context
- No code execution
- Example: "When using Firebase auth, follow this pattern..."

**Hooks**:
- Automated scripts that run at specific events
- Execute actual code
- Example: "After editing Python file, run black formatter"

**MCP**:
- External tool/data integrations
- Provide capabilities Claude doesn't have
- Example: "Query Jira API for ticket details"

**When to use each**:
- Skill: Teaching Claude patterns/best practices
- Hook: Automating repetitive tasks
- MCP: Accessing external systems

## Hooks

### What Are Hooks?

**Hooks**: Automated shell commands that execute at specific lifecycle events

**Key characteristics**:
- Configured in settings.json
- Run before/after tool execution
- Can block, modify, or observe operations
- Receive JSON input via stdin
- Return results via stdout/exit codes

### Hook Events

**9 hook types**:

1. **PreToolUse**: Before tool execution (can block/modify)
2. **PostToolUse**: After tool completes
3. **UserPromptSubmit**: When user submits prompt
4. **Notification**: When Claude sends notification
5. **SessionStart**: Session startup/resume
6. **SessionEnd**: Session termination
7. **Stop**: When Claude finishes responding
8. **SubagentStop**: When subagent completes
9. **PreCompact**: Before context compaction

### Configuration

**Location**: `~/.claude/settings.json`, `.claude/settings.json`, or `.claude/settings.local.json`

**Structure**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-edit-check.sh",
            "timeout": 60
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
            "command": "~/.claude/hooks/format-file.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Hook Input Format

**JSON via stdin**:
```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "old_string": "foo",
    "new_string": "bar"
  }
}
```

### Hook Output Options

**1. Simple (Exit Code Only)**:
```bash
#!/bin/bash
# Check if file is production config
if [[ "$file_path" == *"production.json" ]]; then
  echo "ERROR: Cannot edit production config!" >&2
  exit 2  # Blocking error
fi
exit 0  # Success
```

**2. Advanced (JSON Output)**:
```bash
#!/bin/bash
cat <<EOF
{
  "continue": false,
  "decision": "deny",
  "stopReason": "Production config files are read-only",
  "suppressOutput": false,
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "additionalContext": "Edit .env.local instead"
  }
}
EOF
```

### Exit Codes

- `0`: Success (stdout shown, or becomes context for UserPromptSubmit/SessionStart)
- `2`: Blocking error (stderr fed to Claude, operation blocked)
- Other: Non-blocking error (logged but continues)

### Common Hook Patterns

**Auto-format after edit**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/format.sh"
          }
        ]
      }
    ]
  }
}
```

**format.sh**:
```bash
#!/bin/bash
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Format based on file type
case "$file_path" in
  *.py)
    black "$file_path"
    ;;
  *.ts|*.tsx|*.js|*.jsx)
    prettier --write "$file_path"
    ;;
  *.go)
    gofmt -w "$file_path"
    ;;
esac

exit 0
```

**Inject context at session start**:
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/session-context.sh"
          }
        ]
      }
    ]
  }
}
```

**session-context.sh**:
```bash
#!/bin/bash
cat <<EOF
Current time: $(date)
Recent changes: $(git log --oneline -5)
Active branch: $(git branch --show-current)
Uncommitted files: $(git status --short | wc -l)
EOF
exit 0
```

**Block sensitive file edits**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

**protect-files.sh**:
```bash
#!/bin/bash
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Block sensitive files
case "$file_path" in
  *.env|*.key|*.pem|*credentials*)
    echo "ERROR: Cannot modify sensitive file: $file_path" >&2
    exit 2
    ;;
esac

exit 0
```

### Security Considerations

**CRITICAL**: Hooks execute arbitrary shell commands automatically

**Best practices**:
1. **Validate inputs**: Never trust hook input without sanitization
2. **Quote variables**: Always use `"$var"` not `$var`
3. **Absolute paths**: Use full paths in scripts
4. **Block path traversal**: Prevent `../` patterns
5. **Limit permissions**: Run with minimal privileges
6. **Test thoroughly**: In safe environment first
7. **Avoid sensitive files**: Don't access .env, keys, etc.

**Example of unsafe hook**:
```bash
#!/bin/bash
# UNSAFE - don't do this!
file=$(echo "$input" | jq -r '.tool_input.file_path')
cat $file  # Vulnerable to injection
```

**Safe version**:
```bash
#!/bin/bash
file=$(echo "$input" | jq -r '.tool_input.file_path')

# Validate path
if [[ "$file" == *".."* ]]; then
  echo "ERROR: Path traversal detected" >&2
  exit 2
fi

# Quote variable
cat "$file"
```

### Debugging Hooks

**Enable debug mode**:
```bash
claude --debug
```

**Shows**:
- Which hooks matched
- Commands executed
- Exit status and output
- Execution time

**Common issues**:
- Unescaped quotes in JSON
- Incorrect matcher patterns
- Missing script permissions (`chmod +x`)
- Environment variable access problems

### Hook Performance

**Latency impact**:
- Each hook adds execution time
- PreToolUse hooks block operations
- Multiple hooks run in parallel
- Timeout defaults to 60 seconds

**Optimization**:
- Keep hooks fast (<1 second)
- Use timeouts to prevent hangs
- Run expensive operations async
- Batch operations when possible

## MCP (Model Context Protocol)

### What is MCP?

**MCP**: Model Context Protocol - enables Claude to connect to external tools and data sources

**Key characteristics**:
- Separate server processes
- Tool naming: Prefix with `mcp__`
- Configured in `.claude/settings.json`
- Can provide tools, resources, prompts

### MCP vs Direct Scripts

**From project principles (SEPARATE)**:

**Prefer direct scripts**:
```bash
# Instead of MCP server for Firebase
# Just use Firebase CLI directly
firebase firestore:get users/user123
firebase auth:export auth-users.json
```

**Use MCP when**:
- ✅ Complex state management needed
- ✅ Real-time data streaming
- ✅ Cross-tool coordination
- ✅ Custom protocol required

**Default**: Try scripts first, add MCP only if proven necessary

### MCP Configuration

**Location**: `.claude/settings.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/files"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
}
```

### Popular MCP Servers

**Development**:
- `@modelcontextprotocol/server-filesystem`: Local file access
- `@modelcontextprotocol/server-github`: GitHub API access
- `@modelcontextprotocol/server-gitlab`: GitLab integration

**Project Management**:
- Atlassian MCP: Jira tickets, Confluence docs
- Linear MCP: Linear issues and projects

**Monitoring**:
- Sentry MCP: Production error tracking
- Datadog MCP: Metrics and monitoring

**Automation**:
- Puppeteer MCP: Browser automation
- Playwright MCP: Web testing

### MCP Tool Usage

**Automatic prefix**:
```typescript
// MCP tools are prefixed with mcp__
mcp__github__create_issue({
  title: "Bug: Login fails",
  body: "Description..."
})

mcp__filesystem__read_file({
  path: "/allowed/path/file.txt"
})
```

**In prompts**:
```
User: "Create a GitHub issue for this bug"
// Claude automatically uses mcp__github__create_issue
```

### Auto-Approval for MCP Tools

**Configuration**:
```json
{
  "permissions": {
    "allow": [
      "mcp__filesystem__*",
      "mcp__github__get_*"
    ],
    "deny": [
      "mcp__github__delete_*"
    ]
  }
}
```

**Auto-approve mode**: Shift+Tab to cycle modes

### Limitations for Business Use

**MCP servers are developer-focused**:
- Built by developers, for developers
- Require technical setup
- Node.js/Python dependencies
- Not user-friendly for non-technical users

**For business automation**: Use dedicated platforms instead

## Integration Pattern: Skills + Hooks + MCP

### Example: Firebase Development Workflow

**Skill** (firebase-auth/SKILL.md):
```markdown
---
name: firebase-auth
description: Firebase Authentication best practices
---

# Firebase Auth Patterns

[Instructions on how to implement Firebase auth...]
```

**Hook** (auto-check Firebase connection):
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/firebase-check.sh"
          }
        ]
      }
    ]
  }
}
```

**firebase-check.sh**:
```bash
#!/bin/bash
# Check Firebase CLI is available
if command -v firebase &> /dev/null; then
  echo "Firebase CLI: ✓"
  echo "Active project: $(firebase use)"
else
  echo "Firebase CLI: ✗ (install with: npm install -g firebase-tools)"
fi
exit 0
```

**Direct Script** (no MCP needed):
```
Claude: Let me check your Firebase auth users
Command: firebase auth:export users.json --format=JSON
[Analyzes output]
```

**Result**:
- Skill provides patterns
- Hook checks environment
- Direct script queries data
- No MCP overhead!

## Summary: Advanced Features Optimization

### Skills (SEPARATE + MINIMIZE)

1. **Keep skills ~100 lines**: Focused, single-purpose
2. **Auto-load via triggers**: Keywords + file patterns
3. **Combine related concepts**: Don't over-split
4. **Test with real tasks**: Validate triggers work

### Hooks (AUTOMATE + VALIDATE)

1. **Auto-format after edits**: Prettier, Black, gofmt
2. **Block sensitive files**: Protect .env, keys
3. **Inject context at start**: Recent changes, branch info
4. **Keep hooks fast**: <1 second execution
5. **Security first**: Validate all inputs

### MCP (INTEGRATE SPARINGLY)

1. **Prefer direct scripts**: Firebase CLI, git, etc.
2. **Use MCP for complex needs**: State management, streaming
3. **Auto-approve safely**: Allow reads, deny deletes
4. **Document dependencies**: Node.js, Python, etc.

### Integration Strategy

1. **Skill**: Teach patterns and best practices
2. **Hook**: Automate repetitive tasks
3. **Script**: Query external systems
4. **MCP**: Only when scripts insufficient

**Goal**: Maximum capability, minimum complexity

---

**Next**: [Best Practices for Minimization](./09-BEST-PRACTICES-MINIMIZE.md)
