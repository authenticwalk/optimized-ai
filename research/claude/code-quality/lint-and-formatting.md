# Lint & Code Formatting Best Practices

**Research Quality**: ⭐⭐⭐⭐ (quality-workflow-meta, Pheromind, Roo Code)
**Status**: ✅ IMPLEMENTATION-READY

## Core Principle: Auto-Fix Over Manual

**Philosophy**: Automate formatting, focus reviews on logic

## ESLint + Prettier Setup

### Installation

```bash
npm install -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
npm install -D eslint-plugin-import eslint-plugin-unused-imports
```

### ESLint Configuration

**File**: `.eslintrc.json`

```json
{
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "plugin:import/recommended",
    "plugin:import/typescript",
    "prettier"
  ],
  "plugins": [
    "@typescript-eslint",
    "import",
    "unused-imports"
  ],
  "rules": {
    // Complexity limits (from Pheromind research)
    "complexity": ["error", 10],
    "max-lines-per-function": ["warn", {
      "max": 50,
      "skipBlankLines": true,
      "skipComments": true
    }],
    "max-lines": ["warn", {
      "max": 500,
      "skipBlankLines": true,
      "skipComments": true
    }],

    // Import organization
    "import/order": ["error", {
      "groups": [
        "builtin",   // Node built-ins
        "external",  // npm packages
        "internal",  // Internal imports
        "parent",    // ../
        "sibling",   // ./
        "index"      // ./index
      ],
      "newlines-between": "always",
      "alphabetize": {
        "order": "asc",
        "caseInsensitive": true
      }
    }],

    // Remove unused imports
    "unused-imports/no-unused-imports": "error",
    "unused-imports/no-unused-vars": ["warn", {
      "vars": "all",
      "varsIgnorePattern": "^_",
      "args": "after-used",
      "argsIgnorePattern": "^_"
    }],

    // No console.log in production
    "no-console": ["error", {
      "allow": ["warn", "error"]
    }],

    // Prevent common mistakes
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/await-thenable": "error"
  }
}
```

### Prettier Configuration

**File**: `.prettierrc.json`

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "type-check": "tsc --noEmit",
    "quality": "npm run lint && npm run format:check && npm run type-check"
  }
}
```

## Pre-Commit Hooks (Enforcement)

### Hook 1: Lint Staged Files

**Install lint-staged**:
```bash
npm install -D lint-staged
```

**Configuration** (`.lintstagedrc.json`):
```json
{
  "*.{ts,tsx}": [
    "eslint --fix",
    "prettier --write"
  ],
  "*.{json,md}": [
    "prettier --write"
  ]
}
```

**Hook** (`.husky/pre-commit`):
```bash
#!/bin/bash

# Run lint-staged
npx lint-staged || {
    echo "❌ Linting/formatting failed"
    echo "   Run: npm run lint:fix && npm run format"
    exit 1
}

echo "✅ Linting passed"
```

### Hook 2: Import Organization

**Auto-organize imports** (before lint):
```typescript
// scripts/organize-imports.ts

import { execSync } from 'child_process'
import * as path from 'path'

function organizeImportsInFile(filePath: string) {
    // Use TypeScript compiler API to organize imports
    const ts = require('typescript')

    const program = ts.createProgram([filePath], {
        allowJs: true,
        module: ts.ModuleKind.ES2020
    })

    const sourceFile = program.getSourceFile(filePath)
    const languageService = ts.createLanguageService({
        getCompilationSettings: () => program.getCompilerOptions(),
        getScriptFileNames: () => [filePath],
        getScriptVersion: () => '0',
        getScriptSnapshot: (fileName: string) => {
            if (fileName === filePath) {
                return ts.ScriptSnapshot.fromString(
                    fs.readFileSync(fileName, 'utf8')
                )
            }
            return undefined
        },
        getCurrentDirectory: () => process.cwd(),
        getDefaultLibFileName: ts.getDefaultLibFilePath
    })

    // Get organize imports edits
    const changes = languageService.organizeImports(
        { type: 'file', fileName: filePath },
        {},
        {}
    )

    // Apply changes
    applyTextChanges(filePath, changes[0]?.textChanges || [])
}

// Run on staged files
const stagedFiles = execSync('git diff --cached --name-only')
    .toString()
    .split('\n')
    .filter(f => f.match(/\.tsx?$/))

stagedFiles.forEach(organizeImportsInFile)
```

**Simpler alternative using eslint**:
```bash
# In .husky/pre-commit
npx eslint --fix --rule 'import/order: error' $(git diff --cached --name-only | grep '\.ts$')
```

## VSCode Settings (Team Consistency)

**File**: `.vscode/settings.json`

```json
{
  // Auto-format on save
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",

  // Auto-fix on save
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },

  // TypeScript settings
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.updateImportsOnFileMove.enabled": "always",

  // Prettier as default
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## Complexity Checking (FTA)

**From quality-workflow-meta research**:

### Installation

```bash
npm install -D fta-cli
```

### Configuration

**File**: `fta.config.json`

```json
{
  "hardCap": 50,
  "softCap": 30,
  "exclude": [
    "node_modules",
    "dist",
    "*.test.ts",
    "*.spec.ts"
  ]
}
```

### Check Script

```javascript
// scripts/check-fta-cap.mjs

import fs from 'fs'

const FTA_HARD_CAP = process.env.FTA_HARD_CAP || 50
const FTA_SOFT_CAP = process.env.FTA_SOFT_CAP || 30

const report = JSON.parse(fs.readFileSync('reports/fta.json', 'utf-8'))

const hardViolations = report.files.filter(f => f.complexity > FTA_HARD_CAP)
const softViolations = report.files.filter(f => f.complexity > FTA_SOFT_CAP && f.complexity <= FTA_HARD_CAP)

if (hardViolations.length > 0) {
    console.error(`❌ FTA hard cap (${FTA_HARD_CAP}) exceeded:`)
    hardViolations.forEach(f => {
        console.error(`   ${f.path}: ${f.complexity}`)
    })
    process.exit(1)
}

if (softViolations.length > 0) {
    console.warn(`⚠️  FTA soft cap (${FTA_SOFT_CAP}) exceeded:`)
    softViolations.forEach(f => {
        console.warn(`   ${f.path}: ${f.complexity}`)
    })
    console.warn('   Consider refactoring these files')
}

console.log(`✅ All files under FTA hard cap (${FTA_HARD_CAP})`)
```

### Package.json Scripts

```json
{
  "scripts": {
    "complexity": "fta src --format json > reports/fta.json",
    "complexity:check": "npm run complexity && node scripts/check-fta-cap.mjs"
  }
}
```

### Pre-Commit Integration

```bash
# .husky/pre-commit (add this)

# Check complexity
echo "📊 Checking code complexity..."
npm run complexity:check || exit 1
```

## Learning from Code Quality (AgentDB)

### Store Quality Patterns

```sql
-- After refactoring complex function
INSERT INTO patterns (pattern, context, confidence, outcome)
VALUES (
    'Extract complex logic into smaller functions (complexity 45 → 8)',
    'code_quality',
    0.91,
    'refactoring_success'
);

-- Track violations
INSERT INTO failures (pattern, context, error_message)
VALUES (
    'Function exceeded 50 lines',
    'code_quality',
    'Had to refactor during code review'
);

-- Causal link
INSERT INTO causal_links (cause, effect, confidence)
VALUES (
    'functions_under_50_lines',
    'faster_code_reviews',
    0.85
);
```

## Common Quality Issues & Auto-Fixes

### Issue 1: Unused Imports

**Auto-fix**:
```bash
npx eslint --fix --rule 'unused-imports/no-unused-imports: error' src/
```

### Issue 2: Inconsistent Formatting

**Auto-fix**:
```bash
npx prettier --write src/
```

### Issue 3: Missing Semicolons (or Extra)

**Auto-fix** (Prettier handles this):
```json
// .prettierrc.json
{
  "semi": false  // or true, team decides
}
```

### Issue 4: Unorganized Imports

**Auto-fix**:
```bash
npx eslint --fix --rule 'import/order: error' src/
```

### Issue 5: console.log Left in Code

**Detection + block**:
```json
// .eslintrc.json
{
  "rules": {
    "no-console": ["error", { "allow": ["warn", "error"] }]
  }
}
```

## CI Integration

**GitHub Action** (`.github/workflows/quality-check.yml`):
```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Format Check
        run: npm run format:check

      - name: Type Check
        run: npm run type-check

      - name: Complexity Check
        run: npm run complexity:check
```

## Summary: Quick Reference

### Setup
```bash
# Install tools
npm install -D eslint prettier lint-staged fta-cli

# Configure (create config files)
.eslintrc.json
.prettierrc.json
.lintstagedrc.json

# Add scripts to package.json
lint, format, complexity:check

# Add pre-commit hook
.husky/pre-commit
```

### Pre-Commit Checks
1. lint-staged (auto-fix formatting)
2. ESLint (catch errors)
3. Organize imports
4. Complexity check

### Auto-Fixes Available
- Formatting (Prettier)
- Import organization
- Unused import removal
- Simple ESLint fixes

### Manual Fixes Required
- Complexity reduction
- Logic errors
- Type errors
- Architectural issues

---

**Status**: ✅ Ready for implementation
**Next**: Install ESLint + Prettier + hooks
