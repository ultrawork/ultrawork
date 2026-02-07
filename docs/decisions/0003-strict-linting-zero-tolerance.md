---
title: "ADR-0003: Strict Linting with Zero Tolerance"
category: "decisions"
tags: ["code-quality", "linting", "detekt", "eslint", "adr", "accepted"]
status: "Accepted"
date: "2026-02-07"
last_updated: "2026-02-07"
---

# 0003. Strict Linting with Zero Tolerance

**Status**: Accepted  
**Date**: 2026-02-07  
**Deciders**: Ultrawork Team

## Context

The Ultrawork project requires high code quality standards to ensure:
- Consistent code style across backend (Kotlin) and frontend (TypeScript)
- Early detection of potential bugs and code smells
- Maintainability as the codebase grows
- Reduced cognitive load during code reviews
- Prevention of common anti-patterns and security vulnerabilities

Traditional linting approaches allow warnings to accumulate, leading to:
- "Broken windows" effect where code quality gradually degrades
- Warnings ignored during development and code review
- Inconsistent enforcement across team members
- Technical debt accumulation

We needed a linting strategy that:
- Enforces rules automatically without manual intervention
- Prevents code quality degradation over time
- Provides clear, actionable feedback to developers
- Integrates seamlessly with CI/CD pipeline
- Fails builds on any violations (no warnings allowed)

## Decision

We will enforce **zero-tolerance linting policies** for both backend and frontend codebases, where any linting violation causes build failure.

**Backend (Kotlin):**
- **Tool**: Detekt 1.23.8 with custom configuration
- **Policy**: `maxIssues: 0` (build fails on any issue)
- **Rules Enforced**:
  - **Style**: Max line length 120, magic number detection, max return count 3
  - **Complexity**: Cyclomatic complexity ≤15, method length ≤60 lines, max 6 parameters, max 15 functions per file
  - **Naming**: Enforced naming conventions for functions, classes, variables
  - **Formatting**: Auto-correct enabled for consistent code style
- **Integration**: Gradle task `detektMain` runs before tests in CI/CD

**Frontend (TypeScript):**
- **Tool**: ESLint 9 with Airbnb TypeScript config
- **Policy**: `--max-warnings 0` (build fails on any warning)
- **Rules Enforced**:
  - **Type Safety**: Explicit function return types required, no `any` type, no unused variables
  - **Style**: Max line length 120, one variable declaration per statement, max 1 class per file
  - **React**: Arrow function components required, no prop spreading restrictions
  - **Imports**: No default export preference (named exports encouraged)
- **Integration**: npm script `lint:strict` runs in CI/CD before build

**CI/CD Enforcement:**
- Backend: GitHub Actions runs `./gradlew detektMain test` (detekt before tests)
- Frontend: GitHub Actions runs `npm run lint:strict && tsc --noEmit && npm run test` (lint before type check and tests)
- Both workflows fail immediately on first linting violation

## Consequences

### Positive
- **Consistent Code Quality**: Zero-tolerance policy prevents code quality degradation over time
- **Early Bug Detection**: Linting catches potential bugs before code review (unused variables, type errors, complexity issues)
- **Reduced Code Review Friction**: Automated enforcement eliminates style debates during code review
- **Improved Maintainability**: Enforced complexity limits (cyclomatic complexity, method length) keep code readable
- **Security Benefits**: Detekt and ESLint detect common security anti-patterns (magic numbers, overly complex logic)
- **Developer Productivity**: Auto-correct features (Detekt formatting, ESLint --fix) reduce manual formatting time
- **Onboarding**: New team members learn project conventions through immediate linting feedback
- **CI/CD Integration**: Build failures on linting violations prevent bad code from reaching production

### Negative
- **Initial Friction**: Developers must fix linting violations immediately (cannot defer to later)
- **Learning Curve**: Team members unfamiliar with Detekt/ESLint rules require onboarding time
- **Build Time Overhead**: Linting adds ~5-10 seconds to backend builds, ~3-5 seconds to frontend builds
- **False Positives**: Occasionally rules flag valid code (requires suppression comments or rule adjustment)
- **Refactoring Burden**: Existing code must be updated to meet linting standards before merging

### Neutral
- **Opinionated Standards**: Airbnb TypeScript config and Detekt defaults impose specific style choices
- **Suppression Comments**: Rare cases require `@Suppress` (Kotlin) or `eslint-disable` (TypeScript) comments
- **Configuration Maintenance**: Linting rules may need adjustment as project evolves

## Alternatives Considered

### Option 1: Warning-Based Linting (Allow Warnings)
- **Pros**: Less friction during development, faster initial development
- **Cons**: Warnings accumulate over time, inconsistent enforcement, "broken windows" effect
- **Why rejected**: Experience shows warning-based linting leads to code quality degradation

### Option 2: Manual Code Review Only (No Automated Linting)
- **Pros**: Maximum flexibility, no build time overhead
- **Cons**: Inconsistent style enforcement, code review friction, human error, slower reviews
- **Why rejected**: Manual enforcement does not scale and introduces inconsistency

### Option 3: Linting as Separate CI Job (Non-Blocking)
- **Pros**: Does not block builds, allows warnings to be addressed later
- **Cons**: Warnings ignored in practice, code quality degrades, defeats purpose of linting
- **Why rejected**: Non-blocking linting provides no enforcement mechanism

### Option 4: Stricter Rules (e.g., Max Line Length 80, Cyclomatic Complexity ≤10)
- **Pros**: Even higher code quality standards
- **Cons**: Excessive friction, forces artificial code splitting, reduces productivity
- **Why rejected**: Current rules (line length 120, complexity ≤15) balance quality and productivity

### Option 5: Prettier for Formatting Only (No Linting)
- **Pros**: Automatic formatting without linting overhead
- **Cons**: Does not catch bugs, complexity issues, or type safety violations
- **Why rejected**: Formatting alone insufficient for code quality goals (need complexity, type safety, naming enforcement)

## Implementation Details

**Backend Detekt Configuration** (`backend/config/detekt/detekt.yml`):
```yaml
build:
  maxIssues: 0  # Zero tolerance

style:
  MaxLineLength:
    maxLineLength: 120
  MagicNumber:
    ignoreNumbers: ['-1', '0', '1', '2']
  ReturnCount:
    max: 3

complexity:
  CyclomaticComplexMethod:
    threshold: 15
  LongMethod:
    threshold: 60
  LongParameterList:
    functionThreshold: 6
  TooManyFunctions:
    thresholdInFiles: 15

formatting:
  autoCorrect: true
```

**Frontend ESLint Configuration** (`frontend/.eslintrc.json`):
```json
{
  "extends": [
    "next/core-web-vitals",
    "airbnb-typescript",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ],
  "rules": {
    "max-len": ["error", { "code": 120 }],
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "one-var": ["error", "never"],
    "max-classes-per-file": ["error", 1]
  }
}
```

**CI/CD Integration:**
- Backend: `./gradlew detektMain` runs before tests
- Frontend: `npm run lint:strict` (ESLint with `--max-warnings 0`) runs before type check and tests
- Both fail builds immediately on violations

## Related Documentation

- [ADR-0001: Use Kotlin and Spring Boot for Backend](./0001-use-kotlin-spring-boot.md)
- [ADR-0002: Use Next.js and React for Frontend](./0002-use-nextjs-react.md)
- [Coding Style Guide](../conventions/coding-style.md) - Detailed linting configuration
- [Conventions README](../conventions/README.md) - Overview of all conventions

---

[Back to Architecture Decisions](./README.md) | [Back to main docs](../README.md)
