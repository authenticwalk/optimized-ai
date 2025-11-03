# Test Verification Strategies

## Core Principle: Trust But Verify

**Don't trust AI claims about test results. Parse and verify actual output.**

## Verification Layers

### Layer 1: Test Files Exist

**Check**: Do test files physically exist?

```bash
#!/bin/bash
# scripts/verify-tests-exist.sh

TEST_PATTERN="src/**/*.test.ts"
TEST_COUNT=$(find src -name "*.test.ts" -o -name "*.spec.ts" | wc -l)

if [ "$TEST_COUNT" -eq 0 ]; then
  echo "ERROR: No test files found matching $TEST_PATTERN"
  echo "Required: At least 1 test file"
  exit 1
fi

echo "✓ Found $TEST_COUNT test file(s)"
exit 0
```

### Layer 2: Tests Actually Ran

**Check**: Did the test runner execute?

```typescript
// scripts/verify-tests-ran.ts
function verifyTestsRan(output: string): VerificationResult {
  // Pattern: "Tests: 3 passed, 3 total" (Jest/Vitest)
  const match = /Tests?:\s+(\d+)\s+passed,?\s+(\d+)\s+total/i.exec(output)

  if (!match) {
    return {
      success: false,
      error: "No test execution found in output"
    }
  }

  const passed = parseInt(match[1])
  const total = parseInt(match[2])

  return {
    success: true,
    passed,
    total,
    failed: total - passed
  }
}
```

### Layer 3: All Tests Passed

**Check**: Were there any failures?

```typescript
function verifyAllTestsPassed(output: string): VerificationResult {
  // Look for failure indicators
  const failurePatterns = [
    /(\d+) failed/i,
    /FAIL/i,
    /Error:/i,
    /AssertionError/i
  ]

  for (const pattern of failurePatterns) {
    const match = pattern.exec(output)
    if (match) {
      return {
        success: false,
        error: `Test failures detected: ${match[0]}`
      }
    }
  }

  // Look for success indicator
  if (!/passed/i.test(output)) {
    return {
      success: false,
      error: "No 'passed' indicator found in output"
    }
  }

  return { success: true }
}
```

### Layer 4: Meaningful Tests

**Check**: Do tests have assertions?

```typescript
function verifyTestQuality(testFile: string): QualityReport {
  const content = fs.readFileSync(testFile, 'utf-8')

  // Count tests
  const testCount = (content.match(/\b(test|it)\s*\(/g) || []).length

  // Count assertions
  const assertionPatterns = [
    /\bexpect\(/g,
    /\bassert\./g,
    /\.should\./g
  ]

  let assertionCount = 0
  for (const pattern of assertionPatterns) {
    assertionCount += (content.match(pattern) || []).length
  }

  // Check for low-quality assertions
  const lowQualityPatterns = [
    /expect\(true\)\.toBe\(true\)/g,
    /expect\(.*\)\.toBeDefined\(\)/g,
    /expect\(.*\)\.toBeTruthy\(\)/g
  ]

  let lowQualityCount = 0
  for (const pattern of lowQualityPatterns) {
    lowQualityCount += (content.match(pattern) || []).length
  }

  const ratio = testCount > 0 ? assertionCount / testCount : 0

  return {
    testCount,
    assertionCount,
    assertionRatio: ratio,
    lowQualityAssertions: lowQualityCount,
    quality: ratio >= 2.0 && lowQualityCount === 0 ? 'good' : 'poor'
  }
}
```

### Layer 5: Coverage Didn't Decrease

**Check**: Did coverage go down?

```typescript
function verifyCoverageTrend(): CoverageVerification {
  const current = parseCoverage('coverage/coverage-summary.json')
  const previous = parseCoverage('.coverage-baseline.json')

  const delta = {
    lines: current.lines - previous.lines,
    branches: current.branches - previous.branches,
    functions: current.functions - previous.functions,
    statements: current.statements - previous.statements
  }

  // Allow small decreases (< 1%) but flag larger drops
  const threshold = -1.0
  const decreased = Object.entries(delta).filter(([_, value]) => value < threshold)

  if (decreased.length > 0) {
    return {
      success: false,
      error: `Coverage decreased: ${decreased.map(([k, v]) => `${k}: ${v}%`).join(', ')}`
    }
  }

  return { success: true, delta }
}
```

## Output Parsing Patterns

### Jest/Vitest Output

```
Test Files  1 passed (1)
     Tests  3 passed (3)
  Start at  10:30:45
  Duration  1.23s (transform 200ms, setup 0ms, collect 100ms, tests 500ms)

PASS  src/auth.test.ts
  ✓ should validate email (5ms)
  ✓ should reject invalid password (3ms)
  ✓ should handle edge cases (2ms)
```

**Parse Strategy**:
```typescript
const patterns = {
  summary: /Tests?\s+(\d+)\s+passed/i,
  failed: /Tests?\s+(\d+)\s+failed/i,
  filePass: /PASS\s+(.+\.test\.ts)/g,
  fileFail: /FAIL\s+(.+\.test\.ts)/g,
  testPass: /✓\s+(.+?)\s+\((\d+)ms\)/g,
  testFail: /✗\s+(.+)/g
}
```

### pytest Output

```
============================= test session starts ==============================
collected 5 items

tests/test_auth.py .....                                                 [100%]

============================== 5 passed in 0.50s ===============================
```

**Parse Strategy**:
```python
import re

def parse_pytest_output(output: str) -> TestResult:
    # Look for summary line
    match = re.search(r'(\d+) passed', output)
    passed = int(match.group(1)) if match else 0

    match = re.search(r'(\d+) failed', output)
    failed = int(match.group(1)) if match else 0

    return TestResult(passed=passed, failed=failed, total=passed+failed)
```

### Go test Output

```
=== RUN   TestValidateEmail
--- PASS: TestValidateEmail (0.00s)
=== RUN   TestValidatePassword
--- PASS: TestValidatePassword (0.00s)
PASS
ok      github.com/user/pkg/auth    0.123s
```

**Parse Strategy**:
```go
func parseTestOutput(output string) (*TestResult, error) {
    passCount := strings.Count(output, "--- PASS:")
    failCount := strings.Count(output, "--- FAIL:")

    if !strings.Contains(output, "PASS\nok") && failCount == 0 {
        return nil, fmt.Errorf("test execution not confirmed")
    }

    return &TestResult{
        Passed: passCount,
        Failed: failCount,
        Total:  passCount + failCount,
    }, nil
}
```

## Real-Time Verification

### During Development (Claude Code Hooks)

```json
{
  "hooks": [
    {
      "event": "PostToolUse",
      "tool": "Edit",
      "command": "scripts/verify-and-test.sh"
    }
  ]
}
```

```bash
#!/bin/bash
# scripts/verify-and-test.sh

# Run tests and capture output
npm test 2>&1 | tee .test-output.txt

# Verify tests ran
node scripts/parse-test-output.js .test-output.txt

# Check exit codes
TEST_EXIT=$?
PARSE_EXIT=$?

if [ $TEST_EXIT -ne 0 ] || [ $PARSE_EXIT -ne 0 ]; then
  echo "❌ Tests failed or parsing error"
  exit 1
fi

echo "✅ Tests passed and verified"
exit 0
```

### Before Commit (Git Hooks)

```bash
#!/bin/bash
# .husky/pre-commit

echo "Verifying tests..."

# 1. Tests exist
./scripts/verify-tests-exist.sh || exit 1

# 2. Run tests
npm test 2>&1 | tee .test-output.txt
TEST_EXIT=${PIPESTATUS[0]}

# 3. Parse output
node scripts/parse-test-output.js .test-output.txt || exit 1

# 4. Check if tests passed
if [ $TEST_EXIT -ne 0 ]; then
  echo "❌ Tests failed - commit blocked"
  exit 1
fi

# 5. Check coverage
./scripts/verify-coverage.sh || exit 1

# 6. Check test quality
./scripts/verify-test-quality.sh || exit 1

echo "✅ All verifications passed"
exit 0
```

## Verification Reports

### Create Verification Summary

```typescript
interface VerificationSummary {
  timestamp: string
  testsExist: boolean
  testsRan: boolean
  testsPassed: boolean
  testCount: number
  assertionCount: number
  coverage: {
    lines: number
    branches: number
    functions: number
  }
  complexity: {
    max: number
    exceeded: string[]
  }
  quality: 'excellent' | 'good' | 'poor'
}

function generateVerificationReport(): VerificationSummary {
  return {
    timestamp: new Date().toISOString(),
    testsExist: verifyTestsExist(),
    testsRan: verifyTestsRan(testOutput),
    testsPassed: verifyAllTestsPassed(testOutput),
    testCount: countTests(),
    assertionCount: countAssertions(),
    coverage: parseCoverage(),
    complexity: checkComplexity(),
    quality: calculateQuality()
  }
}

// Save to knowledge base
fs.writeFileSync(
  '.ai-knowledge/verification-reports/latest.json',
  JSON.stringify(generateVerificationReport(), null, 2)
)
```

## Handling AI Claims

### Pattern 1: Explicit Verification Required

```markdown
# In .cursorrules

When you claim tests passed, you MUST:
1. Show the actual test command you ran
2. Show the test runner output
3. Show the summary line (e.g., "Tests: 5 passed, 5 total")
4. Do NOT summarize - paste actual output

Example:
```
$ npm test

PASS  src/auth.test.ts
  ✓ validates email format (3ms)
  ✓ rejects invalid passwords (2ms)

Tests: 2 passed, 2 total
```
```

### Pattern 2: Automatic Verification

```typescript
// After AI responds with test results
async function verifyAITestClaim(aiResponse: string): Promise<void> {
  // Check if AI claimed tests passed
  if (/tests? passed|all passing/i.test(aiResponse)) {
    // Extract test output (if provided)
    const output = extractCodeBlock(aiResponse, 'bash')

    if (!output) {
      throw new Error("AI claimed tests passed but provided no output")
    }

    // Verify output is legitimate
    const verification = verifyTestOutput(output)

    if (!verification.success) {
      throw new Error(`Test claim verification failed: ${verification.error}`)
    }

    // Record in knowledge base
    recordVerification(verification)
  }
}
```

### Pattern 3: Trust Score

```typescript
interface TrustScore {
  testClaimAccuracy: number  // 0-1, how often AI's claims match reality
  verificationPasses: number
  verificationFailures: number
  lastUpdated: string
}

function updateTrustScore(claimed: boolean, actual: boolean): void {
  const score = loadTrustScore()

  if (claimed === actual) {
    score.verificationPasses++
  } else {
    score.verificationFailures++
  }

  const total = score.verificationPasses + score.verificationFailures
  score.testClaimAccuracy = score.verificationPasses / total

  saveTrustScore(score)

  // If trust score drops below threshold, enable strict mode
  if (score.testClaimAccuracy < 0.8) {
    enableStrictVerificationMode()
  }
}
```

## Integration with Self-Evaluation

### Enhanced Self-Evaluation Loop

**Before** (from SPEC):
```
1. Run all tests (`ide.runTests()`)
2. Check linter (`ide.getLinterErrors()`)
```

**After** (enhanced with verification):
```typescript
async function selfEvaluate(): Promise<EvaluationResult> {
  // 1. Verify tests exist
  const testsExist = await verifyTestsExist()
  if (!testsExist.success) {
    return { passed: false, reason: testsExist.error }
  }

  // 2. Run tests and capture output
  const testOutput = await runTests({ captureOutput: true })

  // 3. Parse and verify output
  const testVerification = verifyTestOutput(testOutput.stdout)
  if (!testVerification.success) {
    return { passed: false, reason: testVerification.error }
  }

  // 4. Check test quality
  const quality = await verifyTestQuality()
  if (quality.assertionRatio < 1.5) {
    return { passed: false, reason: "Tests lack sufficient assertions" }
  }

  // 5. Check coverage
  const coverage = await verifyCoverage()
  if (coverage.decreased) {
    return { passed: false, reason: "Coverage decreased" }
  }

  // 6. Check linter
  const lintErrors = await getLinterErrors()
  if (lintErrors.length > 0) {
    return { passed: false, reason: `${lintErrors.length} lint errors` }
  }

  // 7. Check complexity
  const complexity = await checkComplexity()
  if (complexity.exceeded.length > 0) {
    return { passed: false, reason: "Complexity threshold exceeded" }
  }

  return { passed: true }
}
```

## Success Criteria

Verification is effective when:
- ✅ Zero false "tests passed" claims accepted
- ✅ All test executions have verified, parsed output
- ✅ Coverage trends are tracked and validated
- ✅ Test quality metrics are enforced
- ✅ AI cannot bypass verification without detection

## References

- Test output parsing examples
- Coverage verification patterns
- Quality metric definitions
- Trust score algorithms
