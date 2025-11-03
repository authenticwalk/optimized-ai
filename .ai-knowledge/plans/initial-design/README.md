# Initial Design

This folder contains the core design for the Optimized AI system.

## Core Documents

### [SPEC.md](./SPEC.md)
Complete system specification - what we're building and how it works.

### [PLAN.md](./PLAN.md)
Phase 0 implementation plan - experimental framework that validates everything we build.

### [DECISIONS.md](./DECISIONS.md)
Architectural decision records (ADRs) explaining key choices and trade-offs.

## Design Principles

See [./principles](./principles/) for individual principle files:

- **[1-minimize.md](./principles/1-minimize.md)** - Maximum results, minimum overhead
- **[2-separate.md](./principles/2-separate.md)** - Context isolation via skills and subagents
- **[3-validate.md](./principles/3-validate.md)** - Experimental validation of everything
- **[4-learn.md](./principles/4-learn.md)** - Continuous optimization over time

These principles shape our CLAUDE.md and coding standards.

## Supporting Folders

- **[./experiments](./experiments/)** - Experimental validation results
- **[./learnings](./learnings/)** - Lessons learned during implementation

## Quick Start

1. Read [SPEC.md](./SPEC.md) to understand what we're building
2. Read [./principles](./principles/) to understand our core philosophy
3. Read [PLAN.md](./PLAN.md) to see the implementation approach
4. Refer to [DECISIONS.md](./DECISIONS.md) when you need context on why we made specific choices
