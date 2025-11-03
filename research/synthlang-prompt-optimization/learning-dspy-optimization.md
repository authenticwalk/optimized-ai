# Learning: DSPy Automatic Prompt Optimization

**Category**: Learning
**Source**: DSPy framework, Stanford NLP research
**Key Innovation**: Programmatic prompts that optimize themselves automatically

---

## Overview

DSPy (Declarative Self-improving Language Programs) represents a paradigm shift in prompt engineering: instead of manually crafting prompts, you **declare what you want** and let the system optimize how to achieve it.

The key insight: **Prompts are programs**. And like code, they should be:
- Modular and composable
- Automatically optimized
- Version controlled
- Systematically improved through data

While DSPy is Python-specific, its **principles are universal** and highly applicable to our Claude Code optimization goals.

---

## Core Concepts

### 1. Programmatic vs Manual Prompts

**Traditional manual approach**:
```python
# Static string prompt
prompt = """
You are an expert programmer.
Analyze the following code and find bugs.
Be thorough and explain your findings clearly.
...
"""

result = llm.complete(prompt + code)
```

**Problems with manual prompts**:
- Hard to version and maintain
- Difficult to optimize systematically
- Changes require manual testing
- Context grows uncontrollably
- No clear separation of concerns

**DSPy programmatic approach**:
```python
# Declarative signature
class AnalyzeCode(dspy.Signature):
    """Analyze code for bugs and provide detailed explanations."""
    code = dspy.InputField()
    bugs = dspy.OutputField(desc="List of bugs found")
    explanations = dspy.OutputField(desc="Detailed explanation of each bug")

# Module that can be optimized
class CodeAnalyzer(dspy.Module):
    def __init__(self):
        super().__init__()
        self.analyze = dspy.ChainOfThought(AnalyzeCode)

    def forward(self, code):
        return self.analyze(code=code)

# Optimize automatically
optimizer = dspy.BootstrapFewShot(metric=accuracy_metric)
optimized_analyzer = optimizer.compile(CodeAnalyzer(), trainset=examples)
```

**Benefits of programmatic prompts**:
- ✅ Modular and composable (like functions)
- ✅ Automatically optimized with data
- ✅ Consistent structure enforced
- ✅ Version control friendly
- ✅ Testable and measurable

### 2. Signatures: Declaring Intent

**Concept**: A signature defines **what** a module does, not **how**.

```python
# Simple signature
class GenerateAnswer(dspy.Signature):
    """Answer questions based on context."""
    question = dspy.InputField()
    context = dspy.InputField()
    answer = dspy.OutputField()

# Rich signature with descriptions
class OptimizePrompt(dspy.Signature):
    """Optimize a prompt for token efficiency while maintaining quality."""
    original_prompt = dspy.InputField(desc="The prompt to optimize")
    quality_metrics = dspy.InputField(desc="Required quality standards")
    optimized_prompt = dspy.OutputField(desc="Token-efficient version")
    reduction_percentage = dspy.OutputField(desc="Percentage of tokens saved")
```

**Application to our project**:

While we can't use DSPy directly in Claude Code, we can adopt the **signature pattern**:

```markdown
<!-- In CLAUDE.md or Skills -->

## Task Signature: Implement Authentication

**Inputs**:
- auth_method: {firebase, supabase, custom}
- requirements: Security requirements
- context: Existing codebase structure

**Outputs**:
- implementation: Working auth code
- tests: Unit and integration tests
- documentation: Setup and usage docs

**Quality Metrics**:
- Security: No vulnerabilities
- Tests: 100% pass rate
- Token efficiency: < 500 tokens for implementation instructions
```

### 3. Modules: Composable Building Blocks

**Concept**: Modules are composable units that can be optimized independently or together.

```python
# Simple module
class RAG(dspy.Module):
    def __init__(self, k=3):
        super().__init__()
        self.retrieve = dspy.Retrieve(k=k)
        self.generate = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)

# Composed pipeline
class ComplexPipeline(dspy.Module):
    def __init__(self):
        super().__init__()
        self.planner = dspy.ChainOfThought(PlanSignature)
        self.implementer = dspy.ChainOfThought(ImplementSignature)
        self.reviewer = dspy.ChainOfThought(ReviewSignature)

    def forward(self, task):
        plan = self.planner(task=task)
        implementation = self.implementer(plan=plan.plan)
        review = self.reviewer(code=implementation.code)
        return review
```

**Application to our project**:

This maps directly to our **Agents system**:

```markdown
## Modular Workflow (DSPy-inspired)

Main Agent (Orchestrator):
├── Planner Agent (Planning signature)
│   Input: Task description
│   Output: Step-by-step plan
│
├── Implementer Agent (Implementation signature)
│   Input: Plan
│   Output: Code implementation
│
└── Reviewer Agent (Review signature)
    Input: Code + Plan
    Output: Review feedback + approval

Each agent = independent module with clear signature
Optimized separately, composed systematically
```

---

## DSPy Optimization Techniques

### 1. BootstrapFewShot

**What it does**: Automatically generates high-quality few-shot examples.

**How it works**:
1. Start with basic prompt (zero-shot)
2. Run on training examples
3. Keep successful attempts as demonstrations
4. Compile demonstrations into optimized prompt
5. Result: Few-shot prompt that performs better

**Example**:
```python
# Before optimization
basic_prompt = "Translate English to French: {text}"

# Training data
trainset = [
    {"english": "Hello", "french": "Bonjour"},
    {"english": "Thank you", "french": "Merci"},
    # ... more examples
]

# Optimize
optimizer = dspy.BootstrapFewShot(metric=exact_match)
optimized = optimizer.compile(TranslateModule(), trainset=trainset)

# After optimization
# Prompt now includes best demonstrations:
# "Translate English to French
#
# Example 1:
# English: Good morning
# French: Bonjour
#
# Example 2:
# English: See you later
# French: À bientôt
#
# Now translate: {text}"
```

**Key insight**: The system **discovered** which examples are most helpful through experimentation, not human curation.

**Application to our project**:

```markdown
## BootstrapFewShot Pattern for Claude Code

Problem: Which examples should we include in Skills?

Current approach: Manually curate examples

DSPy-inspired approach:
1. Create skill with NO examples initially
2. Run skill on 20 test cases
3. Identify which successes had:
   - Clearest input/output
   - Most edge cases covered
   - Best demonstration of pattern
4. Add those successes as examples in skill
5. Re-test to validate improvement

Automation opportunity:
- Script to track skill usage
- Auto-identify successful patterns
- Suggest examples to add to skill
- A/B test with/without examples
```

### 2. MIPRO (Multi-prompt Instruction Optimization)

**What it does**: Optimizes multiple prompts in a pipeline simultaneously.

**Key innovation**: Understands that changing one prompt affects others downstream.

**Example scenario**:
```
Pipeline:
Extract entities → Classify entities → Generate summary

Traditional: Optimize each step independently
Problem: Optimal prompt for step 1 might hurt step 2

MIPRO: Optimize all steps together
Benefit: Global optimum, not local optima
```

**How it works**:
1. Define multi-step pipeline
2. Provide training data (inputs + expected final outputs)
3. MIPRO explores instruction variations for ALL steps
4. Evaluates end-to-end performance
5. Finds instruction combination that maximizes final metric

**Simplified example**:
```python
# Multi-step pipeline
class RAGPipeline(dspy.Module):
    def __init__(self):
        self.retrieve = dspy.Retrieve()
        self.generate = dspy.ChainOfThought("context, question -> answer")

    def forward(self, question):
        docs = self.retrieve(question)
        return self.generate(context=docs, question=question)

# MIPRO optimizes BOTH retrieve and generate instructions together
optimizer = dspy.MIPRO(
    metric=answer_quality,
    num_candidates=10,  # Try 10 instruction variations
    init_temperature=1.0
)

optimized_pipeline = optimizer.compile(
    RAGPipeline(),
    trainset=train_data,
    num_trials=100
)

# Result: Instructions for retrieve and generate are jointly optimized
# Neither is optimal alone, but together they maximize final quality
```

**Application to our project**:

```markdown
## MIPRO Pattern for Agent Workflows

Problem: Optimizing agents independently might hurt overall quality

Example workflow:
Planner → Implementer → Reviewer

Independent optimization:
- Optimize Planner for best plans (might be overly detailed)
- Optimize Implementer for best code (might need different plan format)
- Optimize Reviewer for best feedback (might conflict with implementation style)
Result: Local optima, suboptimal overall

MIPRO-inspired approach:
1. Define success metric for ENTIRE workflow (e.g., "PR approved without changes")
2. Vary agent instructions systematically
3. Measure end-to-end success
4. Find instruction combination that maximizes final metric

Example variations to test:
- Planner A (detailed) + Implementer A (follows closely)
- Planner B (high-level) + Implementer B (fills in details)
- Planner C (numbered steps) + Implementer C (step-by-step)

Measure: Which combination produces best final PR?
Not: Which produces best plan in isolation?
```

### 3. Compiled Prompts

**Concept**: After optimization, DSPy "compiles" the optimized prompt.

**Compilation includes**:
- Optimized instructions
- Best few-shot examples
- Structured format
- Type hints and constraints

**Before compilation (abstract)**:
```python
class SummarizeCode(dspy.Signature):
    """Summarize code functionality."""
    code = dspy.InputField()
    summary = dspy.OutputField()
```

**After compilation (concrete optimized prompt)**:
```
Task: Summarize code functionality

Instructions:
1. Identify the main purpose of the code
2. List key functions and their roles
3. Note any important dependencies
4. Keep summary under 100 words

Example 1:
Code: [example successful case from training]
Summary: [high-quality summary that worked well]

Example 2:
Code: [another successful case]
Summary: [another high-quality summary]

Now summarize this code:
{code}

Output format:
Summary: [your summary here]
```

**Key insight**: The compiled version is **data-driven**, not hand-crafted. It's proven to work on real examples.

**Application to our project**:

```markdown
## Compilation Pattern for Skills

Current: We manually write Skills based on intuition

DSPy-inspired: Skills should be "compiled" from successful interactions

Process:
1. Start with minimal skill (basic instructions only)
2. Track usage: Which tasks use this skill?
3. Analyze successes: What worked well?
4. Extract patterns: What made successes successful?
5. "Compile" skill: Update with proven instructions + examples
6. Version control: Track skill evolution over time

Example "compilation" cycle:
Version 1.0: Basic firebase-auth skill (intuition-based)
After 20 uses: 15 successful, 5 failed
Analysis: Successes all handled session persistence explicitly
Compilation: Add "Always manage session state" instruction
Version 1.1: Updated skill with proven pattern
After 20 more uses: 19 successful, 1 failed
Improvement: 75% → 95% success rate
```

---

## Metrics & Optimization

### Fitness Functions in DSPy

**Concept**: You define success, DSPy optimizes for it.

**Common metrics**:
```python
# Exact match
def exact_match(example, prediction):
    return example.answer == prediction.answer

# F1 score (for overlapping content)
def f1_score(example, prediction):
    # Calculate word overlap
    return f1(example.answer, prediction.answer)

# Custom quality metric
def code_quality(example, prediction):
    score = 0
    if prediction.code_compiles: score += 0.3
    if prediction.tests_pass: score += 0.3
    if prediction.follows_style: score += 0.2
    if prediction.token_efficient: score += 0.2
    return score

# Multi-objective
def balanced_metric(example, prediction):
    quality = quality_score(example, prediction)
    efficiency = 1.0 - (tokens_used / max_tokens)
    return 0.7 * quality + 0.3 * efficiency  # Weight quality higher
```

**Application to our project**:

```markdown
## Fitness Functions for Our Optimization

### Task Success Metric
def task_success(task, result):
    checklist:
    - [ ] Task completed as requested (0.4)
    - [ ] Tests pass (0.2)
    - [ ] No security issues (0.2)
    - [ ] Follows project patterns (0.1)
    - [ ] Token efficient (0.1)

    score = sum(checked_items * weights)
    return score

### CLAUDE.md Instruction Metric
def instruction_effectiveness(instruction, task_results):
    - adherence_rate: How often was instruction followed?
    - success_correlation: Do tasks succeed more when instruction followed?
    - token_cost: How many tokens does instruction consume?

    effectiveness = (
        0.5 * adherence_rate +
        0.4 * success_correlation +
        0.1 * (1 - token_cost_normalized)
    )
    return effectiveness

### Skill Quality Metric
def skill_quality(skill, usage_history):
    - load_accuracy: Does skill load when needed? (0.3)
    - success_rate: Task success when skill used? (0.4)
    - token_efficiency: Tokens used vs task complexity (0.2)
    - clarity: Misunderstandings or confusion? (0.1)

    quality = calculate_weighted_average()
    return quality
```

### Optimization Loop

**DSPy's optimization process**:
```
1. Proposal: Generate candidate prompt variations
2. Evaluation: Test candidates on training data
3. Selection: Keep best performers
4. Iteration: Generate new candidates from best
5. Convergence: Stop when no improvement for N iterations
```

**Application to our project**:

```markdown
## Optimization Loop for CLAUDE.md

Weekly Optimization Cycle:

Monday: Measure baseline
- Run 20 standard tasks
- Measure: success rate, token usage, quality scores
- Document in .ai-knowledge/metrics.json

Tuesday: Generate variations
- Identify 3 instructions with low adherence
- Generate 3 variations of each (9 total)
- Variations: shorter, structured, math notation, etc.

Wednesday-Thursday: Evaluate
- Test each variation on 5 relevant tasks
- Measure same metrics as baseline
- Document results

Friday: Select and update
- Choose best variation for each instruction
- Update CLAUDE.md
- Document changes in version control
- Add notes on why this variation won

Next Monday: Measure new baseline
- Compare to previous week
- Validate improvement
- Iterate

Convergence: Stop optimizing an instruction when:
- 3 consecutive weeks with no improvement
- Success rate > 95%
- Token usage < 50 tokens for that instruction
```

---

## Principles Applicable to Our Project

### 1. Separation of Concerns

**DSPy principle**: Separate data (examples), logic (signatures), and optimization.

```python
# DSPy way
signature = AnalyzeCode  # What to do
module = dspy.ChainOfThought(signature)  # How to do it
optimizer = dspy.BootstrapFewShot(metric)  # How to improve
compiled = optimizer.compile(module, data)  # Optimized version
```

**Our application**:

```markdown
## Separation in Claude Code

What to do: Signatures in CLAUDE.md or Skill frontmatter
---
name: analyze-code
signature:
  input: [code, context]
  output: [bugs, recommendations]
  metric: completeness × accuracy
---

How to do it: Instructions in Skill body
# Code Analysis Instructions
1. Read code carefully
2. Check for common vulnerabilities
3. ...

How to improve: Optimization experiments
experiments/optimize-code-analysis/
├── variations/
│   ├── v1-detailed-instructions.md
│   ├── v2-checklist-format.md
│   └── v3-math-notation.md
├── results/
│   └── 2025-11-10-comparison.json
└── winner.md (selected best variation)

Optimized version: Updated Skill
After testing, v2-checklist-format won
→ Update Skill with checklist approach
→ Document in version control
```

### 2. Data-Driven Improvement

**DSPy principle**: Use actual task data to improve prompts, not intuition.

**Our application**:

```markdown
## Capturing Task Data

After every task, log:
{
  "taskId": "t-2025-11-03-001",
  "description": "Implement Firebase auth",
  "skillsUsed": ["firebase-auth", "testing"],
  "instructionsFollowed": [
    "Check session before login",
    "Handle errors gracefully"
  ],
  "instructionsIgnored": [
    "Log all auth events"  // ← This instruction isn't working!
  ],
  "success": true,
  "quality": 4.5,
  "tokensUsed": 1250,
  "duration": "25 min"
}

Weekly analysis:
- Which instructions are most followed? (keep them)
- Which are most ignored? (rewrite or remove)
- Which correlate with success? (emphasize them)
- Which are token-heavy but low-value? (optimize or remove)

Data-driven decisions:
"Log all auth events" ignored in 15/20 tasks
→ Hypothesis: Instruction unclear or low priority
→ Test: Make more prominent vs remove entirely
→ Result: Tasks succeed fine without it
→ Decision: Remove from CLAUDE.md, save 15 tokens
```

### 3. Systematic Exploration

**DSPy principle**: Explore the space of possible prompts systematically, not randomly.

**Exploration strategies**:
```python
# DSPy explores:
- Different instruction phrasings
- Different example selections
- Different reasoning approaches (Chain of Thought, ReAct, etc.)
- Different module combinations

# Not random guessing
# Systematic variation + measurement
```

**Our application**:

```markdown
## Systematic Optimization Matrix

Dimension 1: Format
- [ ] Prose
- [ ] Bullet list
- [ ] Numbered list
- [ ] Table
- [ ] XML tags
- [ ] Math notation

Dimension 2: Detail Level
- [ ] High (explicit step-by-step)
- [ ] Medium (key points)
- [ ] Low (principles only)

Dimension 3: Examples
- [ ] No examples
- [ ] 1 example
- [ ] 2-3 examples
- [ ] 5+ examples

For important instruction, test combinations:
- Prose + High detail + 2 examples (baseline)
- List + Medium detail + 1 example (variation 1)
- Table + Low detail + 0 examples (variation 2)
- XML + Medium detail + 2 examples (variation 3)

Measure each on 5 tasks
Select best performer
Update CLAUDE.md

This is systematic, not guessing
```

### 4. Composition Over Monoliths

**DSPy principle**: Build complex systems from simple, optimizable modules.

```python
# DSPy way: Composable modules
class SimplePipeline(dspy.Module):
    def __init__(self):
        self.step1 = OptimizableModule1()
        self.step2 = OptimizableModule2()
        self.step3 = OptimizableModule3()

    def forward(self, input):
        x = self.step1(input)
        y = self.step2(x)
        z = self.step3(y)
        return z

# Each module can be optimized independently
# But also jointly (MIPRO)
```

**Our application**:

```markdown
## Composable Skills & Agents

Instead of monolithic CLAUDE.md with everything:

Core CLAUDE.md (immutable principles)
├── MINIMIZE: < 100 lines
├── SEPARATE: Use skills
├── VALIDATE: Run experiments
└── LEARN: Update .ai-knowledge/

Composable Skills (optimizable modules)
├── firebase-auth (authentication)
├── firestore-queries (database)
├── testing-patterns (quality)
├── deployment (operations)
└── ... (add as needed)

Composable Agents (workflow modules)
├── planner (breaks down tasks)
├── implementer (writes code)
├── reviewer (checks quality)
└── learner (updates knowledge)

Task execution = Compose relevant modules
Example: "Add auth" = Core + firebase-auth + testing-patterns
Example: "Deploy" = Core + deployment + testing-patterns

Each module optimizable separately
Overall workflow optimizable jointly (MIPRO-style)
```

---

## Implementing DSPy Principles Without DSPy

**Challenge**: DSPy is Python. We use TypeScript + Claude Code.

**Solution**: Adopt the **patterns and principles**, not the library.

### Pattern 1: Signatures for Clarity

```markdown
## In .claude/skills/firebase-auth/SKILL.md

---
name: firebase-auth
signature:
  purpose: "Implement Firebase authentication flows"
  inputs:
    - auth_type: {email, google, phone}
    - security_level: {standard, enhanced}
    - existing_setup: {boolean}
  outputs:
    - auth_code: "Working authentication implementation"
    - tests: "Unit and integration tests"
    - security_review: "Security checklist completed"
  quality_metrics:
    - tests_pass: required
    - no_vulnerabilities: required
    - follows_firebase_best_practices: required
  optimization_goal: "Minimize tokens while maintaining security"
---

# Firebase Authentication Implementation

[Instructions here]
```

### Pattern 2: Bootstrap Examples from Success

```typescript
// experiments/runner/bootstrap-examples.ts

interface TaskResult {
  skill: string;
  input: any;
  output: any;
  success: boolean;
  quality: number;
}

async function bootstrapExamples(
  skillName: string,
  taskHistory: TaskResult[]
) {
  // 1. Filter to this skill's successes
  const successes = taskHistory
    .filter(t => t.skill === skillName && t.success && t.quality >= 4.0)
    .sort((a, b) => b.quality - a.quality);

  // 2. Select diverse, high-quality examples
  const examples = selectDiverseExamples(successes, maxExamples = 3);

  // 3. Format for skill
  const formattedExamples = examples.map(ex => ({
    input: ex.input,
    output: ex.output,
    notes: `Quality: ${ex.quality}, Tokens: ${ex.tokens}`
  }));

  // 4. Update skill file
  await updateSkillWithExamples(skillName, formattedExamples);

  // 5. A/B test: skill with vs without examples
  const improvement = await abTest(
    skillName,
    testCases,
    withExamples = true,
    withoutExamples = false
  );

  console.log(`Examples improved success rate by ${improvement}%`);
}
```

### Pattern 3: Multi-Prompt Optimization

```typescript
// experiments/runner/optimize-workflow.ts

interface WorkflowStep {
  agent: string;
  instructions: string;
}

interface Workflow {
  steps: WorkflowStep[];
}

async function optimizeWorkflow(
  workflow: Workflow,
  testCases: TestCase[],
  metric: (result: any) => number
) {
  const variations: Workflow[] = [];

  // Generate instruction variations for each step
  for (let i = 0; i < workflow.steps.length; i++) {
    const step = workflow.steps[i];
    const instructionVariations = await generateVariations(step.instructions, count = 3);

    // Create workflow variations
    for (const variation of instructionVariations) {
      const variantWorkflow = { ...workflow };
      variantWorkflow.steps[i] = { ...step, instructions: variation };
      variations.push(variantWorkflow);
    }
  }

  // Test all variations end-to-end
  const results = [];
  for (const variant of variations) {
    const score = await runWorkflow(variant, testCases, metric);
    results.push({ workflow: variant, score });
  }

  // Select best overall workflow
  const best = results.sort((a, b) => b.score - a.score)[0];

  // Update all agent instructions
  for (let i = 0; i < best.workflow.steps.length; i++) {
    await updateAgentInstructions(
      best.workflow.steps[i].agent,
      best.workflow.steps[i].instructions
    );
  }

  return best;
}
```

### Pattern 4: Compilation from Data

```typescript
// .ai-knowledge/skill-compilation.ts

interface SkillUsage {
  skillName: string;
  uses: number;
  successes: number;
  failures: number;
  commonPatterns: string[];
  commonPitfalls: string[];
  bestExamples: Example[];
}

async function compileSkill(usage: SkillUsage) {
  const skill = loadSkill(usage.skillName);

  // Add proven patterns from usage data
  const provenPatterns = usage.commonPatterns
    .filter(p => correlatesWithSuccess(p, usage));

  // Add examples from best uses
  const examples = usage.bestExamples.slice(0, 3);

  // Add warnings from common failures
  const warnings = usage.commonPitfalls
    .map(p => `⚠️ Avoid: ${p}`);

  // Generate optimized skill
  const compiledSkill = {
    ...skill,
    instructions: [
      ...skill.instructions,
      "## Proven Patterns",
      ...provenPatterns,
      "## Examples",
      ...examples.map(formatExample),
      "## Common Pitfalls",
      ...warnings
    ].join('\n\n')
  };

  // Write compiled version
  await writeSkill(`${usage.skillName}-compiled`, compiledSkill);

  // A/B test: original vs compiled
  const improvement = await abTest(
    skill,
    compiledSkill,
    testCases
  );

  if (improvement > 0.1) {  // 10% improvement threshold
    console.log(`Compiled version is ${improvement * 100}% better - adopting`);
    await writeSkill(usage.skillName, compiledSkill);
  }
}
```

---

## Validation Experiments

### Experiment 1: Signature Effectiveness

**Hypothesis**: Adding explicit signatures to Skills improves task success rate by 15%.

**Method**:
1. Select 5 existing Skills
2. Create versions with and without signatures
3. Run 10 tasks per skill (5 with signature, 5 without)
4. Measure success rate and quality scores
5. Compare with t-test

**Success criteria**:
- ✅ Signatures improve success rate by ≥ 15%
- ✅ Statistical significance (p < 0.05)
- ✅ No increase in tokens or confusion

### Experiment 2: Bootstrap Examples

**Hypothesis**: Auto-generated examples from successful tasks improve skill quality by 20%.

**Method**:
1. Select skill with no examples currently
2. Collect 20 successful uses of that skill
3. Bootstrap 3 best examples using selection algorithm
4. A/B test: skill with vs without examples
5. Measure success rate, quality, adherence

**Success criteria**:
- ✅ Examples improve success rate by ≥ 20%
- ✅ No increase in tokens > 100
- ✅ Examples are actually helpful (subjective validation)

### Experiment 3: Multi-Agent Optimization

**Hypothesis**: Jointly optimizing agent instructions improves final PR quality by 25%.

**Method**:
1. Define 3-agent workflow: Planner → Implementer → Reviewer
2. Create 3 instruction variations for each agent (27 combinations)
3. Test each combination on 5 complex tasks
4. Measure final PR quality (does it get approved without changes?)
5. Select best combination

**Success criteria**:
- ✅ Best combination outperforms baseline by ≥ 25%
- ✅ Improvement is consistent across task types
- ✅ Best combination is not just "Planner best" + "Implementer best" (proves joint optimization value)

---

## Action Items

### This Week
- [ ] Read DSPy documentation for deeper understanding
- [ ] Design signature format for our Skills
- [ ] Create task logging system (capture successes/failures)
- [ ] Identify 3 Skills to add signatures to

### Next 2 Weeks
- [ ] Implement bootstrap examples script
- [ ] Run Experiment 1: Signature effectiveness
- [ ] Run Experiment 2: Bootstrap examples
- [ ] Document results in experiments/LEARNINGS.md

### Weeks 3-4
- [ ] Design multi-agent optimization framework
- [ ] Run Experiment 3: Joint optimization
- [ ] Create skill compilation script
- [ ] Test compilation on 2-3 high-usage skills

### Long-term
- [ ] Build automated optimization pipeline
- [ ] Integrate with .ai-knowledge/ for continuous improvement
- [ ] Create dashboard for tracking optimization progress
- [ ] Share learnings with community

---

## Key Takeaways

1. **Prompts are programs** - Treat them like code: modular, testable, optimizable
2. **Declare intent, optimize execution** - Signatures separate what from how
3. **Data beats intuition** - Use actual task results to improve prompts
4. **Compose, don't monolith** - Build from small, optimizable modules
5. **Optimize together, not alone** - Joint optimization finds global optimum

**For our project**:
- ✅ Add signatures to Skills (clarity + optimization target)
- ✅ Bootstrap examples from successful tasks (data-driven)
- ✅ Jointly optimize agent workflows (MIPRO-inspired)
- ✅ Compile skills from usage data (continuous improvement)
- ✅ Measure everything (enable optimization)

DSPy's principles are universal and highly applicable to Claude Code, even though we can't use the library directly.

---

**Document Version**: 1.0
**Last Updated**: 2025-11-03
**Status**: Ready for experimentation
**Related Docs**: README.md, pattern-continuous-optimization.md
