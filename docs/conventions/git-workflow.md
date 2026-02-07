---
title: "Git Workflow"
category: "conventions"
tags: ["git", "workflow", "branching", "commits", "pull-requests"]
last_updated: "2026-02-07"
---

# Git Workflow

This document defines the Git branching strategy, commit conventions, and pull request process for the Ultrawork project.

## Branch Strategy

### Main Branches

**`main`**
- Production-ready code
- Protected branch
- All changes via pull requests
- CI must pass before merge
- Triggers deployment workflow on merge

### Feature Branches

Use descriptive prefixes for all feature branches:

**`feature/`** - New features
```bash
feature/user-authentication
feature/task-board-drag-drop
feature/email-notifications
```

**`fix/`** - Bug fixes
```bash
fix/login-redirect-loop
fix/memory-leak-in-websocket
fix/timezone-calculation
```

**`docs/`** - Documentation changes
```bash
docs/api-endpoints
docs/deployment-guide
docs/coding-conventions
```

**`refactor/`** - Code refactoring
```bash
refactor/extract-auth-service
refactor/simplify-state-management
refactor/database-query-optimization
```

**`test/`** - Test additions or modifications
```bash
test/user-service-integration
test/e2e-authentication-flow
```

**`chore/`** - Maintenance tasks
```bash
chore/update-dependencies
chore/configure-ci-pipeline
```

**`ci/`** - CI/CD changes
```bash
ci/add-deploy-workflow
ci/optimize-build-cache
```

### Branch Naming Rules

- Use lowercase with hyphens
- Be descriptive but concise
- Include ticket/issue number if applicable: `feature/ULTRA-123-user-profile`
- Avoid generic names like `feature/updates` or `fix/bugs`

## Commit Message Format

### Structure

```
<type>: <subject>

[optional body]

[optional footer]
```

### Types

Based on project commit history:

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **refactor**: Code refactoring (no functional changes)
- **test**: Test additions or modifications
- **chore**: Maintenance tasks
- **ci**: CI/CD changes
- **perf**: Performance improvements
- **style**: Code style changes (formatting, whitespace)

### Subject Line

- Use imperative mood: "add feature" not "added feature"
- Start with lowercase (unless proper noun)
- No period at the end
- Maximum 72 characters
- Be specific and descriptive

### Examples from Project

**Good commits:**
```
feat(deploy): implement Ansible playbooks for provision, deploy, restart
ci: add manual deploy and restart GitHub Actions workflows
refactor: suppress UnusedParameter warnings in stub implementations
docs: create knowledge base hub scaffold
feat(infra): create Docker Compose stack with Nginx reverse proxy
```

**Scope (optional):**
- Use parentheses to specify component: `feat(auth):`, `fix(api):`, `docs(readme):`
- Common scopes: `auth`, `api`, `ui`, `deploy`, `infra`, `backend`, `frontend`

### Body (when needed)

- Separate from subject with blank line
- Explain **why** not **what** (code shows what)
- Wrap at 72 characters
- Use bullet points for multiple changes

Example:
```
refactor: extract authentication logic into service layer

- Move JWT validation from controller to AuthService
- Improve testability by isolating business logic
- Reduce controller complexity from 150 to 50 lines
```

### Footer (when applicable)

**Breaking changes:**
```
BREAKING CHANGE: API endpoint /auth/login now requires email instead of username
```

**Issue references:**
```
Closes #123
Fixes #456
Related to #789
```

## Pull Request Process

### Before Creating PR

1. **Ensure branch is up to date**
   ```bash
   git checkout main
   git pull origin main
   git checkout feature/your-branch
   git rebase main
   ```

2. **Run all checks locally**
   ```bash
   # Backend
   cd backend
   ./gradlew clean build
   ./gradlew detekt
   ./gradlew test
   
   # Frontend
   cd frontend
   npm run lint:strict
   npm run typecheck
   npm run test
   npm run build
   ```

3. **Verify zero linting errors**
   - Backend: `maxIssues: 0`
   - Frontend: `max-warnings: 0`

4. **Commit all changes**
   - Follow commit message format
   - Make atomic commits (one logical change per commit)

### Creating the PR

**Title Format:**
```
<type>: <brief description>
```

Examples:
- `feat: add user profile editing`
- `fix: resolve login redirect loop`
- `docs: update API documentation`

**Description Template:**

```markdown
## Summary
Brief description of changes and motivation.

## Changes
- Bullet point list of key changes
- Focus on what and why

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Linting passes (zero errors/warnings)

## Screenshots (if UI changes)
[Add screenshots or GIFs]

## Related Issues
Closes #123
```

### PR Requirements

**Must pass before merge:**
- ✅ All CI checks pass
- ✅ Zero linting errors/warnings
- ✅ All tests pass
- ✅ Build succeeds
- ✅ Code review approved
- ✅ No merge conflicts

**CI Pipeline checks:**
- Backend: Gradle build, Detekt, JUnit tests
- Frontend: ESLint, TypeScript, Vitest, build
- Infrastructure: Ansible syntax validation

### Code Review Guidelines

**For Authors:**
- Keep PRs focused and reasonably sized
- Respond to feedback promptly
- Update PR based on review comments
- Re-request review after changes

**For Reviewers:**
- Review within 24 hours
- Check for code quality and adherence to conventions
- Verify tests cover new functionality
- Ensure documentation is updated
- Look for potential bugs or edge cases
- Be constructive and specific in feedback

### Merging

**Merge Strategy:**
- Use "Squash and merge" for feature branches
- Preserve meaningful commit history
- Edit squash commit message if needed

**After Merge:**
1. Delete feature branch
2. Verify deployment workflow triggers
3. Monitor deployment status
4. Verify changes in production

## Deployment Workflow

### Automatic Deployment

When PR is merged to `main`:
1. GitHub Actions workflow triggers
2. Runs infrastructure deployment playbook
3. Deploys backend and frontend
4. Restarts services
5. Verifies health checks

### Manual Deployment

Use GitHub Actions manual workflows:
```
Actions → Deploy Infrastructure → Run workflow
Actions → Restart Services → Run workflow
```

## Common Workflows

### Starting New Feature

```bash
# Update main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/your-feature-name

# Make changes, commit frequently
git add .
git commit -m "feat: add initial implementation"

# Push to remote
git push -u origin feature/your-feature-name

# Create PR on GitHub
```

### Fixing a Bug

```bash
# Create fix branch from main
git checkout main
git pull origin main
git checkout -b fix/bug-description

# Fix bug, add test
git add .
git commit -m "fix: resolve bug description"

# Push and create PR
git push -u origin fix/bug-description
```

### Updating Branch with Main

```bash
# Rebase on main (preferred)
git checkout main
git pull origin main
git checkout feature/your-branch
git rebase main

# Or merge main (if rebase is problematic)
git checkout feature/your-branch
git merge main
```

### Amending Last Commit

```bash
# Make additional changes
git add .
git commit --amend --no-edit

# Force push (only if not reviewed yet)
git push --force-with-lease
```

## Best Practices

### Commits

- **Atomic commits**: One logical change per commit
- **Commit often**: Small, focused commits are easier to review
- **Test before committing**: Ensure code works
- **Write clear messages**: Future you will thank you

### Branches

- **Keep branches short-lived**: Merge within a few days
- **Sync with main regularly**: Avoid large merge conflicts
- **Delete after merge**: Keep repository clean

### Pull Requests

- **Small PRs**: Easier to review, faster to merge
- **Self-review first**: Check your own diff before requesting review
- **Update documentation**: Keep docs in sync with code
- **Add tests**: Don't merge untested code

### Collaboration

- **Communicate**: Use PR comments for discussions
- **Be responsive**: Address feedback promptly
- **Ask questions**: Better to clarify than assume
- **Help others**: Review PRs from teammates

## Troubleshooting

### Merge Conflicts

```bash
# Update your branch
git checkout main
git pull origin main
git checkout feature/your-branch
git rebase main

# Resolve conflicts in editor
# After resolving each file:
git add <resolved-file>
git rebase --continue

# Force push (rebase rewrites history)
git push --force-with-lease
```

### Failed CI Checks

1. Check CI logs for specific errors
2. Run checks locally to reproduce
3. Fix issues and push new commit
4. CI will automatically re-run

### Accidental Commit to Main

```bash
# Create branch from current state
git branch feature/accidental-changes

# Reset main to remote
git checkout main
git reset --hard origin/main

# Continue work on feature branch
git checkout feature/accidental-changes
```

## References

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Best Practices](https://git-scm.com/book/en/v2)
- [GitHub Flow](https://guides.github.com/introduction/flow/)

## Related Documentation

- [Conventions Overview](./README.md) - All coding conventions
- [Coding Style Guide](./coding-style.md) - Code style standards
- [Testing Standards](./testing-standards.md) - Testing practices

---

[Back to Conventions](./README.md) | [Back to main docs](../README.md)
