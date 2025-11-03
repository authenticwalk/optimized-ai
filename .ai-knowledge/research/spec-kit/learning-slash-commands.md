# Learning: Slash Commands Implementation

## Discovery Context

Discovered in `templates/commands/*.md` files and `README.md` documentation during spec-kit analysis on 2025-11-03.

Spec-Kit implements custom slash commands for AI agents (Claude Code, GitHub Copilot, Cursor, etc.) that provide structured workflows for software development.

## The Core Insight

**Slash commands are markdown files that expand into prompts when invoked.**

They work like on-demand skills:
- Each command is a separate `.md` file
- Commands have YAML frontmatter with metadata and scripts
- When user types `/speckit.specify`, the AI agent loads and processes that command file
- The command can execute scripts and provide structured guidance to the LLM

**Key Innovation**: Commands aren't just documentation - they're executable workflows that combine:
1. Script execution (bash/PowerShell)
2. Template loading
3. Structured LLM prompting
4. Validation logic

## Deep Analysis

### Structure of a Slash Command

```markdown
---
description: Short description of what this command does
scripts:
  sh: scripts/bash/script-name.sh --args
  ps: scripts/powershell/script-name.ps1 -Args
---

## User Input

```text
$ARGUMENTS
```

## Outline

[Step-by-step workflow the AI should follow]

## [Additional Sections]

[Detailed guidance, rules, examples]
```

### How It Works

1. **User Invokes**: `/speckit.specify Create a photo album app`

2. **Agent Loads**: `templates/commands/specify.md` file

3. **Script Execution**:
   - Frontmatter contains script references
   - Agent runs appropriate script (sh or ps based on OS)
   - Script outputs JSON with paths, branch names, etc.

4. **LLM Processing**:
   - Agent reads command content
   - Follows step-by-step outline
   - Uses user input (`$ARGUMENTS`)
   - Applies rules and guidelines

5. **Output Generation**:
   - Creates/updates files based on templates
   - Runs validation checks
   - Reports completion to user

### Key Components

#### 1. Frontmatter Metadata

```yaml
---
description: Create feature specification from natural language
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---
```

**Purpose**:
- Machine-readable command metadata
- Script execution definitions
- Platform-specific variants (bash vs PowerShell)

#### 2. User Input Capture

```markdown
## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).
```

**Purpose**:
- Explicitly marks where user's command arguments go
- Forces AI to acknowledge and use input
- Prevents AI from ignoring what user typed

#### 3. Structured Workflow (Outline)

```markdown
## Outline

1. **Setup**: Run `{SCRIPT}` and parse JSON output
2. **Load context**: Read existing files
3. **Execute workflow**: Follow step-by-step process
4. **Validate**: Run checks
5. **Report**: Output summary
```

**Purpose**:
- Clear, numbered steps
- Deterministic workflow
- Easy to follow and debug

#### 4. Detailed Guidance

```markdown
## General Guidelines

- Focus on **WHAT** users need, not **HOW** to implement
- Avoid technical details in specifications
- Mark ambiguities with [NEEDS CLARIFICATION: question]
```

**Purpose**:
- Constraints on LLM behavior
- Quality guidelines
- Best practices

### Commands in Spec-Kit

| Command | Purpose | Key Feature |
|---------|---------|-------------|
| `/speckit.constitution` | Create project principles | Template-based, versioned |
| `/speckit.specify` | Create feature spec | Auto-branch creation, validation |
| `/speckit.plan` | Generate tech plan | Research agents, gates |
| `/speckit.tasks` | Break down into tasks | Dependency-ordered, parallelizable |
| `/speckit.implement` | Execute implementation | Orchestrates actual coding |
| `/speckit.clarify` | Ask questions about spec | Structured questioning |
| `/speckit.analyze` | Validate consistency | Cross-artifact analysis |
| `/speckit.checklist` | Generate quality checks | Custom validation |

### Workflow Example

**User Goal**: Build a photo album app

**Step 1**: `/speckit.constitution Create principles for code quality`
- Loads `constitution.md` command
- Walks through principle creation
- Updates `memory/constitution.md`

**Step 2**: `/speckit.specify Build an app to organize photos in albums`
- Loads `specify.md` command
- Runs `create-new-feature.sh` script
  - Creates branch `001-photo-albums`
  - Creates `specs/001-photo-albums/` directory
- Generates `spec.md` from template
- Validates with checklist
- Reports branch and file locations

**Step 3**: `/speckit.plan Use Vite, vanilla JS, SQLite`
- Loads `plan.md` command
- Runs `setup-plan.sh` script
- Generates:
  - `plan.md` (technical approach)
  - `research.md` (tech decisions)
  - `data-model.md` (database schema)
  - `contracts/` (API specs)
- Updates agent context file (CLAUDE.md, GITHUB_COPILOT.md, etc.)

**Step 4**: `/speckit.tasks`
- Loads `tasks.md` command
- Analyzes all design documents
- Generates ordered task list
- Marks parallel opportunities
- Outputs `tasks.md`

**Step 5**: `/speckit.implement`
- Loads `implement.md` command
- Parses `tasks.md`
- Executes each task in order
- Runs tests, validates
- Creates working code

## Implementation Details

### Script Integration Pattern

Commands reference scripts but don't execute them directly:

```yaml
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
```

The `{ARGS}` token gets replaced with user's actual arguments.

**Script Execution**:
```bash
# Agent transforms this:
/speckit.specify Build photo album app

# Into this script call:
scripts/bash/create-new-feature.sh --json "Build photo album app"

# Script outputs JSON:
{
  "BRANCH_NAME": "001-photo-albums",
  "SPEC_FILE": "/path/to/specs/001-photo-albums/spec.md",
  "FEATURE_DIR": "/path/to/specs/001-photo-albums"
}

# Agent parses JSON and uses values in subsequent steps
```

### Template Loading Pattern

Commands reference templates that get loaded and filled:

```markdown
3. Load `templates/spec-template.md` to understand required sections.

4. Write the specification to SPEC_FILE using the template structure...
```

**Agent Process**:
1. Read template from filesystem
2. Identify placeholders (e.g., `[FEATURE NAME]`, `$ARGUMENTS`)
3. Fill placeholders with actual values
4. Write completed content to target file

### Validation Pattern

Commands include validation logic:

```markdown
6. **Specification Quality Validation**: After writing spec:

   a. **Create checklist**: Generate checklist file
   b. **Run validation**: Review spec against each item
   c. **Handle failures**: Update spec, re-run validation
   d. **Report**: Update checklist with pass/fail status
```

**Ensures**:
- Quality gates before proceeding
- Self-review built into workflow
- Iterative refinement

### Multi-Agent Coordination Pattern

Some commands spawn parallel agents:

```markdown
### Phase 0: Research
1. Extract unknowns from Technical Context
2. For each unknown:
     Task: "Research {unknown} for {feature context}"
3. Consolidate findings in research.md
```

**Enables**:
- Parallel research tasks
- Specialized sub-agents
- Coordinated results aggregation

## Experiments & Validation

### Evidence from Git History

Reviewing commits shows evolution:

1. **Initial commands** (early commits): Simple, linear
2. **Added validation** (mid commits): Checklists, gates
3. **Added parallelization** (recent): Research agents, parallel tasks
4. **Refined templates** (continuous): Reduced ambiguity, added constraints

**Pattern**: Commands evolved from simple prompts to sophisticated workflows

### User Feedback Analysis

From issues/PRs in spec-kit repo:

**Issue #598**: "Agent context file not updating correctly"
- **Learning**: Need explicit script for agent context updates
- **Solution**: Added `update-agent-context.sh` script
- **Incorporated**: Into `/speckit.plan` command

**Issue #1019**: "Branch number collisions"
- **Learning**: Must check remote branches, not just local
- **Solution**: Enhanced `create-new-feature.sh` to fetch remote
- **Incorporated**: Into `/speckit.specify` command

**Pattern**: Real-world usage drives command refinement

## Applicability to Our Project

### Direct Applications

#### 1. Enhanced Skills with Scripts

**Current** (`.claude/skills/research-conductor.md`):
```markdown
# Research Conductor Skill

## Purpose
Conduct focused research...

## Workflow
1. Clone repository
2. Analyze structure
3. Create documentation
```

**Enhanced** (with spec-kit pattern):
```markdown
---
description: Conduct focused research into external projects
scripts:
  sh: scripts/research/setup-research.sh --json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

## Outline

1. **Setup**: Run `{SCRIPT}` to create research directories
2. **Clone**: Automated git clone to .tmp/
3. **Analyze**: Structured analysis workflow
4. **Document**: Generate research artifacts
5. **Report**: Summary and next steps
```

**Benefits**:
- Script automation reduces manual steps
- JSON output provides structured data
- Clear workflow prevents confusion

#### 2. Command Aliases for Workflows

Create workflow commands in `.claude/commands/`:

```markdown
# .claude/commands/research.md
---
description: Research external project
scripts:
  sh: scripts/research/setup.sh --project "{ARGS}"
---

Quick workflow to research an external project and document learnings.

## Steps

1. Run setup script to clone and prepare
2. Load research-conductor skill
3. Execute structured analysis
4. Generate comprehensive documentation
5. Commit research to `.ai-knowledge/research/`
```

**Usage**: `/research https://github.com/example/project`

#### 3. Validation Commands

```markdown
# .claude/commands/validate-pr.md
---
description: Validate PR readiness
scripts:
  sh: scripts/validation/pre-pr-check.sh
---

Runs comprehensive checks before creating PR:

1. **Code Quality**
   - All tests pass
   - Linter clean
   - No TODOs/FIXMEs

2. **Self-Evaluation**
   - Changes match task requirements
   - Edge cases considered
   - No new spin incidents

3. **Knowledge Update**
   - Learnings documented
   - Patterns updated
   - Metrics recorded

4. **Report**: Create PR with summary
```

**Usage**: `/validate-pr`

### Adaptations for Our System

#### Keep Our Approach Better In These Areas:

1. **Learning System**: Spec-kit has no learning - we keep `.ai-knowledge/`
2. **Spin Detection**: Spec-kit has no monitoring - we keep our detection
3. **Minimal First**: Spec-kit templates are comprehensive - we stay minimal
4. **Flexible Workflow**: Spec-kit is formal phases - we allow flexibility

#### Adopt From Spec-Kit:

1. **Script Integration**: Add `scripts:` to our skills
2. **Validation Gates**: Add quality checklists
3. **JSON Outputs**: Scripts output structured data
4. **Multi-Agent Spawning**: Research agents pattern

### Implementation Roadmap

#### Phase 1: Add Script Support
- Add `scripts/` directory structure
- Enhance skills with `scripts:` frontmatter
- Create core automation scripts:
  - `scripts/research/setup-research.sh`
  - `scripts/validation/pre-commit.sh`
  - `scripts/features/create-feature.sh`

#### Phase 2: Add Validation Commands
- Create `.claude/commands/` directory
- Add workflow commands:
  - `research.md` - Research workflow
  - `validate.md` - Quality validation
  - `feature.md` - Feature creation
- Create validation checklists

#### Phase 3: Advanced Coordination
- Multi-agent research spawning
- Parallel task execution
- Result aggregation patterns

## Trade-Offs

### Advantages of Slash Commands Pattern

✅ **Clear Workflows**: Step-by-step guidance prevents confusion
✅ **Script Automation**: Reduces manual repetitive tasks
✅ **Quality Gates**: Built-in validation improves outcomes
✅ **Reusable**: Once created, available for all similar tasks
✅ **Versionable**: Commands are files, can be versioned and improved

### Disadvantages to Consider

❌ **Overhead**: Creating commands takes time
❌ **Rigidity**: Formal workflows might slow quick iterations
❌ **Learning Curve**: Users must learn command syntax
❌ **Maintenance**: Commands need updates as workflows evolve

### Balance for Our Project

**Recommendation**: Hybrid approach
- Core workflows → Commands (research, feature, validate)
- Exploratory work → Flexible (direct conversation)
- Learning system → Always active (passive capture)

## Related Learnings

- [Template-Constrained LLM Behavior](./learning-template-constraints.md) - How templates guide commands
- [Script-Based Automation](./learning-script-automation.md) - Script implementation patterns
- [Checklist Validation](./pattern-checklist-validation.md) - Quality gates

## Key Takeaways

1. **Slash commands = Executable workflows** - Not just documentation
2. **Scripts + Templates + Prompts** - Powerful combination
3. **Validation built-in** - Quality gates prevent downstream issues
4. **Platform-agnostic** - Works across Claude, Copilot, Cursor, etc.
5. **Iterative refinement** - Commands improve based on real usage

## Open Questions

1. How to handle command versioning when format changes?
2. Can commands call other commands (composition)?
3. How to test commands before deploying to users?
4. What's the optimal command granularity?
5. How to handle command failures mid-workflow?

## Next Steps

1. Design our command structure for `.claude/commands/`
2. Create core scripts in `scripts/` directory
3. Implement validation pattern in workflows
4. Experiment: Command vs no-command for same task
5. Document best practices from experiments

---

**Related Files**:
- Source: `.tmp/spec-kit/templates/commands/*.md`
- Our Skills: `.claude/skills/*.md`
- Our Principles: `.plan/initial-design/principles/`

**Tags**: #slash-commands #workflows #automation #skills #validation
