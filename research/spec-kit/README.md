# Spec-Kit Research

## What They Built

**Spec-Kit** is an open-source toolkit by GitHub that implements "Spec-Driven Development" (SDD) - a methodology where specifications become executable and directly generate working code, rather than just guiding implementation.

**Core Value Proposition**: Focus on product scenarios and predictable outcomes instead of "vibe coding" every piece from scratch.

**Approach**: Specifications → Implementation Plans → Tasks → Code (with AI agents doing the transformation)

## Why We're Researching This

Our project (Optimized AI) is building a self-learning AI coding assistant for PM-mode workflow. Spec-Kit aligns with several of our goals:

### Alignment with Our Principles

**From `.plan/initial-design/SPEC.md` and principles**:

1. **PM-Mode Workflow** (Our Goal): User creates tasks, AI implements
   - **Spec-Kit Match**: User writes spec, AI generates plan/tasks/code via slash commands

2. **Minimize Principle** (`.plan/initial-design/principles/1-minimize.md`): Maximum results, minimum overhead
   - **Spec-Kit Match**: Templates constrain LLM behavior, reducing token waste and confusion

3. **Separate Principle** (`.plan/initial-design/principles/2-separate.md`): Context isolation through skills
   - **Spec-Kit Match**: Slash commands are like skills - loaded on-demand for specific workflows

4. **Self-Evaluation** (Our Goal): AI validates before committing
   - **Spec-Kit Match**: Checklist-based validation at each phase

5. **Direct Scripts** (Our ADR-003): Prefer CLI over MCP
   - **Spec-Kit Match**: Bash/PowerShell scripts automate workflows directly

## What We Explored

We conducted deep analysis on the following aspects that align with our project goals:

### Core Insights (One file per insight)

1. **[Slash Commands Implementation](./learning-slash-commands.md)** - How custom AI agent commands work
2. **[Template-Constrained LLM Behavior](./learning-template-constraints.md)** - Using templates to reduce tokens and improve quality
3. **[Constitution-Driven Development](./learning-constitution.md)** - Immutable principles that enforce architecture
4. **[Script-Based Automation](./learning-script-automation.md)** - Bash/PS scripts for workflow automation
5. **[Phase-Based Workflow](./pattern-phase-workflow.md)** - Structured multi-phase development
6. **[Git Branch Management](./learning-git-automation.md)** - Automatic numbering and branch creation
7. **[Checklist Validation](./pattern-checklist-validation.md)** - Self-review through structured checklists
8. **[Separation of What vs How](./pattern-separation-of-concerns.md)** - Spec (what) → Plan (how) → Tasks → Code

### Complex Topics Requiring Deeper Analysis

9. **[Templates System](./templates/README.md)** - Deep dive into template design and structure
10. **[Scripts System](./scripts/README.md)** - Deep dive into automation scripts

## What We Skipped

Areas we didn't explore deeply (not relevant to our CLI focus):

- **VSCode-specific integrations** - We're building CLI-first, not IDE extensions
- **Python CLI implementation details** - We're using TypeScript
- **Agent-specific variations** (Cursor, Gemini, Copilot) - We're focused on Claude Code primarily
- **Documentation generation** (docfx) - Different tooling approach
- **Deployment/release workflows** - Not our current phase

## Key Insights Summary

### What Works Well

1. **Slash Commands = On-Demand Skills**
   - Maps to our "SEPARATE" principle
   - Each command is focused, single-purpose (like skills)
   - Loaded only when needed → token efficiency

2. **Templates Constrain LLMs**
   - Prevents over-engineering and premature implementation details
   - Forces explicit uncertainty markers (`[NEEDS CLARIFICATION]`)
   - Built-in checklists act as "unit tests for specifications"
   - Aligns with our "MINIMIZE" principle

3. **Constitution as Immutable Principles**
   - Similar to our principles/ folder
   - Enforced through template checkpoints
   - Provides consistency across all AI-generated code
   - Matches our "VALIDATE" principle

4. **Scripts > MCPs**
   - Directly validates our ADR-003 decision
   - Simple bash scripts drive workflows
   - No middleware overhead
   - Easy to test and maintain

5. **Phase Gates & Checklists**
   - Self-validation before proceeding
   - Matches our "self-evaluation before commit" requirement
   - Quality gates prevent broken code

### Potential Issues & Learnings

1. **Heavy on Structure** (Potential Issue)
   - Very formal workflow (constitution → specify → plan → tasks → implement)
   - Might be too rigid for quick iterations
   - **Our Approach**: More flexible - user can start with task description directly

2. **Template Verbosity** (Observation)
   - Templates are comprehensive but long
   - Might conflict with our "minimize" principle
   - **Our Approach**: Start with minimal, expand only when validated

3. **Limited Learning System** (Gap)
   - Spec-Kit doesn't appear to learn from past specs/patterns
   - No knowledge base or pattern recognition
   - **Our Advantage**: Our "LEARN" principle with `.ai-knowledge/` folder

4. **Branch-Per-Feature Overhead** (Trade-off)
   - Every feature gets a new numbered branch
   - Great for organization, but adds overhead for small changes
   - **Our Approach**: Support this but don't mandate it

## How This Applies to Our Project

### Specific Recommendations

#### 1. Implement Slash Commands as Skills

**Current State**: We have skill files in `.claude/skills/`

**Spec-Kit Pattern**: Slash commands in `templates/commands/*.md`

**Action**:
- ✅ Keep our skills approach (already aligned)
- ✅ Add script references in skills (like their `scripts:` section)
- ✅ Consider command aliases for common workflows

**Example** (update our research-conductor skill):
```yaml
name: research-conductor
description: Research external projects
scripts:
  sh: scripts/research/clone-and-analyze.sh
  create-docs: scripts/research/generate-docs.sh
```

#### 2. Add Template-Based Validation

**Current State**: No formal templates

**Spec-Kit Pattern**: Templates with placeholders and validation checklists

**Action**:
- Create minimal templates for our workflows:
  - `templates/research-template.md` (for research conductor)
  - `templates/feature-template.md` (for new features)
  - `templates/skill-template.md` (for new skills)
- Include checklists in each template
- Keep them SMALL (aligned with our minimize principle)

#### 3. Formalize Our Constitution

**Current State**: principles/*.md files

**Spec-Kit Pattern**: Single constitution.md with versioning

**Action**:
- Create `.plan/initial-design/CONSTITUTION.md`
- Consolidate all principles
- Add version tracking and amendment process
- Make it referenceable in skills/commands

#### 4. Script Automation

**Current State**: Minimal scripts

**Spec-Kit Pattern**: Comprehensive bash/PS scripts for workflows

**Action**:
- Add `scripts/` directory:
  - `scripts/research/` - Research automation
  - `scripts/features/` - Feature creation
  - `scripts/validation/` - Quality checks
- Keep scripts simple and direct (no MCP wrapper needed)

#### 5. Validation Checklists

**Current State**: TodoWrite for tracking, no formal validation

**Spec-Kit Pattern**: Checklist files with specific validation items

**Action**:
- Add `.plan/checklists/` directory
- Create validation checklists:
  - `pre-commit-checklist.md`
  - `pr-readiness-checklist.md`
  - `skill-quality-checklist.md`
- Reference from skills/workflows

### Alignment with Our Goals

| Our Goal | Spec-Kit Pattern | How to Apply |
|----------|------------------|--------------|
| PM-mode workflow | Slash commands workflow | Add workflow commands to skills |
| Minimize tokens | Template constraints | Use templates to focus LLM output |
| On-demand skills | Slash commands | Already aligned - keep current approach |
| Self-evaluation | Checklist validation | Add formal checklists |
| Direct scripts | Bash/PS automation | Add scripts/ directory |
| Learning system | (gap in spec-kit) | Keep our `.ai-knowledge/` approach (unique advantage) |
| Spin detection | (not in spec-kit) | Keep our approach (unique advantage) |

## Technical Architecture Comparison

### Spec-Kit Architecture
```
User → Slash Command → Script → Template → LLM → Output → Validation
                                    ↓
                              Constitution (gates)
```

### Our Architecture (Optimized AI)
```
User → Task → Skill → Script/Direct → LLM → Self-Eval → Commit
                ↓           ↓
          Principles   .ai-knowledge (learning)
                            ↓
                      Pattern Recognition
```

### Key Differences

1. **Spec-Kit**: Formal, phase-based (constitution → spec → plan → tasks)
2. **Optimized AI**: Flexible, learning-based (task → skill → execute → learn)

3. **Spec-Kit**: No learning system
4. **Optimized AI**: Continuous learning with pattern recognition

5. **Spec-Kit**: Heavy templates
6. **Optimized AI**: Minimal, validated templates

## Implementation Priority

Based on our current phase (foundation):

### Phase 0 (Now)
- ✅ **Research complete** - This document

### Phase 1 (Next)
- Create minimal templates for common workflows
- Add scripts/ directory with core automation
- Formalize CONSTITUTION.md from principles/

### Phase 2 (Later)
- Add validation checklists
- Implement command aliases in skills
- Create skill-testing framework

### Phase 3 (Future)
- Advanced workflow automation
- Cross-skill coordination
- Pattern promotion from learnings

## Experiments to Run

Based on our "VALIDATE" principle:

### Exp-001: Template vs No-Template
**Hypothesis**: "Minimal templates reduce tokens 20% without quality loss"

**Design**:
- Control: No template, just task description
- Treatment: Minimal template (50 lines)
- Scenarios: 10 common tasks
- Metrics: Tokens, quality, completion time

### Exp-002: Checklist Impact
**Hypothesis**: "Checklists reduce errors by 40%"

**Design**:
- Control: No checklist validation
- Treatment: Checklist before commit
- Scenarios: 20 commits
- Metrics: Error rate, fix cycles, quality

### Exp-003: Script vs Manual
**Hypothesis**: "Scripts reduce setup time 60%"

**Design**:
- Control: Manual workflow steps
- Treatment: Automated script
- Scenarios: New feature creation, research setup
- Metrics: Time, error rate, consistency

## Meta-Patterns Identified

### Pattern 1: Progressive Formalization
- Start informal (user describes feature)
- Add structure iteratively (spec → plan → tasks)
- Each phase adds constraints
- **Application**: Our workflow could benefit from similar progression

### Pattern 2: Template as Prompt
- Templates aren't just structure - they're sophisticated prompts
- Force LLM to think systematically
- Prevent common mistakes through explicit guidance
- **Application**: Use in our skills to constrain LLM behavior

### Pattern 3: Validation Gates
- Don't trust LLM output without validation
- Build validation into the workflow
- Make validation automatic (checklists)
- **Application**: Add to our self-evaluation loop

### Pattern 4: Separation of Concerns
- What (spec) ≠ How (plan) ≠ Steps (tasks)
- Each artifact has single responsibility
- Changes cascade appropriately
- **Application**: Separate `.plan/` (what) from implementation

## Related Research

- [Claude Code Documentation](https://docs.claude.com/claude-code) - Our primary AI agent
- [MCP Protocol](https://modelcontextprotocol.io) - Context protocol (less relevant after ADR-003)
- [Cursor Documentation](https://cursor.sh/docs) - Alternative AI agent

## Questions for Future Investigation

1. **Scalability**: How does spec-kit handle large codebases? (100+ features)
2. **Team Collaboration**: How do multiple users coordinate specs?
3. **Version Migration**: How to upgrade old specs when constitution changes?
4. **Performance**: Token usage in practice vs our minimalist approach?
5. **Learning**: Could we add learning system on top of spec-kit patterns?

## Conclusion

Spec-Kit provides valuable patterns for:
- ✅ Slash commands → Apply as enhanced skills
- ✅ Template constraints → Add minimal templates
- ✅ Constitution → Formalize our principles
- ✅ Script automation → Add scripts/ directory
- ✅ Validation checklists → Add to workflows

**Unique advantages we maintain**:
- 🌟 Learning system (`.ai-knowledge/`)
- 🌟 Spin detection
- 🌟 Lighter-weight approach
- 🌟 Pattern recognition from past tasks

**Recommendation**: Adopt their structural patterns (templates, checklists, scripts) while maintaining our learning-based approach as a differentiator.

## Next Steps

1. Create comprehensive deep-dive documents (linked above)
2. Start Phase 1 implementation (templates, scripts, constitution)
3. Run validation experiments
4. Update roadmap based on learnings

---

**Research Completed**: 2025-11-03
**Researcher**: Claude (Optimized AI)
**Repository**: https://github.com/github/spec-kit
**Commit Analyzed**: e6d6f3c (latest at time of research)
