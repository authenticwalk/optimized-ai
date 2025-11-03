# Learning: Script-Based Automation

## Discovery Context

Discovered in `scripts/bash/*.sh` and `scripts/powershell/*.ps1` files during spec-kit analysis on 2025-11-03.

Spec-Kit uses bash and PowerShell scripts to automate workflows, avoiding the need for MCP middleware. Scripts are called from slash commands and output structured JSON for the LLM to consume.

## The Core Insight

**Scripts > MCPs for most automation needs.**

Instead of building MCP servers as middleware:
1. Write simple bash/PowerShell scripts
2. Output JSON for structured data
3. Call scripts directly from commands/skills
4. LLM parses JSON and continues workflow

**Validates our ADR-003**: Direct CLI scripts over MCP middleware.

## Deep Analysis

### Script Pattern

```bash
#!/usr/bin/env bash
set -e  # Exit on error

# Parse arguments
JSON_MODE=false
while [ $i -le $# ]; do
    case "$arg" in
        --json) JSON_MODE=true ;;
        --arg) SOME_ARG="$value" ;;
    esac
done

# Do work
# ... (git operations, file creation, etc.)

# Output results
if [ "$JSON_MODE" = true ]; then
    # Structured JSON output for LLM
    cat <<EOF
{
  "BRANCH_NAME": "$branch",
  "SPEC_FILE": "$spec_file",
  "FEATURE_DIR": "$feature_dir"
}
EOF
else
    # Human-readable output
    echo "Created branch: $branch"
    echo "Spec file: $spec_file"
fi
```

### Key Scripts in Spec-Kit

| Script | Purpose | Key Features |
|--------|---------|--------------|
| `create-new-feature.sh` | Create feature branch and spec | Auto-numbering, remote check, JSON output |
| `setup-plan.sh` | Initialize planning artifacts | Directory creation, template copying |
| `check-prerequisites.sh` | Validate environment | Tool detection, JSON output |
| `update-agent-context.sh` | Update CLAUDE.md/etc | Preserves manual edits, adds tech stack |

### Example: create-new-feature.sh

**What It Does**:
1. Generates short name from description
2. Checks remote branches for existing numbers
3. Calculates next available number
4. Creates feature branch (e.g., `003-photo-albums`)
5. Creates specs directory
6. Outputs JSON with paths

**JSON Output**:
```json
{
  "BRANCH_NAME": "003-photo-albums",
  "SPEC_FILE": "/path/to/specs/003-photo-albums/spec.md",
  "FEATURE_DIR": "/path/to/specs/003-photo-albums"
}
```

**Why JSON**: LLM can reliably parse and use these values in subsequent steps.

### Script Design Patterns

#### Pattern 1: JSON Mode Flag

```bash
JSON_MODE=false
--json) JSON_MODE=true ;;

if [ "$JSON_MODE" = true ]; then
    # Machine-readable
else
    # Human-readable
fi
```

**Purpose**: Same script serves both CLI users and LLM consumers

#### Pattern 2: Error Handling

```bash
set -e  # Exit on error

if [ -z "$REQUIRED_ARG" ]; then
    echo "Error: Missing required argument" >&2
    exit 1
fi
```

**Purpose**: Fail fast with clear error messages

#### Pattern 3: Platform Detection

Spec-kit provides both bash and PowerShell versions:
- `scripts/bash/*.sh` for Linux/macOS
- `scripts/powershell/*.ps1` for Windows

Commands specify both:
```yaml
scripts:
  sh: scripts/bash/script.sh
  ps: scripts/powershell/script.ps1
```

#### Pattern 4: Git Safety

```bash
# Fetch remotes to get latest info
git fetch --all --prune 2>/dev/null || true

# Check both remote and local branches
remote_branches=$(git ls-remote --heads origin | grep pattern)
local_branches=$(git branch | grep pattern)

# Find highest number from both sources
max_num=...
```

**Purpose**: Prevent branch number collisions

## Applicability to Our Project

### Scripts We Should Create

#### 1. Research Setup Script

```bash
#!/usr/bin/env bash
# scripts/research/setup-research.sh

PROJECT_URL="$1"
PROJECT_NAME=$(basename "$PROJECT_URL" .git)

mkdir -p .tmp research/"$PROJECT_NAME"
cd .tmp && git clone "$PROJECT_URL" "$PROJECT_NAME"

if [ "$JSON_MODE" = true ]; then
    cat <<EOF
{
  "PROJECT_DIR": ".tmp/$PROJECT_NAME",
  "RESEARCH_DIR": "research/$PROJECT_NAME",
  "PROJECT_NAME": "$PROJECT_NAME"
}
EOF
fi
```

**Usage**: From research-conductor skill

#### 2. Feature Creation Script

```bash
#!/usr/bin/env bash
# scripts/features/create-feature.sh

FEATURE_NAME="$1"
BRANCH_NAME="feature/$FEATURE_NAME"

git checkout -b "$BRANCH_NAME"
mkdir -p ".ai-knowledge/plans/$FEATURE_NAME"

if [ "$JSON_MODE" = true ]; then
    cat <<EOF
{
  "BRANCH_NAME": "$BRANCH_NAME",
  "PLAN_DIR": ".ai-knowledge/plans/$FEATURE_NAME"
}
EOF
fi
```

#### 3. Validation Script

```bash
#!/usr/bin/env bash
# scripts/validation/pre-commit-check.sh

# Run tests
npm test || exit 1

# Run linter
npm run lint || exit 1

# Check workspace cleanliness
if [ -n "$(find . -maxdepth 1 -name '*.tmp' -o -name '*.log')" ]; then
    echo "Error: Temporary files in workspace" >&2
    exit 1
fi

if [ "$JSON_MODE" = true ]; then
    cat <<EOF
{
  "TESTS": "PASS",
  "LINTER": "PASS",
  "WORKSPACE": "CLEAN"
}
EOF
fi
```

### Integration with Skills

Update research-conductor skill:

```markdown
---
name: research-conductor
description: Research external projects
scripts:
  sh: scripts/research/setup-research.sh --json
---

## Workflow

1. **Setup**: Run `{SCRIPT}` with project URL
2. **Parse JSON**: Extract PROJECT_DIR, RESEARCH_DIR
3. **Analyze**: Follow research methodology
4. **Document**: Create structured artifacts

[Rest of skill...]
```

### Benefits

✅ **Simple**: No MCP protocol complexity
✅ **Testable**: Scripts can be tested independently
✅ **Debuggable**: Run scripts manually to debug
✅ **Maintainable**: Standard bash/PowerShell
✅ **Fast**: Direct execution, no middleware
✅ **Portable**: Works across different AI agents

## Key Takeaways

1. **Scripts > MCPs** - For most automation needs
2. **JSON Output** - Enables LLM parsing
3. **Platform Support** - Provide bash + PowerShell
4. **Error Handling** - Fail fast with clear messages
5. **Git Safety** - Check remotes, handle conflicts
6. **Testable** - Can run independently

## Implementation Priority

### Phase 1: Core Scripts
- `scripts/research/setup-research.sh`
- `scripts/validation/pre-commit-check.sh`

### Phase 2: Feature Scripts
- `scripts/features/create-feature.sh`
- `scripts/features/setup-plan.sh`

### Phase 3: Advanced Scripts
- `scripts/validation/pr-readiness.sh`
- `scripts/learning/update-knowledge.sh`

## Next Steps

1. Create `scripts/` directory structure
2. Implement Phase 1 scripts
3. Update skills to reference scripts
4. Test script integration
5. Document script patterns

---

**Key Insight**: Simple bash scripts with JSON output provide 90% of the value of MCPs with 10% of the complexity.

**Tags**: #scripts #automation #json #bash #powershell #adr-003
