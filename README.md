# GetSkill Community Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-29-blue.svg)](skills/)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-blueviolet.svg)](https://claude.ai)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-orange.svg)](https://cursor.sh)

**29 ready-to-use skills for AI coding assistants. Copy, paste, done.**

Skip the trial-and-error. Use community-tested skills that actually work.

---

## Table of Contents

- [GetSkill Community Skills](#getskill-community-skills)
  - [Table of Contents](#table-of-contents)
  - [New Here?](#new-here)
  - [Popular Skills](#popular-skills)
  - [Compatibility](#compatibility)
  - [Quick Start](#quick-start)
    - [Claude Code](#claude-code)
    - [Cursor](#cursor)
    - [Done! Now use them:](#done-now-use-them)
  - [All Skills](#all-skills)
    - [Frontend (3)](#frontend-3)
    - [Backend (6)](#backend-6)
    - [Testing (4)](#testing-4)
    - [Productivity (7)](#productivity-7)
    - [Planning \& Workflow (9)](#planning--workflow-9)
  - [Contributing](#contributing)
    - [What Makes a Good Skill?](#what-makes-a-good-skill)
  - [References](#references)
  - [License](#license)

---

## New Here?

**What are skills?** Skills are markdown files that teach AI assistants how to do specific tasks perfectly. Instead of explaining what you want every time, install a skill and get consistent, high-quality results.

**Example:** Instead of "create a landing page with Next.js, use Tailwind, make it responsive..." just install the `nextjs-landing-page` skill and say "create a landing page".

---

## Popular Skills

| Skill | What it does |
|-------|-------------|
| [nextjs-landing-page](skills/nextjs-landing-page) | Generate modern landing pages with Next.js 14 + Tailwind |
| [readme-generator](skills/readme-generator) | Create professional READMEs in seconds |
| [code-review](skills/code-review) | Structured code review with security checklist |
| [react-component](skills/react-component) | Create React components with TypeScript + tests |
| [brainstorming](skills/brainstorming) | Design thinking before implementation |
| [clean-code](skills/clean-code) | Pragmatic coding standards |

[Browse all 29 skills →](skills/)

---

## Compatibility

These skills work with any AI coding assistant that supports SKILL.md files:

| Tool | Install Path | How to Use |
|------|--------------|------------|
| **Claude Code** | `~/.claude/skills/` | Auto-triggered by context |
| **Cursor** | `.cursor/skills/` | Reference `@skill-name` in chat |
| **Gemini CLI** | `.gemini/skills/` | Reference in prompt |
| **Other agents** | Check tool docs | Follow tool's skill format |

---

## Quick Start

### Claude Code

```bash
# Install a single skill
curl -o ~/.claude/skills/nextjs-landing-page.md \
  https://raw.githubusercontent.com/GetSkill-Agent/getskill-skills/main/skills/nextjs-landing-page/SKILL.md

# Install all skills
git clone https://github.com/GetSkill-Agent/getskill-skills.git
cp getskill-skills/skills/*/SKILL.md ~/.claude/skills/
```

### Cursor

```bash
# Install to your project
git clone https://github.com/GetSkill-Agent/getskill-skills.git
mkdir -p .cursor/skills
cp getskill-skills/skills/*/SKILL.md .cursor/skills/
```

### Done! Now use them:

```
"create a landing page"
"review this code"
"generate a readme"
```

---

## All Skills

### Frontend (3)
- [nextjs-landing-page](skills/nextjs-landing-page) - Landing pages with Next.js 14
- [react-component](skills/react-component) - React components with TypeScript
- [react-form-validation](skills/react-form-validation) - Form validation patterns

### Backend (6)
- [nextjs-api-route](skills/nextjs-api-route) - Next.js API routes
- [nextjs-server-actions](skills/nextjs-server-actions) - Server actions patterns
- [api-integration](skills/api-integration) - Third-party API integration
- [prisma-crud](skills/prisma-crud) - Prisma models + CRUD
- [database-schema](skills/database-schema) - Database schema design
- [zod-validation](skills/zod-validation) - Zod schemas for validation

### Testing (4)
- [jest-unit-test](skills/jest-unit-test) - Unit tests with Jest
- [react-testing-library](skills/react-testing-library) - React component tests
- [playwright-e2e](skills/playwright-e2e) - End-to-end tests
- [test-strategy](skills/test-strategy) - Test planning and strategy

### Productivity (7)
- [readme-generator](skills/readme-generator) - Professional READMEs
- [code-review](skills/code-review) - Structured code reviews
- [git-commit-message](skills/git-commit-message) - Commit message guidelines
- [error-debugging](skills/error-debugging) - Debug errors systematically
- [typescript-refactor](skills/typescript-refactor) - Refactoring patterns
- [copywriting](skills/copywriting) - Marketing copy and content
- [tanstack-query](skills/tanstack-query) - TanStack Query patterns

### Planning & Workflow (9)
- [brainstorming](skills/brainstorming) - Design thinking before implementation
- [concise-planning](skills/concise-planning) - Turn requests into atomic checklists
- [plan-writing](skills/plan-writing) - Multi-step task planning with dependencies
- [lint-and-validate](skills/lint-and-validate) - Quality control after code changes
- [verification-before-completion](skills/verification-before-completion) - Pre-merge verification checklist
- [clean-code](skills/clean-code) - Pragmatic coding standards
- [architecture-decision-records](skills/architecture-decision-records) - Document technical decisions
- [receiving-code-review](skills/receiving-code-review) - Handle review feedback effectively
- [requesting-code-review](skills/requesting-code-review) - Ask for code review at the right time

---

## Contributing

Have a skill that works? Share it.

1. Fork this repo
2. Add your skill to `skills/your-skill-name/`
3. Include `SKILL.md` and `skill.yaml`
4. Open a Pull Request

See [Contributing Guide](CONTRIBUTING.md) for details.

### What Makes a Good Skill?

- **Actionable** - Step-by-step instructions, not just information
- **Tested** - Works with Claude Code in real projects
- **Focused** - Does one thing well
- **Complete** - Includes success criteria and common pitfalls

## FAQ

### What is SKILL.md?

SKILL.md is a markdown file that teaches AI coding agents (like Claude Code, Cursor, Windsurf) how to complete specific tasks. It's like a recipe: the agent reads the instructions and follows them step-by-step.

### Where do I put SKILL.md files?

For **Claude Code**: `~/.claude/skills/skill-name/SKILL.md` (global) or `.claude/skills/` (per-project)

For **Cursor**: `.cursor/skills/skill-name/SKILL.md`

### How is SKILL.md different from CLAUDE.md?

- **CLAUDE.md** = Project-wide context and rules (always active)
- **SKILL.md** = Task-specific instructions (triggered on-demand)

Think of CLAUDE.md as "background knowledge" and SKILL.md as "recipes to follow".

### What is skill.yaml?

An optional metadata file that accompanies SKILL.md:

```yaml
id: my-skill-name
description: What this skill does
version: 1.0.0
tags: [frontend, react]
```

### Which AI agents support SKILL.md?

Any agent that reads markdown instructions:
- Claude Code (Anthropic)
- Cursor
- Windsurf (Codeium)
- Cline
- Aider
- Continue

### How do I create my own skill?

1. Create a folder: `skills/my-skill-name/`
2. Add `SKILL.md` with: Trigger Conditions, Steps, Success Criteria
3. Add `skill.yaml` with metadata
4. Test it in your project
5. Share via PR or [skill-share](https://github.com/GetSkill-Agent/skill-share)

### Can I use skills offline?

Yes! Once downloaded, skills work without internet (the AI agent may still need connectivity).

---

## References

This repository includes skills adapted from:
- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) - A large catalog of 550+ agentic skills for AI coding assistants

---

## License

MIT - All skills are open for everyone to use.
