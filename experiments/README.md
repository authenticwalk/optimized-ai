# Experiments Directory

This directory contains all experimental validation work for the optimized-ai project.

## Philosophy

**Every optimization must be empirically validated.** No theoretical improvements, no "should work" assumptions. Only proven, measured, reproducible improvements based on rigorous experimental validation.

See [Principle 3: VALIDATE](../.plan/initial-design/principles/3-validate.md) for full framework.

## File Structure

```
experiments/
├── README.md              # This file
├── EXPERIMENTS-LOG.md     # Chronological record of all experiments
├── LEARNINGS.md           # Distilled insights and validated optimizations
├── scenarios/             # Test scenario definitions
├── results/               # Artifacts from experiment runs
│   └── exp-XXX/          # Per-experiment results
│       ├── control/       # Control group artifacts
│       └── treatment/     # Treatment group artifacts
├── runner/                # CLI tool to run experiments
├── validation/            # Automated validation tools
└── ab-testing/            # Statistical comparison tools
```

## Quick Start

### Running an Experiment

1. **Design**: Define hypothesis and experimental design
2. **Setup**: Create control and treatment configurations
3. **Run**: Execute experiments with runner tool
4. **Validate**: Check results with validation engine
5. **Analyze**: Compare control vs treatment statistically
6. **Document**: Append to EXPERIMENTS-LOG.md using template
7. **Decide**: ADOPT, REJECT, or REFINE based on data

### Documentation Flow

```
Experiment → EXPERIMENTS-LOG.md (detailed) → LEARNINGS.md (summary)
```

**EXPERIMENTS-LOG.md**: Full details of every experiment (never delete)  
**LEARNINGS.md**: Quick reference of validated optimizations and insights

## Key Files

### EXPERIMENTS-LOG.md

Chronological record of ALL experiments. Each experiment is appended using the markdown template provided in the file. This is the source of truth for experimental data.

**Never delete experiments** - failures are valuable data!

### LEARNINGS.md

Distilled insights from experiments. This is a living document that summarizes:
- Validated optimizations (ADOPTED)
- Failed approaches (REJECTED)
- Work in progress (REFINING)
- Cross-cutting patterns
- Anti-patterns to avoid

Update this after completing each experiment.

## Experiment Template

See the template in [EXPERIMENTS-LOG.md](./EXPERIMENTS-LOG.md) or [3-validate.md](../.plan/initial-design/principles/3-validate.md#experiment-template).

Each experiment includes:
- **Hypothesis**: Specific, testable prediction
- **Experimental Design**: Control vs treatment, scenarios, metrics
- **Results**: Raw data from all runs
- **Analysis**: Statistical comparison
- **Conclusion**: Decision (ADOPT/REJECT/REFINE) with rationale
- **Learnings**: Insights for future work

## Metrics Tracked

### Efficiency
- Time to completion
- Token usage
- Tool call count
- File edit count
- Repetitive edits

### Quality
- Tests pass
- Linter clean
- Requirements met
- Code complexity
- Security issues

### Behavior
- Spin detected
- Errors repeated
- Pattern compliance
- Manual intervention needed

### Learning
- Pattern reuse
- Failure avoidance
- Improvement over time

## Decision Criteria

**ADOPT** - Statistically significant improvement, no quality degradation, reproducible  
**REJECT** - No improvement, quality degradation, or inconsistent  
**REFINE** - Shows promise but needs more work

## Tools (To Be Built)

1. **Runner** - CLI to execute Claude Code with prompts, capture metrics
2. **Validator** - Automated code quality checking
3. **Analyzer** - Statistical comparison of control vs treatment
4. **Reporter** - Generate markdown reports from results

## Contributing

When running experiments:
1. Follow the template exactly
2. Run sufficient iterations (minimum 5, preferably 10)
3. Document everything (including failures)
4. Update both EXPERIMENTS-LOG.md and LEARNINGS.md
5. Save all artifacts in results/ directory
6. Include statistical analysis

## Related Documentation

- [Principle 3: VALIDATE](../.plan/initial-design/principles/3-validate.md) - Full validation framework
- [PHASE-0-PLAN.md](../.plan/initial-design/PHASE-0-PLAN.md) - Implementation plan
- [EXPERIMENTS-LOG.md](./EXPERIMENTS-LOG.md) - All experiment records
- [LEARNINGS.md](./LEARNINGS.md) - Summary insights

