# Git Commit Message

> Write clear, consistent git commit messages following conventional commits.

## Trigger

- User needs to write a commit message
- User wants to improve commit message quality
- User is setting up commit conventions

## Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

## Instructions

### 1. Type (Required)

| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code change, no feature/fix |
| `perf` | Performance improvement |
| `test` | Adding tests |
| `chore` | Maintenance tasks |
| `ci` | CI/CD changes |
| `build` | Build system changes |

### 2. Scope (Optional)

Area of the codebase affected:
- `feat(auth)` - authentication module
- `fix(api)` - API routes
- `docs(readme)` - README file

### 3. Description (Required)

**Rules:**
- Start with lowercase
- Use imperative mood ("add" not "added")
- No period at the end
- Max 50 characters

**Good examples:**
- `feat(auth): add OAuth2 login support`
- `fix(api): handle null response from server`
- `docs: update installation instructions`

**Bad examples:**
- `Fixed bug` (vague)
- `feat: Add new feature.` (capitalized, period)
- `updated the login page to fix issue #123` (too long, not imperative)

### 4. Body (Optional)

When to include:
- Complex changes needing explanation
- Breaking changes
- Non-obvious reasoning

**Format:**
```
feat(api): add rate limiting to endpoints

Implement token bucket algorithm with configurable limits.
Default: 100 requests per minute per IP.

This prevents API abuse and ensures fair usage.
```

### 5. Footer (Optional)

**Breaking changes:**
```
feat(api)!: change response format

BREAKING CHANGE: API now returns { data, meta } instead of raw data.
Migration: wrap existing handlers to extract .data property.
```

**Issue references:**
```
fix(auth): resolve session timeout issue

Closes #123
```

## Quick Reference

```
feat:     New feature
fix:      Bug fix
docs:     Documentation
style:    Formatting
refactor: Code restructure
perf:     Performance
test:     Tests
chore:    Maintenance
```

## Examples by Scenario

### Adding a feature
```
feat(users): add password reset functionality
```

### Fixing a bug
```
fix(cart): prevent duplicate items on rapid clicks
```

### Refactoring
```
refactor(api): extract validation into middleware
```

### Documentation
```
docs(api): add rate limit examples to README
```

### Breaking change
```
feat(auth)!: require API key for all endpoints

BREAKING CHANGE: All endpoints now require X-API-Key header.
```

### Multiple related changes
```
feat(dashboard): redesign analytics page

- Add new chart components
- Implement date range picker
- Update data fetching logic
```

## Success Criteria

- [ ] Type is correct for the change
- [ ] Description is in imperative mood
- [ ] Description is under 50 characters
- [ ] Breaking changes are marked with `!`
- [ ] Related issues are referenced

## Common Pitfalls

- Don't mix multiple unrelated changes in one commit
- Don't use vague descriptions ("fix bug", "update code")
- Don't forget to mark breaking changes
- Don't use past tense ("added" → "add")
