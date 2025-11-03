# Initial Design - Planning Documents

This folder contains all the design and planning documents from the discovery phase.

## 📖 Reading Order

If you're new to this project, read in this order:

### 1. Start Here: Core Philosophy
**[CORE-PRINCIPLES.md](CORE-PRINCIPLES.md)** - Read this first!

The four principles that drive everything:
- **MINIMIZE**: Maximum results, minimum overhead (< 250 line configs)
- **SEPARATE**: Load skills on-demand, use subagents for context isolation
- **VALIDATE**: Prove everything with experiments, no theory
- **LEARN**: System optimizes itself over time

**Time to read**: 15 minutes

---

### 2. How We Validate: Scientific Method
**[EXPERIMENTAL-VALIDATION.md](EXPERIMENTAL-VALIDATION.md)**

The experimental framework for validating every optimization:
- CLI test harness that runs Claude Code with prompts
- A/B testing framework
- Scenario library
- Results database
- Example experiment records

**Key insight**: Every claim must be backed by data.

**Time to read**: 20 minutes

---

### 3. What We're Building: Complete Spec
**[SPEC.md](SPEC.md)**

Full system specification covering:
- Architecture overview
- All components (skills, subagents, MCPs)
- Configuration files
- Workflows
- Success metrics

**Time to read**: 30 minutes

---

### 4. How We Build It: Implementation Plan
**[REVISED-ROADMAP.md](REVISED-ROADMAP.md)**

13-week implementation roadmap:
- **Phase 0**: Experimental framework (MUST GO FIRST)
- **Phase 1**: Minimal core (< 50 lines .cursorrules)
- **Phase 2**: Skill architecture (load on-demand)
- **Phase 3**: Subagent system (context isolation)
- **Phases 4-9**: Core features + advanced capabilities
- **Phase 10**: Production polish

Each phase includes validation experiments.

**Time to read**: 25 minutes

---

### 5. Why These Choices: Decision Records
**[DECISIONS.md](DECISIONS.md)**

15 Architectural Decision Records (ADRs):
- Why local knowledge base
- Why git-ignore .plan folder
- Why MCP over custom protocol
- Why no React/Tailwind
- Why TypeScript but avoid config complexity
- Why integration tests over unit tests
- Why never auto-merge to main
- And more...

Each decision includes rationale and consequences.

**Time to read**: 20 minutes

---

### 6. Discovery Process: Interview Summary
**[INTERVIEW-SUMMARY.md](INTERVIEW-SUMMARY.md)**

Complete requirements capture from initial interview:
- Core problem statement
- Pain points with current AI tools
- Technical preferences
- Coding philosophy
- Solution requirements
- Success metrics

**Use this to understand the "why" behind the project.**

**Time to read**: 15 minutes

---

## 🎯 Quick Reference by Topic

### Architecture
- **Overall**: [SPEC.md](SPEC.md)
- **Principles**: [CORE-PRINCIPLES.md](CORE-PRINCIPLES.md) 
- **Decisions**: [DECISIONS.md](DECISIONS.md)

### Implementation
- **Roadmap**: [REVISED-ROADMAP.md](REVISED-ROADMAP.md)
- **Experiments**: [EXPERIMENTAL-VALIDATION.md](EXPERIMENTAL-VALIDATION.md)

### Background
- **Requirements**: [INTERVIEW-SUMMARY.md](INTERVIEW-SUMMARY.md)

---

## 🔑 Key Concepts

### Skills
On-demand loadable instructions for specific contexts:
```
Core: 50 lines (always loaded)
Skill: 100 lines (loaded when needed)
Total: 150 lines vs 500+ lines if all loaded
```

### Subagents
Isolated AI instances with fresh context:
```
Main Agent: Orchestrates
Planner Subagent: Creates plan (fresh context)
Implementer Subagent: Writes code (fresh context)
Reviewer Subagent: Reviews code (fresh context)
```

### Experimental Validation
Scientific method applied to AI optimization:
```
1. Hypothesis: "Spin detection reduces wasted effort by 70%"
2. Design: Control vs treatment, 10 runs each
3. Execute: Run experiments, capture metrics
4. Analyze: Statistical comparison
5. Decide: ADOPT, REJECT, or REFINE based on data
```

---

## 📊 Success Metrics (Target)

By the end of implementation:
- ✅ **60%+ token reduction** vs monolithic approach
- ✅ **40%+ speed improvement** in task completion
- ✅ **Equal or better quality** (tests, linting, requirements)
- ✅ **<5% spin rate** (AI getting stuck)
- ✅ **<2 rounds of PR feedback** needed
- ✅ **All claims backed by experimental data**

---

## 🚀 Implementation Status

**Current Phase**: Phase 0 - Experimental Framework

**Progress**:
- ✅ Requirements gathered
- ✅ Core principles defined
- ✅ Experimental framework designed
- ✅ Roadmap created
- ⏳ Experiment runner (in progress)
- ⏳ Baseline measurements (pending)

**Next Steps**:
1. Build experiment runner CLI
2. Create scenario library
3. Establish baseline measurements
4. Validate we can run A/B tests
5. Begin Phase 1 (Minimal Core)

---

## 🤔 Questions?

### "Why start with experiments instead of features?"
Cannot optimize without measurement. Building features without validation is guessing.

### "Why such small configs (< 250 lines)?"
More instructions = more tokens + more conflicts. Every line must prove its value.

### "Why skills instead of one big .cursorrules?"
Different tasks need different context. Load only what's relevant. Separation of concerns.

### "Why subagents?"
Complex tasks benefit from fresh context. Avoid pollution. Clear handoffs.

### "Why validate everything?"
Too much AI hype is theoretical. We prove everything with data or don't claim it.

### "What if experiments show something doesn't work?"
Perfect! Failed experiments prevent wasted effort. Document and try alternative.

---

## 📝 Document Metadata

- **Created**: 2025-11-03
- **Last Updated**: 2025-11-03
- **Phase**: Initial Design (Pre-Implementation)
- **Status**: Complete, ready for Phase 0 implementation

---

## 🔗 Related Resources

### In This Repo
- [Main README](../../README.md) - Project overview
- [Research folder](../../research/) - Analysis of existing AI optimization patterns

### External References
- [ruvnet/claude-flow](https://github.com/ruvnet/claude-flow) - Inspiration for .plan folder structure
- [Model Context Protocol](https://modelcontextprotocol.io/) - MCP specification

---

## 📄 License

MIT - See [LICENSE](../../LICENSE)

