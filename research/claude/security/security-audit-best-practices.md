# Security Audit Best Practices

**Research Quality**: ⭐⭐⭐⭐ (Pheromind, quality-workflow-meta, AgentDB patterns)
**Status**: ✅ IMPLEMENTATION-READY

## Pre-Commit Security Checks

### Hook 1: Secret Scanning

**From Pheromind research** (`scan-secrets.js`):

```javascript
#!/usr/bin/env node
// scripts/scan-secrets.js

const fs = require('fs')
const { execSync } = require('child_process')

const SECRET_PATTERNS = [
    {
        name: 'AWS Access Key',
        pattern: /AKIA[0-9A-Z]{16}/g,
        severity: 'critical'
    },
    {
        name: 'API Key',
        pattern: /api[_-]?key\s*[:=]\s*['"][^'"]{20,}['"]/gi,
        severity: 'high'
    },
    {
        name: 'Private Key',
        pattern: /-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----/g,
        severity: 'critical'
    },
    {
        name: 'JWT Token',
        pattern: /eyJ[a-zA-Z0-9-_]+\.eyJ[a-zA-Z0-9-_]+\.[a-zA-Z0-9-_]+/g,
        severity: 'high'
    },
    {
        name: 'Database URL',
        pattern: /(postgres|mysql|mongodb):\/\/[^\s]+/gi,
        severity: 'high'
    },
    {
        name: 'Generic Secret',
        pattern: /(password|passwd|pwd|secret|token)\s*[:=]\s*['"][^'"]{8,}['"]/gi,
        severity: 'medium'
    }
]

function scanFile(filePath) {
    const content = fs.readFileSync(filePath, 'utf8')
    const findings = []

    SECRET_PATTERNS.forEach(({ name, pattern, severity }) => {
        const matches = content.match(pattern)
        if (matches) {
            findings.push({
                file: filePath,
                type: name,
                severity,
                matches: matches.length,
                preview: matches[0].substring(0, 50) + '...'
            })
        }
    })

    return findings
}

function scanStagedFiles() {
    // Get staged files
    const stagedFiles = execSync('git diff --cached --name-only')
        .toString()
        .split('\n')
        .filter(Boolean)
        .filter(f => !f.includes('node_modules'))
        .filter(f => !f.includes('.test.'))
        .filter(f => !f.match(/\.(jpg|png|gif|pdf|lock)$/))

    const allFindings = []

    stagedFiles.forEach(file => {
        if (fs.existsSync(file)) {
            const findings = scanFile(file)
            allFindings.push(...findings)
        }
    })

    return allFindings
}

// Run scan
const findings = scanStagedFiles()

if (findings.length > 0) {
    console.error('❌ SECURITY: Potential secrets found!')
    console.error('')

    findings.forEach(({ file, type, severity, matches, preview }) => {
        console.error(`  ${severity.toUpperCase()}: ${type} in ${file}`)
        console.error(`    Found ${matches} occurrence(s)`)
        console.error(`    Preview: ${preview}`)
        console.error('')
    })

    console.error('ACTION REQUIRED:')
    console.error('  1. Remove hardcoded secrets')
    console.error('  2. Use environment variables')
    console.error('  3. Add to .gitignore if needed')
    console.error('')

    process.exit(1)
}

console.log('✅ No secrets detected')
```

**Hook integration** (`.husky/pre-commit`):
```bash
#!/bin/bash

# Scan for secrets
echo "🔒 Scanning for secrets..."
node scripts/scan-secrets.js || exit 1
```

### Hook 2: Dependency Audit

```bash
#!/bin/bash
# .husky/pre-commit (add this check)

# Check for known vulnerabilities
echo "🔍 Running dependency audit..."
npm audit --audit-level=high || {
    echo "❌ High/critical vulnerabilities found"
    echo "   Run: npm audit fix"
    exit 1
}

echo "✅ No high/critical vulnerabilities"
```

### Hook 3: SQL Injection Check

```javascript
// scripts/check-sql-injection.js

const fs = require('fs')
const { execSync } = require('child_process')

const SQL_INJECTION_PATTERNS = [
    // String concatenation in SQL
    /execute\(['"].*?\+/g,
    /query\(['"].*?\+/g,

    // Template literals without parameterization
    /query\(`.*?\$\{/g,
    /execute\(`.*?\$\{/g
]

function checkFile(filePath) {
    const content = fs.readFileSync(filePath, 'utf8')
    const issues = []

    SQL_INJECTION_PATTERNS.forEach(pattern => {
        if (pattern.test(content)) {
            issues.push({
                file: filePath,
                issue: 'Possible SQL injection - use parameterized queries'
            })
        }
    })

    return issues
}

// Run check
const stagedFiles = execSync('git diff --cached --name-only')
    .toString()
    .split('\n')
    .filter(f => f.match(/\.(ts|js)$/))

const issues = stagedFiles.flatMap(f => fs.existsSync(f) ? checkFile(f) : [])

if (issues.length > 0) {
    console.error('❌ SECURITY: Possible SQL injection found!')
    issues.forEach(({ file, issue }) => {
        console.error(`  ${file}: ${issue}`)
    })
    console.error('')
    console.error('Use parameterized queries instead:')
    console.error('  ❌ query(`SELECT * FROM users WHERE id = ${userId}`)')
    console.error('  ✅ query("SELECT * FROM users WHERE id = ?", [userId])')
    process.exit(1)
}
```

## Periodic Security Audits

### Weekly Automated Scan

**GitHub Action** (`.github/workflows/security-scan.yml`):
```yaml
name: Weekly Security Scan

on:
  schedule:
    # Run every Monday at 9am
    - cron: '0 9 * * 1'
  workflow_dispatch:  # Allow manual trigger

jobs:
  security-scan:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run npm audit
        run: npm audit --json > audit-report.json
        continue-on-error: true

      - name: Scan for secrets
        run: node scripts/scan-secrets.js

      - name: Check for outdated deps
        run: npm outdated --json > outdated-report.json
        continue-on-error: true

      - name: Security summary
        run: node scripts/generate-security-report.js

      - name: Create issue if vulnerabilities found
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '🔒 Security vulnerabilities detected',
              body: 'Weekly security scan found issues. See workflow run for details.',
              labels: ['security', 'automated']
            })
```

### Security Checklist (Manual Review)

```markdown
## Security Audit Checklist

### Authentication & Authorization
- [ ] All API endpoints require authentication
- [ ] Authorization checks on sensitive operations
- [ ] JWT tokens have expiration
- [ ] Password hashing (bcrypt/argon2)
- [ ] No default credentials

### Input Validation
- [ ] All user input validated
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevented (sanitize HTML)
- [ ] Command injection prevented (no eval, no shell injection)
- [ ] File upload restrictions (type, size)

### Data Protection
- [ ] Secrets in environment variables (not code)
- [ ] Database credentials encrypted
- [ ] HTTPS enforced
- [ ] Sensitive data encrypted at rest
- [ ] No PII in logs

### Dependencies
- [ ] All dependencies up to date
- [ ] No known vulnerabilities (npm audit)
- [ ] Lock file committed (package-lock.json)
- [ ] License compliance

### API Security
- [ ] Rate limiting implemented
- [ ] CORS configured correctly
- [ ] Content-Type validation
- [ ] Request size limits
- [ ] Error messages don't leak info

### Infrastructure
- [ ] Environment variables for config
- [ ] No hardcoded IPs/URLs
- [ ] Firewall rules configured
- [ ] Database access restricted
- [ ] Logs don't contain secrets
```

## Learning from Security Issues (AgentDB)

### Store Security Patterns

```sql
-- After finding vulnerability
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'SQL injection via string concatenation',
    'security_audit',
    0.95,
    'vulnerability_found'
);

-- Store fix
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'Use parameterized queries for all SQL',
    'security_fix',
    0.92,
    'vulnerability_fixed'
);

-- Causal link
INSERT INTO causal_links (cause, effect, confidence)
VALUES (
    'parameterized_queries',
    'prevent_sql_injection',
    0.98
);
```

### Query Before Code Changes

```sql
-- Check for known security issues in this context
SELECT pattern, confidence
FROM patterns
WHERE context = 'security_audit'
AND outcome = 'vulnerability_found'
ORDER BY confidence DESC;

-- Expected results after learning:
-- "SQL injection via string concatenation" (0.95)
-- "XSS via unescaped user input" (0.91)
-- "Hardcoded API keys" (0.89)
```

## Common Vulnerabilities (Top 10)

### 1. Hardcoded Secrets

**Bad**:
```typescript
const apiKey = "sk-1234567890abcdef"  // ❌
```

**Good**:
```typescript
const apiKey = process.env.API_KEY  // ✅
```

### 2. SQL Injection

**Bad**:
```typescript
db.query(`SELECT * FROM users WHERE id = ${userId}`)  // ❌
```

**Good**:
```typescript
db.query("SELECT * FROM users WHERE id = ?", [userId])  // ✅
```

### 3. XSS (Cross-Site Scripting)

**Bad**:
```typescript
element.innerHTML = userInput  // ❌
```

**Good**:
```typescript
element.textContent = userInput  // ✅
// Or use DOMPurify for HTML
element.innerHTML = DOMPurify.sanitize(userInput)
```

### 4. Command Injection

**Bad**:
```typescript
exec(`convert ${userFilename} output.jpg`)  // ❌
```

**Good**:
```typescript
execFile('convert', [userFilename, 'output.jpg'])  // ✅
```

### 5. Path Traversal

**Bad**:
```typescript
fs.readFileSync(`./uploads/${userFilename}`)  // ❌
// userFilename could be "../../../etc/passwd"
```

**Good**:
```typescript
const safePath = path.join('./uploads', path.basename(userFilename))
if (!safePath.startsWith('./uploads')) throw new Error('Invalid path')
fs.readFileSync(safePath)  // ✅
```

### 6. Weak Randomness

**Bad**:
```typescript
const token = Math.random().toString(36)  // ❌ Predictable
```

**Good**:
```typescript
const token = crypto.randomBytes(32).toString('hex')  // ✅
```

### 7. No Rate Limiting

**Bad**:
```typescript
app.post('/login', async (req, res) => {
    // No rate limiting ❌
    const user = await authenticate(req.body)
})
```

**Good**:
```typescript
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 5  // Limit each IP to 5 requests per window
})

app.post('/login', limiter, async (req, res) => {  // ✅
    const user = await authenticate(req.body)
})
```

### 8. Insecure Dependencies

**Check regularly**:
```bash
npm audit
npm outdated
```

### 9. Insufficient Logging

**Bad**:
```typescript
try {
    await deleteUser(userId)
} catch (err) {
    // Silent failure ❌
}
```

**Good**:
```typescript
try {
    await deleteUser(userId)
} catch (err) {
    logger.error('Failed to delete user', {  // ✅
        userId,
        error: err.message,
        stack: err.stack
    })
    throw err
}
```

### 10. Missing HTTPS

**Enforce HTTPS**:
```typescript
app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https' && process.env.NODE_ENV === 'production') {
        res.redirect(`https://${req.header('host')}${req.url}`)
    } else {
        next()
    }
})
```

## Security Automation Summary

### Pre-Commit (Every Commit)
- [ ] Scan for secrets
- [ ] Check for SQL injection patterns
- [ ] Validate no hardcoded credentials

### Pre-Push (Before Remote)
- [ ] Run npm audit
- [ ] Check dependency licenses

### Weekly (Automated)
- [ ] Dependency vulnerability scan
- [ ] Check for outdated packages
- [ ] Create issue if problems found

### Monthly (Manual)
- [ ] Full security checklist review
- [ ] Review access controls
- [ ] Update security docs

---

**Status**: ✅ Ready for implementation
**Priority**: 🔥 HIGH - Security is critical
**Next**: Install secret scanning hook
