---
title: "Ultrawork Coding Conventions"
category: "conventions"
tags: ["coding-standards", "best-practices", "conventions", "style-guide"]
last_updated: "2026-02-07"
---

# Ultrawork Coding Conventions

This directory contains the coding standards, workflows, and best practices for the Ultrawork project.

## Overview

Ultrawork is a full-stack application with strict quality standards enforced through automated tooling:

- **Backend**: Kotlin 2.0.21 + Spring Boot 3.4.0 with Detekt (zero issues policy)
- **Frontend**: TypeScript + Next.js 15 with ESLint (zero warnings policy)
- **Testing**: JUnit 5 (backend) and Vitest (frontend)
- **CI/CD**: GitHub Actions with automated deployment

## Quick Reference

### Zero Tolerance Policy

Both codebases enforce strict linting:
- Backend: `maxIssues: 0` (Detekt)
- Frontend: `max-warnings: 0` (ESLint)

**No code merges without passing all checks.**

### Line Length

Maximum **120 characters** across all code.

### One Class Per File

Both backend and frontend enforce one class per file for maintainability.

### Explicit Types

- **Backend**: Kotlin type inference allowed but encouraged for clarity
- **Frontend**: Explicit function return types required (`@typescript-eslint/explicit-function-return-type`)

## Documentation

### [Coding Style Guide](./coding-style.md)

Comprehensive style standards for both backend and frontend:

**Backend (Kotlin)**:
- Detekt configuration and rules
- Complexity thresholds (cyclomatic complexity ≤15, methods ≤60 lines)
- Naming conventions (camelCase, PascalCase, UPPER_SNAKE_CASE)
- Magic number restrictions
- Formatting and auto-correction

**Frontend (TypeScript/Next.js)**:
- ESLint configuration with airbnb-typescript
- Type safety requirements (no `any`, explicit return types)
- React component patterns (arrow functions for named components)
- Import/export conventions
- Next.js App Router patterns

**Key Topics**:
- Technology stack versions
- Linting rules and enforcement
- IDE configuration
- Pre-commit checklist

### [Git Workflow](./git-workflow.md)

Branch strategy, commit conventions, and pull request process:

**Branch Naming**:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation
- `refactor/` - Code refactoring
- `test/` - Test changes
- `chore/` - Maintenance
- `ci/` - CI/CD changes

**Commit Format**:
```
<type>: <subject>

[optional body]

[optional footer]
```

**PR Requirements**:
- All CI checks pass
- Zero linting errors/warnings
- All tests pass
- Code review approved
- No merge conflicts

**Key Topics**:
- Commit message examples from project history
- PR description template
- Code review guidelines
- Deployment workflow
- Common workflows and troubleshooting

### [Testing Standards](./testing-standards.md)

Testing requirements, conventions, and best practices:

**Backend Testing**:
- JUnit 5 + MockK + Testcontainers
- Unit tests, integration tests, controller tests
- Backtick test naming: `` `should add token to blacklist` ``
- Current coverage: 8 test files

**Frontend Testing**:
- Vitest + Testing Library + Playwright
- Component tests, hook tests, E2E tests
- Accessible query priority (getByRole, getByLabelText)
- Current coverage: 1 test file

**Coverage Goals**:
- Controllers/Components: 80%+
- Services/Hooks: 90%+
- Utilities: 95%+

**Key Topics**:
- Test structure and organization
- Mocking strategies
- Running tests locally and in CI
- Best practices by test type

## Quick Start

### Setting Up Your Environment

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd ultrawork
   ```

2. **Backend setup**
   ```bash
   cd backend
   ./gradlew build
   ./gradlew detekt
   ```

3. **Frontend setup**
   ```bash
   cd frontend
   npm install
   npm run lint:strict
   npm run typecheck
   ```

### Before Every Commit

**Backend**:
```bash
cd backend
./gradlew detekt          # Must pass with 0 issues
./gradlew test            # All tests must pass
./gradlew build           # Build must succeed
```

**Frontend**:
```bash
cd frontend
npm run lint:strict       # Must pass with 0 warnings
npm run typecheck         # No type errors
npm run test              # All tests must pass
npm run build             # Build must succeed
```

### Creating a Pull Request

1. Create feature branch: `git checkout -b feature/your-feature`
2. Make changes following coding standards
3. Run all checks locally (linting, tests, build)
4. Commit with conventional format: `feat: add user profile`
5. Push and create PR with description
6. Wait for CI checks and code review
7. Merge when approved and all checks pass

## Enforcement

### Automated Checks

**Pre-merge requirements** (enforced by CI):
- ✅ Detekt: 0 issues
- ✅ ESLint: 0 warnings
- ✅ TypeScript: 0 errors
- ✅ Tests: All passing
- ✅ Build: Success

### Code Review

All PRs require:
- Code review approval
- Adherence to style guide
- Appropriate test coverage
- Updated documentation (if applicable)

### Continuous Integration

GitHub Actions runs on every push:
- Linting (backend and frontend)
- Type checking (frontend)
- Tests (backend and frontend)
- Build verification

**Deployment** triggers automatically on merge to `main`.

## Tools and Configuration

### Backend

- **Build**: Gradle 8.x
- **Linter**: Detekt 1.23.8
- **Config**: `backend/config/detekt/detekt.yml`
- **Tests**: JUnit 5, MockK, Testcontainers

### Frontend

- **Build**: Next.js 15.5.12
- **Linter**: ESLint 9 with airbnb-typescript
- **Config**: `frontend/.eslintrc.json`
- **Tests**: Vitest 4.0.18, Playwright 1.58.2

### IDE Recommendations

**IntelliJ IDEA / WebStorm**:
- Detekt plugin
- ESLint integration
- Prettier integration
- Line length ruler at 120

**VS Code**:
- ESLint extension
- Prettier extension
- Kotlin extension
- EditorConfig support

## Contributing

### For New Contributors

1. Read all convention documents
2. Set up your development environment
3. Configure your IDE with project settings
4. Run checks locally before pushing
5. Follow PR template and guidelines

### For Maintainers

1. Enforce conventions in code reviews
2. Update conventions as project evolves
3. Keep tooling configurations in sync
4. Document new patterns and decisions

## Project Structure

```
ultrawork/
├── backend/              # Kotlin/Spring Boot API
│   ├── src/
│   │   ├── main/kotlin/
│   │   └── test/kotlin/
│   ├── config/detekt/
│   └── build.gradle.kts
├── frontend/             # TypeScript/Next.js UI
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── __tests__/
│   ├── .eslintrc.json
│   └── package.json
├── infrastructure/       # Ansible, Docker, Nginx
├── docs/                 # Documentation
│   └── conventions/      # This directory
└── .github/workflows/    # CI/CD pipelines
```

## Version History

- **v1.0** (2026-02-07): Initial conventions documentation
  - Coding style guide
  - Git workflow
  - Testing standards

## Questions?

If you have questions about conventions:
1. Check the relevant documentation file
2. Search project history for examples
3. Ask in code review or team discussion
4. Propose updates via PR if conventions are unclear

## Related Documentation

- [Architecture Decisions](../decisions/README.md) - Technical decision records
- [API Documentation](../api/README.md) - Backend API reference
- [Architecture](../architecture/README.md) - System design and components
- [Deployment](../deployment/README.md) - Infrastructure and deployment guide
- [Guides](../guides/README.md) - Getting started and troubleshooting

---

**Remember**: These conventions exist to maintain code quality and team velocity. When in doubt, prioritize clarity and consistency.

[Back to main docs](../README.md)
