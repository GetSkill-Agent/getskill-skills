# GetSkill Community Skills

**20+ ready-to-use skills for Claude Code. Copy, paste, done.**

Skip the trial-and-error. Use community-tested skills that actually work.

## Popular Skills

| Skill | What it does |
|-------|-------------|
| [nextjs-landing-page](skills/nextjs-landing-page) | Generate modern landing pages with Next.js 14 + Tailwind |
| [readme-generator](skills/readme-generator) | Create professional READMEs in seconds |
| [code-review](skills/code-review) | Structured code review with security checklist |
| [react-component](skills/react-component) | Create React components with TypeScript + tests |
| [prisma-crud](skills/prisma-crud) | Generate Prisma models with CRUD operations |
| [jest-unit-test](skills/jest-unit-test) | Write comprehensive unit tests |

[Browse all 20+ skills →](skills/)

## Quick Start (30 seconds)

**Option 1: Install a single skill**

```bash
# Example: Install the landing page skill
curl -o ~/.claude/skills/nextjs-landing-page.md \
  https://raw.githubusercontent.com/GetSkill-Agent/getskill-skills/main/skills/nextjs-landing-page/SKILL.md
```

**Option 2: Install all skills**

```bash
# Clone and copy all
git clone https://github.com/GetSkill-Agent/getskill-skills.git
cp getskill-skills/skills/*/SKILL.md ~/.claude/skills/
```

Done. Now use them:

```
"create a landing page"
"review this code"
"generate a readme"
```

## All Skills

### Frontend
- [nextjs-landing-page](skills/nextjs-landing-page) - Landing pages with Next.js 14
- [react-component](skills/react-component) - React components with TypeScript
- [react-form-validation](skills/react-form-validation) - Form validation patterns

### Backend
- [nextjs-api-route](skills/nextjs-api-route) - Next.js API routes
- [nextjs-server-actions](skills/nextjs-server-actions) - Server actions patterns
- [api-integration](skills/api-integration) - Third-party API integration
- [prisma-crud](skills/prisma-crud) - Prisma models + CRUD
- [database-schema](skills/database-schema) - Database schema design
- [zod-validation](skills/zod-validation) - Zod schemas for validation

### Testing
- [jest-unit-test](skills/jest-unit-test) - Unit tests with Jest
- [react-testing-library](skills/react-testing-library) - React component tests
- [playwright-e2e](skills/playwright-e2e) - End-to-end tests
- [test-strategy](skills/test-strategy) - Test planning and strategy

### Productivity
- [readme-generator](skills/readme-generator) - Professional READMEs
- [code-review](skills/code-review) - Structured code reviews
- [git-commit-message](skills/git-commit-message) - Commit message guidelines
- [error-debugging](skills/error-debugging) - Debug errors systematically
- [typescript-refactor](skills/typescript-refactor) - Refactoring patterns
- [copywriting](skills/copywriting) - Marketing copy and content
- [tanstack-query](skills/tanstack-query) - TanStack Query patterns

## Contributing

Have a skill that works? Share it.

1. Fork this repo
2. Add your skill to `skills/your-skill-name/`
3. Include `SKILL.md` and `skill.yaml`
4. Open a Pull Request

See [Contributing Guide](CONTRIBUTING.md) for details.

## What Makes a Good Skill?

- **Actionable** - Step-by-step instructions, not just information
- **Tested** - Works with Claude Code in real projects
- **Focused** - Does one thing well
- **Complete** - Includes success criteria and common pitfalls

## License

MIT - All skills are open for everyone to use.
