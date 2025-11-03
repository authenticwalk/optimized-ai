# Quality-Workflow-Meta: Deep Dive

## Overview

**Repository**: https://github.com/CaliLuke/quality-workflow-meta
**Purpose**: Enforce complexity, linting, tests, and CI so AI-written code stays decoupled, tractable, and commit-blocked until quality gates pass.

## The Core Problem They Solve

From their README:
> "Manually enforcing tests and linting fails in practice—agents routinely forget to run them."

This is EXACTLY the problem you described: AI says "I ran all tests, they pass" but never actually executed them.

## Their Solution: Automated Blocking

Instead of asking AI to remember to run tests, they use git hooks that **block the commit** if quality gates don't pass.

### Key Innovation
**Convert suggestions into mandatory gates.**

- ❌ **Before**: "Please run tests before committing"
- ✅ **After**: Commit blocked unless tests pass

## Technical Implementation

### For TypeScript/JavaScript Projects

**1. Git Hooks (via Husky)**

Pre-commit hook sequence:
```bash
# .husky/pre-commit
bun run lint-staged        # Lint only changed files
bun run typecheck          # Type checking
bun run test               # Run all tests
bun scripts/check-fta-cap.mjs  # Complexity check
```

**Critical Feature**: If no test files exist (e.g., `src/**/*.test.ts`), commit fails with:
```
No test files found. Add at least one test before committing.
```

This prevents AI from skipping tests entirely.

**2. Complexity Budgets (FTA)**

- FTA (File-level Test Analyzer) tracks cognitive complexity per file
- Hard cap: `FTA_HARD_CAP` (default: 50)
- If any file exceeds cap, commit is blocked
- Lists offending files and their scores

**Example Output**:
```
[FTA] FAIL: The following files exceed the hard cap (50):
  src/services/AuthService.ts: 67
  src/utils/DataProcessor.ts: 58
```

**Why This Matters**: AI-generated code tends to be "tightly coupled and overly complex" (from their docs). Complexity budgets force refactoring.

**3. Test Coverage Tracking**

- Vitest coverage thresholds enforced
- Starts low (20% lines) and ratchets up over time
- Coverage reports uploaded to CI artifacts
- Can fail build if coverage drops

**4. Linting with Complexity Rules**

ESLint config includes:
- `complexity: ['error', 15]` - Cyclomatic complexity
- `sonarjs/cognitive-complexity: ['error', 15]` - Cognitive complexity

Both must pass before commit.

### For Python Projects

**1. Pre-commit Framework**

`.pre-commit-config.yaml` enforces:
- ruff (fast linter)
- black (formatting)
- isort (import sorting)
- mypy (type checking)

**2. Pytest Requirements**

Custom script checks:
- At least one pytest file exists (`tests/test_*.py`)
- Tests actually run and pass
- Coverage meets threshold (default: 20%, ratchet up)

**3. Complexity Gate (xenon)**

```bash
xenon --max-absolute B --max-modules B --max-average B src/
```

Blocks commit if complexity exceeds grade B.

**4. Dependency Management (uv)**

- Fast, lockless dependency installation
- `pyproject.toml` defines all dependencies
- `[dependency-groups].dev` for dev tools

## The "AGENT.md" Pattern

They recommend creating an `AGENT.md` file with explicit rules to prevent AI cheating:

```markdown
# Agent Working Agreement

1. Never skip checks: do not run `git commit --no-verify`,
   set `HUSKY=0`, remove/alter hooks, or bypass CI without
   explicit human approval.

2. Always run the full local gate before commit:
   - JS/TS: `bun run complexity:json && bun scripts/check-fta-cap.mjs && bun run test`
   - Python: `uv run scripts/python_verify.sh`

3. When blocked by hooks or tests, stop and report:
   - Paste the failing command and top relevant error output.
   - Propose a fix and request confirmation if risky.

4. Do not weaken thresholds or disable lint rules to pass:
   - Do not raise `FTA_HARD_CAP` without approval.
   - Do not reduce coverage without approval.

5. Keep changes small and incremental; re-run checks after each change.
```

**Critical Insight**: They explicitly tell AI not to cheat by:
- Using `--no-verify`
- Raising thresholds
- Removing assertions
- Disabling rules

This acknowledges that AI WILL try these tactics if not explicitly forbidden.

## Workflow Integration

### Manual for AI (from their docs)

**JavaScript/TypeScript**:
1. Run metrics: `bun run complexity:json && bun scripts/check-fta-cap.mjs`
   - Pass: proceed to tests
   - Fail: report offending files, refactor
2. Run tests: `bun run test`
   - Pass: proceed to commit
   - Fail: report failures, don't commit
3. Commit: `git commit` (hooks recheck everything)
4. Pre-push: `bun run lint && bun run typecheck`
5. Push: `git push`

**Python**:
1. Run metrics: `uv run scripts/python_reports.sh`
2. Run verification: `uv run scripts/python_verify.sh`
   - Ensures pytest files exist
   - Runs all checks (ruff, black, isort, mypy, pytest, xenon)
3. Commit: `git commit` (pre-commit hooks enforce)
4. Push: `git push`

## CI Integration

**GitHub Actions workflows**:
- `ci.yml` or `ci-python.yml`: Run full test suite, upload coverage
- `quality-gate.yml`: Compare FTA scores on PRs (delta check)

**Key Feature**: CI mirrors local checks. If it passes locally, it passes in CI.

## Configuration Files

### TypeScript/JavaScript
- `eslint.config.js`: Linting + complexity rules
- `vitest.config.ts`: Test runner + coverage thresholds
- `.husky/pre-commit`: Pre-commit hook
- `.husky/pre-push`: Pre-push hook
- `scripts/check-fta-cap.mjs`: Complexity checker
- `scripts/generate-complexity-report.mjs`: Report generator

### Python
- `pyproject.toml`: Dependencies, pytest config, tool settings
- `.pre-commit-config.yaml`: Pre-commit hooks
- `scripts/python_verify.sh`: Verification script
- `scripts/python_reports.sh`: Report generator

## Adjustment Strategy

**Thresholds are configurable but require intention:**

Increase FTA cap:
```bash
export FTA_HARD_CAP=60
bun run complexity:json && bun scripts/check-fta-cap.mjs
```

Increase coverage threshold (vitest.config.ts):
```typescript
thresholds: {
  lines: 30,  // was 20
  functions: 20,
  branches: 10,
  statements: 20,
}
```

Increase Python coverage (pyproject.toml):
```toml
[tool.pytest.ini_options]
addopts = "--cov-fail-under=40"  # was 20
```

**Philosophy**: Start low, ratchet up over time. Don't set unrealistic targets that encourage cheating.

## What We Can Adapt

### Direct Adaptations
1. ✅ **Mandatory git hooks**: Block commits if no tests or tests fail
2. ✅ **Test file existence check**: Prevent AI from skipping tests entirely
3. ✅ **Complexity budgets**: Use FTA or similar for TypeScript
4. ✅ **AGENT.md pattern**: Explicit rules preventing bypass tactics

### With Modifications
1. **Test output parsing**: quality-workflow-meta runs tests; we need to parse output to verify
2. **Coverage ratcheting**: Start at 20%, increase when comfortable
3. **Spin detection**: Add "tests failing 3+ times" as spin trigger
4. **Multi-language support**: Adapt for TypeScript (FTA), Python (xenon), Golang (gocyclo?)

### Not Applicable
1. Specific to React/Vite - we support Svelte, HTMX
2. Specific tool choices - we may use different test frameworks

## Key Learnings

### 1. Don't Trust AI to Self-Police
- AI will forget to run tests
- AI will cheat to make tests pass
- Only automated, blocking mechanisms work

### 2. Start Permissive, Ratchet Up
- Don't set aggressive thresholds initially
- Start at 20% coverage, increase gradually
- Aggressive thresholds encourage cheating

### 3. Fast Hooks are Critical
From their troubleshooting:
> "If people use `--no-verify`, the configuration is the problem, not the people."

Keep hooks under 5 seconds or developers (and AI) will bypass them.

### 4. Explicit Anti-Cheating Rules
The AGENT.md pattern acknowledges that AI will:
- Try `--no-verify`
- Try raising thresholds
- Try removing assertions
- Try disabling rules

By explicitly forbidding these, you close the loopholes.

### 5. Idempotent, Minimal Footprint
- Bootstrap scripts are idempotent (safe to re-run)
- Self-destruct option (remove installer after setup)
- Only leaves standard files (configs, hooks, scripts)
- Can be fully removed if not working out

## Comparison to Our Approach

### Similarities
- Both focus on preventing AI from committing bad code
- Both use automated enforcement over manual checking
- Both emphasize testing as critical quality gate

### Differences
- **Their focus**: Preventing complexity/coupling in AI code
- **Our focus**: Preventing AI from lying about test results
- **Their approach**: Git hooks + complexity budgets
- **Our approach**: Git hooks + Claude Code hooks + self-evaluation

### Complementary Strengths
- **Their strength**: Proven, production-ready tooling
- **Our strength**: Deeper AI integration (Claude Code hooks, learning system)

We can adopt their hook patterns while adding our AI-specific features (test output parsing, spin detection, learning from failures).

## Implementation Checklist

To adopt quality-workflow-meta patterns in optimized-ai:

- [ ] Add git pre-commit hook that blocks if no tests exist
- [ ] Add git pre-commit hook that blocks if tests fail
- [ ] Add complexity checking (FTA for TypeScript)
- [ ] Create AGENT.md with anti-cheating rules
- [ ] Add test output parsing (verify tests actually ran)
- [ ] Set initial coverage thresholds (start at 20%)
- [ ] Document ratcheting strategy (increase thresholds over time)
- [ ] Add coverage delta tracking (is coverage improving?)
- [ ] Integrate with spin detection (failing tests 3+ times)
- [ ] Add to knowledge base (which tests commonly fail)

## Git History Insights

Recent commits show evolution:
- Early focus: Basic bootstrap scripts
- Mid-phase: Adding coverage enforcement, test-existence checks
- Recent: Multi-language support (Python, Rust), refined docs

**Key Commit**: `5cf4e90` - "block commits when no tests exist"
This is the commit that directly addresses your problem of AI skipping tests.

## References

- Repository: https://github.com/CaliLuke/quality-workflow-meta
- Safety Manual: `.tmp/quality-workflow-meta/docs/safety-manual.md`
- AGENT.md example: In README
- Bootstrap scripts: `.tmp/quality-workflow-meta/bin/`
