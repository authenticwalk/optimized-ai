# Research Conductor Skill

## Purpose
Conduct **focused, selective research** into external projects to extract **only the specific learnings, patterns, and innovations** that align with our project goals.

**Not a comprehensive analysis** - we cherry-pick what's relevant to our objectives.

## Activation
Use this skill when asked to:
- Research a repository/tool/technology for specific insights
- Extract learnings relevant to our project goals
- Understand how others solved problems we're facing
- Document innovations we might adapt
- Research official documentation + best practices across the web
- Compare different approaches/frameworks for a specific need

## Key Principle: Selective Over Comprehensive

**FIRST: Read our project goals**
- Read `.plan/initial-design/SPEC.md` for what we're building
- Read `.plan/initial-design/principles/*.md` for our core principles
- Read `.plan/initial-design/DECISIONS.md` for context on our choices

**THEN: Be selective in research**
- Their README is enough for general understanding
- Deep-dive ONLY on aspects that align with our goals
- Skip features/patterns not relevant to our objectives

**Example**: If researching a hooks-based learning system:
- ✅ Deep-dive: How hooks intercept and augment behavior
- ✅ Deep-dive: How they store and retrieve learnings
- ✅ Deep-dive: Self-evaluation mechanisms
- ❌ Skip: Their UI implementation
- ❌ Skip: Their deployment strategy
- ❌ Skip: Features unrelated to our goals

## Research Methodology

### Phase 0: Understand Our Goals (ALWAYS DO THIS FIRST)
1. **Read Project Context**
   - Read `.plan/initial-design/SPEC.md` - understand what we're building
   - Read `.plan/initial-design/principles/*.md` - understand our principles
   - Read `.plan/initial-design/DECISIONS.md` - understand our constraints

2. **Identify What to Look For**
   - What problems are we trying to solve?
   - What approaches are we exploring?
   - What patterns would be most valuable?
   - What should we avoid based on our principles?

### Phase 1: Initial Discovery

Choose your research path based on the task:

#### Path A: Repository/Project Research
1. **High-Level Understanding**
   - Clone repository to `.tmp/` for local analysis
   - Read their README for general understanding
   - Identify their core value proposition
   - Note their technology stack

2. **Selective Identification**
   - Which features/patterns align with our goals?
   - Which innovations could we adapt?
   - Which mistakes did they make (via git history)?
   - What should we skip as irrelevant?

#### Path B: Documentation + Best Practices Research
1. **Find Official Documentation**
   - Use WebSearch to find official docs
   - Use WebFetch to extract key information
   - Focus on features/patterns relevant to our goals

2. **Find Community Best Practices**
   - Search for GitHub repos with examples/snippets
   - Search for blog posts, guides, discussions
   - Look for "awesome-{technology}" lists
   - Find framework comparisons if choosing between options

3. **Gather Code Examples**
   - Clone repos with good examples to `.tmp/`
   - Extract relevant code snippets
   - Note what makes them effective

### Phase 2: Selective Deep Analysis
**ONLY for aspects relevant to our goals:**

1. **Cherry-Pick Patterns**
   - Extract ONLY patterns that solve our problems
   - Document ONLY innovations applicable to us
   - Analyze ONLY mistakes we might make too
   - Skip everything else

2. **Focused Deep-Dives**
   - Create one file per relevant learning
   - One file per applicable pattern
   - One file per relevant mistake
   - Complex topics become subfolders

### Phase 3: Synthesis & Documentation
1. **Create Overview (README.md)**
   - Maximum 400 lines
   - High-level summary of all findings
   - Links to individual deep-dive files
   - Key takeaways and recommendations

2. **Create Individual Deep-Dive Files**
   - **One file = One concept/learning/mistake/pattern/innovation**
   - Each file is a complete, thorough exploration of that single item
   - Include code examples, git history, experiments, analysis
   - Link to related concepts where relevant

3. **Create Subfolders for Complex Topics**
   - When a topic needs multiple files to explain, create a subfolder
   - Subfolder has its own README.md with overview
   - Each subfile in the folder explores one aspect
   - Example: `hooks/` folder with `pre-execution.md`, `post-execution.md`, etc.

## Output Structure

**Principle: One File = One Deep Dive**

Each file is a complete deep-dive into ONE specific concept, learning, mistake, or pattern. When a concept itself needs breakdown, it becomes a subfolder with its own README.md and subfiles.

```
/research/{project-name}/
├── README.md                                    # High-level overview (max 400 lines)
├── learning-hooks-for-interception.md          # ONE specific learning
├── learning-sqlite-for-persistence.md          # ANOTHER specific learning
├── learning-context-injection-timing.md        # ANOTHER specific learning
├── mistake-over-aggressive-extraction.md       # ONE specific mistake
├── mistake-missing-error-recovery.md           # ANOTHER specific mistake
├── pattern-hook-based-augmentation.md          # ONE specific pattern
├── innovation-automatic-learning.md            # ONE specific innovation
├── comparison-to-project-goals.md              # How this relates to our project
├── hooks/                                       # Complex topic needing breakdown
│   ├── README.md                               # Overview of hooks
│   ├── pre-execution-hooks.md                  # Deep dive into pre-execution
│   ├── post-execution-hooks.md                 # Deep dive into post-execution
│   └── error-hooks.md                          # Deep dive into error handling
└── sqlite/                                      # Complex topic needing breakdown
    ├── README.md                               # Overview of SQLite usage
    ├── schema-design.md                        # Deep dive into schema
    ├── query-optimization.md                   # Deep dive into queries
    └── why-sqlite-not-alternatives.md          # Deep dive into decision
```

## Workflow

### Step 1: Understand Our Goals
```bash
# ALWAYS START HERE
# Read our project goals to know what to look for
cat .plan/initial-design/SPEC.md
cat .plan/initial-design/principles/*.md
cat .plan/initial-design/DECISIONS.md
```

Ask yourself:
- What problems are we solving?
- What patterns would help us?
- What should we avoid based on our principles?

### Step 2: Setup & Initial Review
```bash
# Create research directory
mkdir -p research/{project-name}

# Clone repository
cd .tmp
git clone {repository-url} {project-name}
cd {project-name}

# Read their README for overview
cat README.md

# Analyze git history for mistakes/improvements
git log --oneline --grep="fix\|bug\|revert\|mistake" > ../../research/{project-name}/mistakes-from-git.txt
git log --oneline --grep="improve\|refactor\|optimize" > ../../research/{project-name}/improvements-from-git.txt
```

### Step 3: Selective Identification
Based on our goals, identify:
- ✅ Which 3-5 aspects align with our goals?
- ✅ Which innovations could we adapt?
- ✅ Which mistakes did they make that we might make?
- ❌ What should we skip as irrelevant?

**Document your selectivity** - note what you're skipping and why.

### Step 4: Focused Deep-Dives
**ONLY for the selected aspects:**
1. Create one file per learning/pattern/mistake/innovation
2. Include git commits showing evolution
3. Extract code examples
4. Document experiments they ran
5. Note how it applies to our project

### Step 5: Application Analysis
For each insight, explicitly connect to our goals:
- Which principle does this support?
- Which problem in our spec does this solve?
- How would we adapt this to our architecture?
- What would we need to change?

## Research Templates

### README.md Template
```markdown
# {Project Name} Research

## What They Built
Brief description of the project and its value proposition.

## Why We're Researching This
Specific alignment with our project goals (reference `.plan/initial-design/SPEC.md`).

**Example:**
> We're building a self-learning AI system. This project uses hooks to capture
> learnings automatically, which aligns with our goal of minimal overhead learning
> (principle 1-minimize.md and 4-learn.md).

## What We Explored
List ONLY the aspects we deep-dived on:
- [How they use hooks for interception](./learning-hooks-for-interception.md)
- [SQLite for learning persistence](./learning-sqlite-for-persistence.md)
- [Automatic self-evaluation](./innovation-self-evaluation.md)

## What We Skipped
Briefly note what we didn't explore and why:
- Their UI implementation (not relevant to our CLI focus)
- Their deployment strategy (different infrastructure)
- Their user management (we use Firebase)

## Key Insights
**Learnings** (one file each):
- [Hooks for Clean Interception](./learning-hooks-for-interception.md)
- [SQLite as Embedded Knowledge Store](./learning-sqlite-for-persistence.md)

**Mistakes to Avoid** (one file each):
- [Over-aggressive Learning Extraction](./mistake-over-aggressive-extraction.md)
- [Context Pollution from Too Many Learnings](./mistake-context-pollution.md)

**Patterns We Can Use** (one file each):
- [Hook-based Augmentation](./pattern-hook-augmentation.md)
- [Pre/Post Execution Pattern](./pattern-pre-post-execution.md)

**Innovations Worth Adapting** (one file each):
- [Automatic Learning Extraction](./innovation-automatic-learning.md)

## Complex Topics
For topics needing multiple files:
- [Hooks Implementation](./hooks/README.md)

## How This Applies to Our Project
**Alignment with our goals:**
- Reference specific files from `.plan/initial-design/`
- Show how learnings support our principles
- Note gaps this research revealed in our plan

**Specific recommendations:**
1. Actionable steps based on research
2. What to implement first
3. What to avoid based on mistakes found
4. Framework/tool recommendations with rationale

## Gaps & Opportunities
Things not in our original plan that this research suggests we should consider:
- Gap 1: Description and recommendation
- Gap 2: Description and recommendation
```

### Individual Deep-Dive Template

Each file explores ONE concept in depth:

#### Learning File Template (`learning-{name}.md`)
```markdown
# Learning: {Specific Learning Title}

## Discovery Context
How and where this learning was discovered (git commit, code analysis, docs).

## The Core Insight
Clear, concise statement of the learning.

## Deep Analysis
Detailed exploration of:
- Why this approach was taken
- What problem it solves
- How it compares to alternatives
- Edge cases and considerations

## Implementation Details
```language
// Relevant code examples showing this learning in practice
```

## Experiments & Validation
If experiments were conducted:
- What was tested
- Results and observations
- Conclusions drawn

## Applicability to Our Project
Specific ways this learning could apply to our project goals.

## Related Learnings
Links to related files that build on this concept.
```

#### Mistake File Template (`mistake-{name}.md`)
```markdown
# Mistake: {Specific Mistake}

## Git History Analysis
- Commit where mistake was introduced
- Commit where it was fixed
- Time between introduction and fix

## The Mistake
Clear description of what went wrong.

## Impact
What problems this caused:
- User impact
- System impact
- Technical debt created

## Root Cause
Deep analysis of why this happened:
- Missing validation
- Wrong assumptions
- Overlooked edge cases

## The Fix
How it was corrected:
```language
// Code showing the fix
```

## Prevention
How to avoid this mistake in future:
- Patterns to follow
- Tests to write
- Documentation to create

## Our Project Application
How we can avoid this mistake in our project.
```

#### Pattern File Template (`pattern-{name}.md`)
```markdown
# Pattern: {Pattern Name}

## Pattern Description
Clear description of the pattern and its intent.

## Problem It Solves
What recurring problem this pattern addresses.

## Implementation
How the pattern is implemented:
```language
// Code example showing pattern in use
```

## Variations Observed
Different ways this pattern was applied:
- Variation 1: Context and implementation
- Variation 2: Context and implementation

## Trade-offs
Advantages and disadvantages of this pattern.

## When to Use
Specific conditions where this pattern is appropriate.

## When to Avoid
Situations where this pattern is not suitable.

## Application to Our Project
Specific recommendations for using this pattern in our project.
```

#### Innovation File Template (`innovation-{name}.md`)
```markdown
# Innovation: {Innovation Name}

## What Makes It Innovative
Why this approach is novel or unique.

## The Traditional Approach
How this problem was typically solved before.

## The New Approach
How this innovation changes the approach:
```language
// Code demonstrating the innovation
```

## Technical Deep Dive
Detailed analysis of:
- Architecture decisions
- Technology choices
- Implementation challenges overcome

## Impact & Results
Measurable improvements or benefits:
- Performance gains
- Developer experience improvements
- Maintenance benefits

## Limitations
Known constraints or trade-offs of this innovation.

## Evolution Potential
How this innovation could be extended or improved.

## Applicability
How we could adapt this innovation for our project.
```

## Example Research Tasks

### Repository Analysis
```bash
# Clone and analyze
git clone https://github.com/example/project .tmp/project
cd .tmp/project

# Analyze structure
find . -type f -name "*.md" | head -20
find . -type f -name "*.js" -o -name "*.ts" | wc -l

# Check commit patterns
git log --format="%s" | grep -i "fix\|improve\|refactor" | head -20

# Identify key files
git log --format="" --name-only | sort | uniq -c | sort -rn | head -20
```

### Pattern Extraction
Look for:
- Hooks usage and implementation
- State management patterns
- Error handling approaches
- Testing strategies
- Configuration management
- Plugin/extension systems

### Innovation Documentation
Document:
- Novel approaches to common problems
- Unique architectural decisions
- Creative use of technologies
- Performance optimizations
- Developer experience improvements

## Special Focus Areas

### For AI/Claude-related Projects
- Prompt engineering patterns
- Context management strategies
- Tool integration approaches
- Memory/learning systems
- Skill/command implementations

### For Developer Tools
- CLI design patterns
- Configuration management
- Plugin architectures
- Error handling and recovery
- User experience optimizations

## Meta-Pattern Recognition

### Identify Recurring Themes
- Solutions that appear across multiple projects
- Common architectural decisions
- Shared problem-solving approaches
- Industry best practices

### Extract Principles
- Design principles used
- Trade-offs made and why
- Constraints and how they shaped decisions
- Values reflected in implementation

## Research Type Examples

### Example 1: Repository Analysis
```
User: Research https://github.com/alirezarezvani/claude-code-tresor and put it in research/claude-code-tresor

Compare it to the goals in .plan/initial-design

Focus on learnings and meta patterns. Create a single max 400 line README.md with individual files exploring each element (scoped to one concept like agents).

For learnings, clone to .tmp/claude-code-tresor and look at git log for improvements/mistakes.
```

**Approach:**
1. Read `.plan/initial-design/` to understand our goals
2. Clone repo and read README
3. Identify 3-5 aspects that align with our goals
4. Deep-dive each aspect in separate files
5. Extract git history for evolution insights
6. Document what we're skipping and why

### Example 2: Documentation + Best Practices Research
```
User: Research Claude MCP and put findings in research/claude/mcp

Find all official documentation, then research best practices from repos/websites.

Stay focused on .plan/initial-design goals but also note things I missed that would really help.

Also research the best framework (TypeScript and Python) for building MCP servers - ideally just write the function and wrap with a decorator.

Research future of MCP in Claude Code vs Skills - when to use each.
```

**Approach:**
1. Read `.plan/initial-design/` to understand our goals
2. WebSearch for official Claude MCP documentation
3. WebFetch key pages from official docs
4. WebSearch for "claude mcp best practices", "claude mcp examples"
5. Search GitHub for repos with MCP implementations
6. Clone 2-3 example repos to `.tmp/`
7. Compare TypeScript frameworks (FastMCP, official SDK, etc.)
8. Create comparison files: `framework-typescript.md`, `framework-python.md`
9. Create analysis: `mcp-vs-skills.md` with pros/cons
10. Extract patterns into individual files

### Example 3: Multi-Topic Research
```
User: Research Claude agents and subagents, put findings in research/claude/subagents

Find official docs + best practices from repos/websites with snippets.

Focus on .plan/initial-design goals but note things I missed that would help.
```

**Approach:**
1. Read `.plan/initial-design/` for context
2. WebSearch official Claude docs on agents/subagents
3. WebSearch "claude agents best practices", "claude subagents patterns"
4. Search GitHub for repos using agents effectively
5. Clone example repos to `.tmp/`
6. Extract specific patterns into files like:
   - `pattern-agent-coordination.md`
   - `pattern-subagent-context-passing.md`
   - `learning-when-to-use-subagents.md`
   - `mistake-too-many-agents.md`
7. Note gaps in our plan that this research reveals

## Quality Checklist

Before completing research:
- [ ] README.md is under 400 lines
- [ ] Connected research to our goals in `.plan/initial-design/`
- [ ] All significant insights have individual deep-dive files
- [ ] Documented what we skipped and why
- [ ] Learnings are actionable
- [ ] Recommendations are specific
- [ ] Code examples are included where relevant
- [ ] Git history insights are documented (if repo research)
- [ ] Best practices from community are captured
- [ ] Framework comparisons are thorough (if applicable)
- [ ] Noted gaps/opportunities in our plan

## Web Research Commands

### Finding Official Documentation
```bash
# Search for official docs
WebSearch: "claude mcp official documentation"
WebSearch: "anthropic mcp documentation"
WebSearch: "claude code agents documentation"

# Fetch specific pages
WebFetch: https://docs.anthropic.com/... "Extract information about MCP servers"
```

### Finding Best Practices
```bash
# Search for patterns and examples
WebSearch: "claude mcp best practices 2024"
WebSearch: "claude mcp examples github"
WebSearch: "awesome-claude-mcp"
WebSearch: "claude agents patterns"

# Search for comparisons
WebSearch: "fastmcp vs mcp-sdk"
WebSearch: "when to use mcp vs skills claude"
```

### Finding Example Code
```bash
# Search GitHub
WebSearch: "site:github.com claude mcp server example"
WebSearch: "site:github.com claude agents pattern"

# Clone promising repos
git clone https://github.com/example/mcp-server .tmp/mcp-example
```

## Usage Example

```
User: Research https://github.com/example/ai-tool and put it in research/ai-tool