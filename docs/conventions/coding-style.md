---
title: "Coding Style Guide"
category: "conventions"
tags: ["coding-style", "detekt", "eslint", "linting", "kotlin", "typescript"]
last_updated: "2026-02-07"
---

# Coding Style Guide

This document defines the coding standards and style conventions for the Ultrawork project.

## General Principles

- **Zero tolerance for linting errors**: Both backend and frontend enforce `maxIssues: 0` and `max-warnings: 0`
- **One class per file**: Maintain clear module boundaries
- **120 character line length**: Consistent across both codebases
- **Explicit over implicit**: Prefer clarity and type safety

## Backend (Kotlin/Spring Boot)

### Technology Stack

- **Kotlin**: 2.0.21
- **Spring Boot**: 3.4.0
- **Java**: 21
- **Linter**: Detekt 1.23.8

### Detekt Configuration

Configuration file: `backend/config/detekt/detekt.yml`

#### Build Rules

```yaml
build:
  maxIssues: 0  # Zero tolerance - build fails on any issue
```

#### Style Rules

**Line Length**
```yaml
MaxLineLength:
  active: true
  maxLineLength: 120
```

**Magic Numbers**
```yaml
MagicNumber:
  active: true
  ignoreNumbers: ['-1', '0', '1', '2']
```
- Extract magic numbers to named constants
- Exceptions: -1, 0, 1, 2 (common indices and flags)

**Return Statements**
```yaml
ReturnCount:
  active: true
  max: 3
```
- Maximum 3 return statements per function
- Refactor complex branching logic if exceeded

#### Complexity Rules

**Cyclomatic Complexity**
```yaml
CyclomaticComplexMethod:
  active: true
  threshold: 15
```

**Method Length**
```yaml
LongMethod:
  active: true
  threshold: 60
```
- Keep methods under 60 lines
- Extract helper methods for complex logic

**Parameter Lists**
```yaml
LongParameterList:
  active: true
  functionThreshold: 6
```
- Maximum 6 parameters per function
- Consider parameter objects for complex signatures

**Functions Per File**
```yaml
TooManyFunctions:
  active: true
  thresholdInFiles: 15
```

#### Naming Conventions

```yaml
naming:
  FunctionNaming:
    active: true
  ClassNaming:
    active: true
  VariableNaming:
    active: true
```

- **Functions**: camelCase (`getUserById`, `validateToken`)
- **Classes**: PascalCase (`UserService`, `AuthController`)
- **Variables**: camelCase (`userId`, `authToken`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`)

#### Formatting

```yaml
formatting:
  active: true
  android: false
  autoCorrect: true
```

- Auto-formatting enabled
- Run `./gradlew detekt` before committing

### Kotlin-Specific Conventions

**Data Classes**
- Use for DTOs and value objects
- Leverage immutability with `val`

**Null Safety**
- Prefer non-nullable types
- Use `?.` and `?:` operators appropriately
- Avoid `!!` operator except when absolutely certain

**Extension Functions**
- Use for utility methods that enhance existing types
- Keep focused and single-purpose

**Coroutines**
- Use for asynchronous operations
- Prefer structured concurrency

## Frontend (TypeScript/Next.js)

### Technology Stack

- **Next.js**: 15.5.12
- **React**: 19.1.0
- **TypeScript**: 5.x
- **Linter**: ESLint 9 with airbnb-typescript

### ESLint Configuration

Configuration file: `frontend/.eslintrc.json`

#### Base Configuration

```json
{
  "extends": [
    "next/core-web-vitals",
    "airbnb-typescript",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ]
}
```

#### Strict Rules

**Line Length**
```json
"max-len": ["error", { "code": 120 }]
```

**Type Safety**
```json
"@typescript-eslint/explicit-function-return-type": "error",
"@typescript-eslint/no-explicit-any": "error",
"@typescript-eslint/no-unused-vars": "error"
```
- All functions must have explicit return types
- No `any` types allowed
- No unused variables

**Code Organization**
```json
"one-var": ["error", "never"],
"max-classes-per-file": ["error", 1]
```
- One variable declaration per statement
- One class per file

**React Conventions**
```json
"react/function-component-definition": ["error", {
  "namedComponents": "arrow-function"
}],
"react/jsx-props-no-spreading": "off"
```
- Named components must use arrow functions
- Props spreading allowed (for component composition)

**Imports**
```json
"import/prefer-default-export": "off"
```
- Named exports preferred over default exports

### TypeScript Conventions

**Type Definitions**
- Define interfaces for object shapes
- Use type aliases for unions and complex types
- Prefer interfaces for extensibility

**Component Props**
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ label, onClick, variant = 'primary' }) => {
  // implementation
};
```

**Hooks**
- Custom hooks must start with `use`
- Keep hooks focused and reusable
- Document complex hook behavior

**State Management**
- Use Zustand for global state
- Keep state close to where it's used
- Prefer local state when possible

### Next.js Patterns

**File Structure**
- `app/` directory for routes (App Router)
- `components/` for reusable components
- `lib/` for utilities and helpers
- `types/` for shared TypeScript types

**Server Components**
- Default to Server Components
- Use Client Components (`'use client'`) only when needed
- Minimize client-side JavaScript

**API Routes**
- Type-safe request/response handling
- Proper error handling and status codes
- Validate input with Zod schemas

## Linting Commands

### Backend
```bash
cd backend
./gradlew detekt          # Run linter
./gradlew detekt --auto-correct  # Auto-fix issues
```

### Frontend
```bash
cd frontend
npm run lint              # Next.js linter
npm run lint:strict       # Strict ESLint (max-warnings 0)
npm run lint:fix          # Auto-fix issues
npm run format            # Format with Prettier
npm run format:check      # Check formatting
npm run typecheck         # TypeScript type checking
```

## Pre-Commit Checklist

- [ ] Run linter and fix all issues
- [ ] Run type checker (frontend)
- [ ] Verify zero warnings/errors
- [ ] Format code consistently
- [ ] Remove unused imports
- [ ] Check for magic numbers (backend)
- [ ] Ensure explicit return types (frontend)

## IDE Configuration

### IntelliJ IDEA / WebStorm

**Kotlin**
- Enable Detekt plugin
- Configure code style to match Detekt rules
- Set line length to 120

**TypeScript**
- Enable ESLint integration
- Enable Prettier integration
- Set line length to 120
- Configure auto-import preferences

### VS Code

**Extensions**
- ESLint
- Prettier
- Kotlin Language
- EditorConfig

**Settings**
```json
{
  "editor.rulers": [120],
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

## References

- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
- [Airbnb TypeScript Style Guide](https://github.com/airbnb/javascript)
- [Next.js Documentation](https://nextjs.org/docs)
- [Detekt Rules](https://detekt.dev/docs/rules/complexity)

## Related Documentation

- [Conventions Overview](./README.md) - All coding conventions
- [Testing Standards](./testing-standards.md) - Testing practices
- [Git Workflow](./git-workflow.md) - Version control standards
- [ADR-0003: Strict Linting](../decisions/0003-strict-linting-zero-tolerance.md) - Linting policy decision

---

[Back to Conventions](./README.md) | [Back to main docs](../README.md)
