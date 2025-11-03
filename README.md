# Optimized AI

A scientifically-validated AI coding assistant system for Claude Code/Cursor that enables PM-mode workflow: you create tasks, AI implements on branches, self-evaluates, and creates PRs. You review and merge.

**Built on empirical evidence, not theory.**

## 🎯 Core Principles

1. **MINIMIZE**: Maximum results, minimum overhead
   - Tiny .cursorrules (< 100 lines total)
   - Load only what's needed (skills on-demand)
   - Every token justified by experiments

2. **SEPARATE**: Context isolation
   - Skills loaded on-demand (not all at once)
   - Subagents for complex tasks (fresh context)
   - No context pollution

3. **VALIDATE**: Prove everything experimentally
   - Scientific method applied to AI optimization
   - A/B testing for every claim
   - No theoretical improvements - only measured results

4. **LEARN**: Continuous self-optimization
   - System improves over time
   - Removes unused instructions
   - Learns optimal patterns

## 🔬 Scientific Approach

Unlike other AI optimization projects that rely on prompts and hope, this project validates everything through rigorous experimentation:

- CLI-based test harness runs Claude Code with prompts
- Captures and validates all outputs
- A/B testing with statistical analysis
- Every optimization proven with data
- Failed experiments documented (as important as successes)

**See: [EXPERIMENTAL-VALIDATION.md](.plan/initial-design/EXPERIMENTAL-VALIDATION.md)**

## 🚀 Quick Start

```bash
# Phase 0 (Experimental Framework) - In Development
# Phase 1 coming soon
npx @optimized-ai/init
```

## 📋 Design Documentation

Core documents in `.plan/initial-design/`:

- **[CORE-PRINCIPLES.md](.plan/initial-design/CORE-PRINCIPLES.md)** - Architectural principles driving all decisions
- **[EXPERIMENTAL-VALIDATION.md](.plan/initial-design/EXPERIMENTAL-VALIDATION.md)** - Scientific validation framework
- **[REVISED-ROADMAP.md](.plan/initial-design/REVISED-ROADMAP.md)** - 13-week implementation plan
- **[SPEC.md](.plan/initial-design/SPEC.md)** - Complete system specification
- **[INTERVIEW-SUMMARY.md](.plan/initial-design/INTERVIEW-SUMMARY.md)** - Requirements capture
- **[DECISIONS.md](.plan/initial-design/DECISIONS.md)** - Architectural decision records

## 📁 Research

Analysis of existing AI orchestration patterns to inform our approach:

- **[research/quick-reference.md](research/quick-reference.md)** - One-page cheat sheet
- **[research/claude-flow-analysis.md](research/claude-flow-analysis.md)** - Claude-flow patterns
- **[research/README.md](research/README.md)** - Complete research guide

## 🎬 Workflow Vision

**1. You (PM Mode):**
```bash
echo "Add user authentication with Firebase" > .plan/current-task.md
```

**2. AI Takes Over:**
- Checks knowledge base for similar tasks
- Creates implementation plan
- Writes code following learned patterns
- Self-evaluates continuously
- Auto-detects and corrects if spinning

**3. AI Creates PR:**
- Only when all tests pass
- Includes self-evaluation
- Never merges to main

**4. You Review:**
- Comment on PR
- AI reads comments and fixes
- Repeat until satisfied
- You merge when ready

## 🏗️ Current Status

**Phase 0 (Experimental Framework)** - Design Complete, Ready to Implement
- Experiment runner CLI
- Scenario library
- Validation engine
- Baseline measurements

See [.plan/STATUS.md](.plan/STATUS.md) for current progress.

**Key Architecture Change**: Single minimal `.cursorrules` file (< 100 lines) with on-demand skill loading. Cursor auto-includes claude.md, so just one config to optimize!

## 🎯 Tech Stack

**Supported Languages:**
- TypeScript (primary)
- Golang (performance)
- Python (data/scripts)

**Supported Frameworks:**
- Svelte, HTMX, Ionic+Angular
- Firebase, Supabase

**Opinionated Choices:**
- No React (separation of concerns)
- No Tailwind (design/structure separation)
- Minimal boilerplate
- Refactor on reuse, not preemptively

## 📝 License

MIT
