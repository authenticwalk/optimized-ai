# Cursor Rules & AI Agent Configuration Research

> Research conducted on awesome-cursorrules and related resources for AI coding assistant configurations

## Executive Summary

This research explores the ecosystem of AI coding assistant configuration files (`.cursorrules`, `CLAUDE.md`, `.mdc`) used to customize AI behavior in tools like Cursor and Claude Code. The findings reveal established patterns, best practices, and a thriving community sharing reusable configurations across 179+ rule sets.

## Key Repositories & Resources

### Primary Collections

1. **PatrickJS/awesome-cursorrules** ⭐ 35.1k stars
   - URL: https://github.com/PatrickJS/awesome-cursorrules
   - **179 total rule sets** across 12+ categories
   - Covers frontend, backend, mobile, testing, DevOps
   - Licensed: CC0-1.0 (public domain)
   - Two directories: `rules/` (stable) and `rules-new/` (experimental)

2. **steipete/agent-rules**
   - URL: https://github.com/steipete/agent-rules
   - Focus: Global rules for Claude Code and Cursor
   - Provides `.mdc` format files (cross-compatible)
   - Can be placed in `~/.claude/CLAUDE.md` for global application

3. **hao-ji-xing/awesome-cursor**
   - URL: https://github.com/hao-ji-xing/awesome-cursor
   - Curated collection of Cursor tools and resources

4. **tugkanboz/awesome-cursorrules**
   - URL: https://github.com/tugkanboz/awesome-cursorrules
   - Alternative curated list of `.cursorrules` files

5. **SabrinaRamonov/ai-coding-rules**
   - URL: https://github.com/SabrinaRamonov/ai-coding-rules
   - Personal CLAUDE.md file examples

### Community Resources

- **Cursor Forum Collection**: 879 `.mdc` Cursor Rules files
  - URL: https://forum.cursor.com/t/created-a-collection-of-879-mdc-cursor-rules-files-for-you-all/51634

- **GitHub Topics**:
  - https://github.com/topics/cursorrules
  - https://github.com/topics/cursor-rules

## High-Level Patterns & Best Practices

### 1. File Structure Patterns

**Three Main Configuration Types:**

| Type | File Name | Scope | Used By |
|------|-----------|-------|---------|
| Project Rules | `.cursorrules` | Project-specific | Cursor IDE |
| Global Rules | `~/.claude/CLAUDE.md` | All projects | Claude Code |
| Modular Rules | `.mdc` files | Composable | Both tools |

**Cross-Compatibility Strategy:**
- `.mdc` files use YAML frontmatter (Cursor reads this)
- Markdown content works for both tools (Claude ignores frontmatter)
- Project-level: Place in `.cursor/rules/` directory

### 2. Rule Organization Principles

**Keep Rules Concise:**
- Target: <500 lines per rule file
- Use modular `.mdc` files for composition
- Avoid duplication through rule blocks

**Naming Conventions:**
- Pattern: `{technology}-{focus}-cursorrules-prompt-file`
- Examples:
  - `angular-typescript-cursorrules-prompt-file`
  - `nextjs-react-tailwind-cursorrules-prompt-file`
  - `typescript-nestjs-best-practices-cursorrules-promp`

### 3. Common Rule Categories

Across 179 rule sets, these categories dominate:

1. **Frontend Frameworks** (Angular, React, Vue, Svelte, Solid, Qwik)
2. **Type Safety** (TypeScript, strict typing patterns)
3. **Styling** (Tailwind CSS, CSS-in-JS, component libraries)
4. **Backend** (NestJS, Express, FastAPI, Rails)
5. **Testing** (Jest, Cypress, Playwright, Vitest)
6. **Code Quality** (ESLint, Prettier, DRY principles)
7. **Architecture** (Component patterns, state management)
8. **Documentation** (JSDoc, inline comments)

### 4. Universal Best Practices

**Code Quality Principles:**
```
✓ KISS (Keep It Simple, Stupid)
✓ YAGNI (You Aren't Gonna Need It)
✓ DRY (Don't Repeat Yourself)
✓ Early returns over nested conditionals
✓ Functional/immutable style when practical
```

**TypeScript Standards:**
```typescript
✓ Always declare types (parameters + return values)
✓ Avoid 'any' type
✓ Use interfaces over types
✓ Avoid enums; prefer const objects
✓ Enable strict mode
```

**Naming Conventions:**
```
✓ PascalCase: Classes, interfaces, types
✓ camelCase: Variables, functions, methods
✓ kebab-case: Files and directories
✓ Prefix handlers: handleClick, handleSubmit
✓ Descriptive names over abbreviations
```

**Function Design:**
```
✓ Max 4 parameters per function
✓ Max 50 executable lines per function
✓ Max 2 levels of nesting
✓ Max 80 characters per line
✓ Single responsibility principle
```

## Angular-Specific Recommendations

### Current Angular Rules (from awesome-cursorrules)

**Available Angular Configurations:**
1. `angular-typescript-cursorrules-prompt-file`
2. `angular-novo-elements-cursorrules-prompt-file`

### Angular Rule Patterns Found

**Core Principles:**
```
Expert Angular programmer using:
- TypeScript (latest)
- Angular 18+
- Jest for testing
- Focus: clear, readable, maintainable code
```

**Code Structure:**
```
✓ Max 2 levels of code nesting
✓ Max 4 function parameters
✓ Max 50 executable lines per function
✓ Max 80 characters per line
```

**Angular-Specific:**
```
✓ Prefer forNext() utility over traditional loops
  Location: libs/smart-ngrx/src/common/for-next.function.ts
✓ Obey .eslintrc.json, .prettierrc, .htmlhintrc, .editorconfig
✓ Maintain JSDoc comments when refactoring
✓ Include all required imports
✓ Proper component/service naming
```

### Recommended Angular Rule Enhancements

**Angular-Specific Patterns to Add:**
```
1. Component Architecture:
   - Smart/Container vs Dumb/Presentational split
   - OnPush change detection strategy by default
   - Standalone components (Angular 14+)
   - Signal-based state management (Angular 16+)

2. Reactive Patterns:
   - Prefer RxJS operators over imperative logic
   - Use async pipe over manual subscriptions
   - Proper subscription cleanup (takeUntil, DestroyRef)
   - Avoid nested subscriptions

3. Performance Optimizations:
   - Lazy loading for routes
   - Track by functions in *ngFor
   - Pure pipes for transformations
   - Virtual scrolling for large lists

4. Angular-Specific TypeScript:
   - Use Angular types: OnInit, OnDestroy, etc.
   - Proper typing for ViewChild/ContentChild
   - Generic types for components/services
   - Strict template checking

5. Testing Patterns:
   - TestBed configuration standards
   - Component testing with fixtures
   - Service testing with dependency injection
   - Async testing patterns (fakeAsync, waitForAsync)
```

## TypeScript-Specific Recommendations

### Current TypeScript Rules Available

**Popular Configurations:**
1. `typescript-code-convention-cursorrules-prompt-file`
2. `javascript-typescript-code-quality-cursorrules-pro`
3. `typescript-nestjs-best-practices-cursorrules-promp`
4. Multiple React+TypeScript combinations

### TypeScript Rule Patterns Found

**Core Quality Standards:**
```
✓ Senior full-stack developer mindset
✓ "10x developer" capabilities
✓ 6 Core Mindsets: Simplicity, Readability, Performance,
  Maintainability, Testability, Reusability
```

**Coding Guidelines:**
```
✓ Early returns (avoid nested conditions)
✓ Descriptive names (prefix handlers: handle*)
✓ Constants over functions when applicable
✓ DRY code with functional/immutable style
✓ Minimal changes (only modify relevant sections)
```

**Documentation:**
```
✓ JSDoc comments at function start
✓ Describe function purpose
✓ Modern ES6+ syntax
✓ TODO comments for problematic code
```

**Problem-Solving:**
```
Chain of Thought methodology:
1. Outline pseudocode plan
2. Confirm approach
3. Implement solution
```

### Recommended TypeScript Rule Enhancements

**Advanced TypeScript Patterns:**
```typescript
1. Type Safety Enhancements:
   - Use discriminated unions for state management
   - Leverage conditional types for complex scenarios
   - Utilize mapped types for transformations
   - Implement branded types for domain modeling
   - Use const assertions for literal types

2. Generic Patterns:
   - Generic constraints with extends
   - Default generic parameters
   - Generic factory functions
   - Type inference optimization

3. Utility Type Usage:
   - Partial, Required, Pick, Omit, Record
   - ReturnType, Parameters for function types
   - Awaited for promise types
   - Custom utility type creation

4. Advanced Patterns:
   - Builder pattern with type safety
   - Factory pattern with generics
   - Dependency injection with interfaces
   - Decorator pattern (when using experimentalDecorators)

5. Error Handling:
   - Result/Either types for functional error handling
   - Custom error classes with type guards
   - Exhaustive checking with never type
   - Type-safe try-catch patterns
```

## CSS/Tailwind-Specific Recommendations

### Current CSS/Tailwind Rules Available

**Popular Configurations:**
1. `tailwind-css-nextjs-guide-cursorrules-prompt-file`
2. `html-tailwind-css-javascript-cursorrules-prompt-fi`
3. `qwik-tailwind-cursorrules-prompt-file`
4. `solidjs-tailwind-cursorrules-prompt-file`
5. Multiple framework+Tailwind combinations

### Tailwind CSS Rule Patterns Found

**Core Styling Principles:**
```
✓ Use Tailwind classes exclusively
✓ Avoid inline styles
✓ Implement responsive design with responsive classes
✓ Use @apply directive for reusable styles
✓ Configure dark mode with dark: variant
✓ Customize via tailwind.config.js/CSS
```

**Tailwind v4.0 Paradigm Shift:**
```
BEFORE (v3): JavaScript configuration
// tailwind.config.js
module.exports = {
  theme: { colors: { primary: '#1234' } }
}

AFTER (v4): CSS-first configuration
/* app.css */
@import "tailwindcss";
@theme {
  --color-primary: #1234;
}
```

**Version 4.0 Key Changes:**
```
✓ @import "tailwindcss" instead of @tailwind directives
✓ @theme directive for design tokens
✓ CSS variables: --color-*, --font-*, --spacing-*
✓ No autoprefixer/postcss-import needed
✓ Container queries built-in (@container)
✓ 3D transforms (rotate-x-*, perspective-*)
✓ Enhanced gradients (bg-linear-45, /oklch, /srgb)
✓ Composable variants (group-has-data-*:opacity-100)
```

**Breaking Changes v4.0:**
```
✗ Opacity utilities removed (bg-opacity-20 → bg-black/20)
✗ Size scaling renamed (shadow-sm → shadow-xs)
✗ Border defaults now currentColor
✗ Ring width default: 3px → 1px
```

### Recommended CSS/Tailwind Rule Enhancements

**Advanced Tailwind Patterns:**
```css
1. Component Pattern Library:
   /* Use @layer components for reusable patterns */
   @layer components {
     .btn-primary {
       @apply px-4 py-2 bg-primary text-white rounded-lg
              hover:bg-primary-dark transition-colors;
     }
   }

2. Utility Extension:
   /* Create custom utilities */
   @utility tab-4 { tab-size: 4; }
   @utility text-balance { text-wrap: balance; }

3. Custom Variants:
   /* Define custom variant conditions */
   @variant pointer-coarse (@media (pointer: coarse));
   @variant supports-grid (@supports (display: grid));

4. Design Token System:
   @theme {
     /* Semantic color naming */
     --color-brand-primary: oklch(0.6 0.2 250);
     --color-brand-secondary: oklch(0.7 0.15 180);

     /* Consistent spacing scale */
     --spacing-xs: 0.25rem;
     --spacing-sm: 0.5rem;
     --spacing-md: 1rem;

     /* Typography scale */
     --font-size-xs: 0.75rem;
     --font-size-sm: 0.875rem;
   }

5. Responsive Design Patterns:
   /* Mobile-first approach */
   <div class="
     grid grid-cols-1          /* Mobile: 1 column */
     sm:grid-cols-2            /* Small: 2 columns */
     lg:grid-cols-3            /* Large: 3 columns */
     xl:grid-cols-4            /* XL: 4 columns */
     gap-4 sm:gap-6 lg:gap-8   /* Progressive spacing */
   ">

6. Container Query Patterns (v4):
   <div class="@container">
     <div class="@sm:grid-cols-2 @lg:grid-cols-3">
       <!-- Responsive to container, not viewport -->
     </div>
   </div>

7. Dark Mode Patterns:
   /* Class-based dark mode */
   <div class="
     bg-white dark:bg-gray-900
     text-gray-900 dark:text-gray-100
   ">

8. Performance Best Practices:
   ✓ Use JIT mode for smaller bundle sizes
   ✓ Configure content paths accurately for purging
   ✓ Avoid unnecessary @apply in production
   ✓ Use CSS variables for dynamic theming
   ✓ Leverage built-in container queries over JS
```

**CSS Architecture Recommendations:**
```
1. File Organization:
   /styles
     /base       - Reset, typography
     /components - Reusable component styles
     /utilities  - Custom utility classes
     /layouts    - Grid systems, containers
     /themes     - Color schemes, variants

2. Naming Strategy:
   - Use semantic class names for components
   - Prefix custom utilities (u-*)
   - Component variants via modifiers (btn--primary)

3. Accessibility Patterns:
   ✓ Focus states (focus-visible:ring-2)
   ✓ Color contrast compliance
   ✓ Screen reader utilities (sr-only)
   ✓ Reduced motion (motion-safe:/motion-reduce:)

4. Progressive Enhancement:
   @supports (backdrop-filter: blur(10px)) {
     .glass-effect {
       backdrop-filter: blur(10px);
     }
   }
```

## Unique Rule Recommendations for Our Stack

### Angular + TypeScript + CSS/Tailwind Integration

**Proposed `.cursorrules` Structure:**
```markdown
# Expert Angular Developer with TypeScript & Tailwind CSS

You are an expert full-stack Angular developer specializing in:
- Angular 18+ with standalone components and signals
- TypeScript 5+ with strict mode
- Tailwind CSS v4 with CSS-first configuration
- RxJS reactive patterns
- Jest/Testing Library for testing

## Core Principles

1. **Code Quality**
   - Write clear, maintainable, performant code
   - Follow SOLID principles
   - Use functional/reactive patterns
   - Prefer immutability

2. **TypeScript Standards**
   - Strict type checking enabled
   - No 'any' types
   - Interfaces for public APIs
   - Generics for reusability
   - Proper error typing

3. **Angular Architecture**
   - Standalone components by default
   - OnPush change detection
   - Signal-based state management
   - Smart/Dumb component separation
   - Reactive forms over template-driven

4. **Styling Standards**
   - Tailwind CSS classes exclusively
   - CSS-first theme configuration (@theme)
   - Responsive design (mobile-first)
   - Dark mode support
   - Accessibility compliance (WCAG 2.1 AA)

## Code Structure Rules

### Functions
- Max 4 parameters
- Max 50 executable lines
- Max 2 levels of nesting
- Early returns for guards
- Single responsibility

### TypeScript
- Always declare types (params + return)
- Use const assertions where appropriate
- Discriminated unions for state
- Type guards for runtime checks
- Branded types for domain models

### Angular Components
- Standalone: true
- changeDetection: OnPush
- Signals for local state
- Observables for async operations
- Proper lifecycle cleanup (DestroyRef)

### Styling
- Tailwind utility classes
- @layer components for patterns
- CSS variables for theming
- Container queries for responsiveness
- Semantic color naming

## Testing Standards

- Unit tests for all components/services
- Integration tests for features
- E2E tests for critical flows
- AAA pattern (Arrange, Act, Assert)
- TestBed for dependency injection
- Marble testing for RxJS

## Documentation

- JSDoc for public APIs
- Inline comments for complex logic
- Component usage examples
- README for module documentation

## Performance

- Lazy loading for routes
- Virtual scrolling for lists
- Track by functions in loops
- Pure pipes for transformations
- Bundle size optimization

## Error Handling

- Result types for predictable errors
- Custom error classes
- Proper error boundaries
- User-friendly error messages
- Logging for debugging
```

## Links for Further Development

### Primary Resources

**Documentation:**
- awesome-cursorrules README: https://github.com/PatrickJS/awesome-cursorrules/blob/main/README.md
- Claude Code Docs: https://docs.claude.com/en/docs/claude-code
- Cursor AI Documentation: https://docs.cursor.com

**Rule Collections:**
- All Angular rules: https://github.com/PatrickJS/awesome-cursorrules/tree/main/rules?filter=angular
- All TypeScript rules: https://github.com/PatrickJS/awesome-cursorrules/tree/main/rules?filter=typescript
- All Tailwind rules: https://github.com/PatrickJS/awesome-cursorrules/tree/main/rules?filter=tailwind

**Specific Rule Files:**
1. Angular + TypeScript:
   - https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules/angular-typescript-cursorrules-prompt-file/.cursorrules

2. TypeScript Conventions:
   - https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules/typescript-code-convention-cursorrules-prompt-file/.cursorrules
   - https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules/javascript-typescript-code-quality-cursorrules-pro/.cursorrules

3. Tailwind Best Practices:
   - https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules/html-tailwind-css-javascript-cursorrules-prompt-fi/.cursorrules
   - Tailwind v4 Rules: https://gist.github.com/danhollick/d902cf60e37950de36cf8e7c43fa0943

**Cross-Tool Configuration:**
- steipete agent-rules: https://github.com/steipete/agent-rules
- Global rules directory: https://github.com/steipete/agent-rules/tree/main/global-rules
- Project rules directory: https://github.com/steipete/agent-rules/tree/main/project-rules

### Community Resources

**Forums & Discussions:**
- Cursor Community Forum: https://forum.cursor.com
- 879 MDC Rules Collection: https://forum.cursor.com/t/created-a-collection-of-879-mdc-cursor-rules-files-for-you-all/51634
- GitHub Discussions: https://github.com/PatrickJS/awesome-cursorrules/discussions

**Blog Posts & Guides:**
- Claude Code vs Cursor Comparison: https://www.qodo.ai/blog/claude-code-vs-cursor/
- How I use Claude Code: https://www.builder.io/blog/claude-code
- Mastering Cursor AI Guide: https://dev.to/mayank_tamrkar/-mastering-cursor-ai-the-ultimate-guide-for-developers-2025-edition-2ihh
- Best Practices Repository: https://github.com/digitalchild/cursor-best-practices

**GitHub Topics to Follow:**
- cursorrules: https://github.com/topics/cursorrules
- cursor-rules: https://github.com/topics/cursor-rules
- claude-ai: https://github.com/topics/claude-ai

### Advanced Topics

**Tool-Specific Configuration:**
- Claude Code GitHub Actions: https://docs.claude.com/en/docs/claude-code/github-actions
- Claude Code GitHub Integration: https://www.eesel.ai/blog/claude-code-github-integration
- Permission Model in Claude Code: https://skywork.ai/blog/permission-model-claude-code-vs-code-jetbrains-cli/

**Related Projects:**
- blueraai/clauder (Enhanced Claude Code): https://github.com/blueraai/clauder
- Claude AI Toolkit: https://github.com/RMNCLDYO/claude-ai-toolkit
- Agent Rules Gist Collection: https://gist.github.com/0xdevalias/f40bc5a6f84c4c5ad862e314894b2fa6

## Action Items

### Immediate Next Steps

1. **Review Existing Rules**
   - Download and analyze Angular TypeScript rules
   - Compare with our current project patterns
   - Identify gaps in our configuration

2. **Create Project-Specific Rules**
   - Draft `.cursorrules` for our Angular stack
   - Add Tailwind v4 configuration patterns
   - Include project-specific conventions

3. **Implement Global Rules**
   - Create `~/.claude/CLAUDE.md` for cross-project standards
   - Add TypeScript best practices
   - Configure code quality standards

4. **Test & Iterate**
   - Apply rules to sample projects
   - Measure AI suggestion quality
   - Refine based on team feedback

### Long-Term Improvements

1. **Build Rule Library**
   - Create modular `.mdc` files
   - Organize by concern (styling, testing, architecture)
   - Share across team repositories

2. **Documentation**
   - Document our custom rules
   - Create usage examples
   - Maintain changelog for rule updates

3. **Community Contribution**
   - Consider contributing our unique rules back
   - Share Angular-specific patterns
   - Engage with awesome-cursorrules community

## Conclusion

The cursorrules ecosystem provides a rich foundation for customizing AI coding assistants. With 179+ established rule sets and a vibrant community, teams can rapidly adopt best practices while tailoring AI behavior to their specific tech stacks.

For Angular + TypeScript + Tailwind CSS development, combining patterns from existing rules with our unique requirements will create a powerful development accelerator.

**Key Takeaways:**
- ✅ Use `.cursorrules` for project-specific configuration
- ✅ Use `CLAUDE.md` for global/cross-project standards
- ✅ Keep rules concise (<500 lines), modular, and composable
- ✅ Focus on patterns, not just syntax rules
- ✅ Include testing, architecture, and performance guidelines
- ✅ Update rules as frameworks evolve (e.g., Angular signals, Tailwind v4)

---

*Research Date: 2025-11-03*
*Primary Source: PatrickJS/awesome-cursorrules (35.1k ⭐)*
*Tools Analyzed: Cursor IDE, Claude Code*
