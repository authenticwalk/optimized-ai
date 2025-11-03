# Learning: Testing Standards

## Discovery Context

**Source File:** `.tmp/pheromind/goodtestsguide.md`
**Context:** Comprehensive testing quality guide based on FIRST principles and the testing pyramid.
**Relevance:** Supports **Principle 3: VALIDATE** and **ADR-006: Integration Tests Over Unit Tests** from our DECISIONS.md.

## The Core Insight

**Good tests follow FIRST principles and a balanced pyramid distribution.**

**FIRST Principles:**
- **Fast:** Unit tests in milliseconds, integration tests in seconds-minutes
- **Independent:** Standalone execution without order dependencies
- **Repeatable:** Consistent results across environments (eliminate flaky tests)
- **Self-Validating:** Unambiguous pass/fail with proper assertions
- **Timely:** Written with or before code (shift-left approach)

**Testing Pyramid Distribution:**
- **Unit Tests: 70-80%** - Fast, isolated, numerous
- **Integration Tests: 15-20%** - Component interactions, data flow
- **E2E Tests: 5-10%** - Critical user journeys only

**Coverage Target:** 80-90% for critical paths (NOT 100%)

## Deep Analysis

### Why FIRST Principles Matter

**Fast:**
- Enables frequent test runs without slowing development
- Unit tests should run in milliseconds to support TDD
- Slow tests get skipped, defeating their purpose

**Independent:**
- Tests can run in any order
- Parallel execution possible
- One test failure doesn't cascade
- Easier to debug isolated failures

**Repeatable:**
- Eliminates "works on my machine" problems
- Flaky tests undermine confidence
- Deterministic test data critical
- Environment consistency required

**Self-Validating:**
- Automated pass/fail determination
- No human interpretation needed
- Clear assertions with meaningful messages
- Proper test output formatting

**Timely:**
- Shift-left testing catches bugs early
- Tests written with code maintain coverage
- Prevents technical debt accumulation
- Cheaper to fix bugs early

### Testing Pyramid Rationale

**Why 70-80% Unit Tests?**
- Cheapest to write and maintain
- Fastest execution (milliseconds)
- Easiest to diagnose failures
- Foundation of confidence
- High coverage of code paths

**Why 15-20% Integration Tests?**
- Validate component interactions
- Test real database/API calls
- Catch integration bugs unit tests miss
- Balance cost vs. value
- Contract testing at boundaries

**Why Only 5-10% E2E Tests?**
- Expensive to write and maintain
- Slow execution (minutes)
- Brittle (UI changes break tests)
- Hard to diagnose failures
- Focus on critical user journeys only

**Anti-Pattern to Avoid:**
- **Ice Cream Cone:** More E2E than unit tests (expensive, slow, brittle)
- **100% Unit Tests:** Misses integration bugs
- **No Unit Tests:** Every bug requires full system test (slow)

### Coverage Philosophy

From Pheromind (aligns with our ADR-006):

> "Target 80-90% code coverage for critical paths while avoiding 100% as a rigid target."

**Why Not 100%?**
- Diminishing returns beyond 80-90%
- Trivial code doesn't need tests (getters/setters)
- Some code is hard to test (timing, UI)
- Better to focus on high-value tests
- Risk-based coverage more effective

**Our ADR-006 Philosophy (Confirmed):**
> "100% coverage is misleading. Focus testing effort on complex logic and edge cases."

**Alignment:** Pheromind validates our existing philosophy!

## Implementation Details

### Unit Test Quality Standards

From `goodtestsguide.md`:

```markdown
**Unit Test Excellence:**
- Isolation: No external dependencies
- Speed: Millisecond execution time
- Naming: Clearly describes functionality
- Data: Deterministic test data
- Coverage: High coverage without targeting 100%
- Structure: AAA pattern (Arrange-Act-Assert)
- Mocking: Proper use of mocks, stubs, spies
- Responsibility: Single responsibility per test
```

**AAA Pattern Example:**

```typescript
// ✅ Good: Clear AAA structure
test('processUser should update account when user is valid', () => {
  // Arrange
  const user = createTestUser({ status: 'active' })
  const mockDB = createMockDatabase()

  // Act
  const result = processUser(user, mockDB)

  // Assert
  expect(result.success).toBe(true)
  expect(mockDB.updateCalled).toBe(true)
  expect(user.lastProcessed).toBeDefined()
})
```

### Integration Test Quality Standards

```markdown
**Integration Test Quality:**
- Dependency Management: Proper test doubles
- Contracts: Verify APIs meet specifications
- Data Flow: Validate information flow between components
- Environment: Realistic test environments mirroring production
- Database: Test actual database interactions
- Backward Compatibility: Ensure new changes don't break existing consumers
```

### E2E Test Quality Standards

```markdown
**E2E Test Requirements:**
- User Scenarios: Simulate real user journeys
- Critical Paths: Prioritize essential business functions
- Stable Selectors: Use robust element identifiers
- Minimal Suite: Keep to essential scenarios (high maintenance cost)
- Cross-Platform: Ensure functionality across browsers/devices
- Clear Data Management: Maintain consistent, isolated test data
```

### Test Data Management

From `goodtestsguide.md`:

```markdown
**Data Quality Standards:**
- Data Masking: Protect sensitive info while maintaining realism
- Synthetic Generation: Create realistic scenarios when production data unavailable
- Version Control: Track and manage test data versions
- Automated Refresh: Keep test data current and relevant
- Environment Consistency: Ensure uniform data across environments
- Subset Sampling: Use representative samples for efficiency
```

## Applicability to Our Project

### Current State (Optimized AI)

**What We Have (ADR-006):**
- Philosophy: Integration tests over unit tests
- Focus on edge cases and complex logic
- Skip tests for what compiler validates
- Browser testing critical for web apps

**What We're Missing:**
- FIRST principles explicitly codified
- Testing pyramid distribution guidance
- Specific coverage targets
- Test quality metrics

### Specific Recommendations

**Recommendation 1: Codify FIRST Principles in Testing Skill**

```markdown
# testing.skill

## FIRST Principles (Always Apply)

Before writing any test, ensure:
- **Fast:** Can it run in <100ms (unit) or <10s (integration)?
- **Independent:** Can it run standalone in any order?
- **Repeatable:** Will it pass consistently across environments?
- **Self-Validating:** Is pass/fail unambiguous?
- **Timely:** Are you writing it with or before the code?

If any answer is "no," refactor the test or code under test.
```

**Recommendation 2: Add Testing Pyramid Guidance**

```markdown
# testing.skill

## Testing Pyramid Distribution

Target distribution for test suites:
- **70-80% Unit Tests:** Fast, isolated, numerous
  - Test individual functions/methods
  - Mock all external dependencies
  - Millisecond execution time
  - High code coverage of critical paths

- **15-20% Integration Tests:** Component interactions
  - Test actual database interactions
  - Validate API contracts
  - Test data flow between components
  - Seconds to minutes execution

- **5-10% E2E Tests:** Critical user journeys ONLY
  - Essential business functions
  - Complete user workflows
  - Cross-browser/platform compatibility
  - Expensive, keep minimal

## Coverage Target: 80-90% of Critical Paths

NOT 100%. Focus on:
- Complex business logic
- Edge cases and error handling
- High-risk areas
- User-critical functionality

Skip testing:
- Trivial getters/setters
- Framework-generated code
- What TypeScript compiler validates
```

**Recommendation 3: Self-Evaluation Checklist Extension**

Add to self-evaluation loop:

```markdown
## Testing Quality Validation

- [ ] FIRST principles satisfied (Fast, Independent, Repeatable, Self-validating, Timely)
- [ ] Testing pyramid distribution maintained (70-80% unit, 15-20% integration, 5-10% E2E)
- [ ] Coverage 80-90% of critical paths (not targeting 100%)
- [ ] Unit tests run in <100ms each
- [ ] Integration tests run in <10s each
- [ ] E2E tests limited to critical user journeys
- [ ] All tests pass
- [ ] No flaky tests (all deterministic)
- [ ] AAA pattern used consistently
- [ ] Clear test naming (describes what's being tested)
```

**Recommendation 4: Test Metrics Tracking**

Store in `.ai-knowledge/metrics.json`:

```json
{
  "testing": {
    "distribution": {
      "unit": 78.5,
      "integration": 16.2,
      "e2e": 5.3
    },
    "coverage": {
      "overall": 87.2,
      "critical": 92.5,
      "target": "80-90%"
    },
    "quality": {
      "firstCompliance": 0.95,
      "flakyTests": 0,
      "avgUnitTestSpeed": "45ms",
      "avgIntegrationSpeed": "4.2s"
    },
    "trend": "improving"
  }
}
```

### Integration with Our Principles

**Principle 3: VALIDATE**
- FIRST principles provide measurable test quality criteria
- Coverage targets set clear validation goals
- Testing pyramid ensures balanced validation strategy

**Principle 4: LEARN**
- Track test metrics over time
- Learn which tests catch bugs
- Identify areas needing more coverage
- Optimize test suite based on data

**ADR-006 (Integration Tests Over Unit Tests)**
- Confirmed: Our philosophy aligns with industry best practices
- Refined: Balance with 70-80% unit, 15-20% integration
- Enhanced: Add FIRST principles for quality

## Experimental Validation

**Experiment: FIRST Principles Impact**

**Hypothesis:** Enforcing FIRST principles reduces flaky tests by 70% and improves test reliability by 40%.

**Design:**
- **Control:** Tests written without explicit FIRST validation
- **Treatment:** Tests validated against FIRST before commit
- **Scenarios:** 20 features requiring tests
- **Runs:** 10 per scenario

**Metrics:**
- Flaky test count (tests that fail intermittently)
- Test reliability (pass rate over 100 runs)
- Test execution speed
- Debugging time for failures
- Developer confidence in tests

**Success Criteria:**
- ≥50% reduction in flaky tests
- ≥30% improvement in reliability
- No significant increase in test writing time
- Higher developer confidence scores

## Related Learnings

- [Code Quality Standards](./learning-code-quality-standards.md) - Testability depends on low complexity
- [Principle 3: VALIDATE](../.plan/initial-design/principles/3-validate.md) - Testing is core validation
- [ADR-006](../.plan/initial-design/DECISIONS.md) - Our existing testing philosophy

## Conclusion

Pheromind's testing standards **validate and enhance** our existing ADR-006 philosophy. Key takeaways:

1. **FIRST principles** provide measurable test quality criteria
2. **Testing pyramid** (70-80% unit, 15-20% integration, 5-10% E2E) balances cost and value
3. **80-90% coverage** target (not 100%) confirmed as optimal
4. **Integration tests priority** aligns with our existing philosophy

**Key Standards to Adopt:**
- Codify FIRST principles in testing skill
- Add testing pyramid guidance
- Set 80-90% coverage target for critical paths
- Track test metrics (distribution, coverage, quality)

**Implementation Priority:** HIGH - Add to Phase 1 (Self-Evaluation Loop)

**Status:** ✅ Ready for experimental validation
