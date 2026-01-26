# Contributing to GetSkill Skills

Thanks for contributing! Your skill helps developers save time and avoid trial-and-error.

## Quick Contribution

### 1. Create a skill directory

```bash
mkdir skills/my-skill-name
```

### 2. Create SKILL.md

This is the main skill content that Claude Code will use.

```markdown
# My Skill Name

> One-line description of what this skill does.

## Trigger

When to use this skill:
- User asks to...
- User needs to...

## Prerequisites

- Required tools
- Dependencies

## Instructions

### 1. First Step

Detailed instructions...

### 2. Second Step

More instructions...

## Success Criteria

- [ ] Checklist item 1
- [ ] Checklist item 2

## Common Pitfalls

- Don't do X
- Watch out for Y
```

### 3. Create skill.yaml

Metadata for the skill registry.

```yaml
id: my-skill-name
name: My Skill Name
description: Brief description (under 100 chars)
version: 1.0.0
author: your-github-username
tags:
  - relevant
  - tags
```

### 4. Submit a Pull Request

1. Fork this repo
2. Create a branch: `git checkout -b add-my-skill`
3. Add your skill files
4. Commit: `git commit -m "Add my-skill-name skill"`
5. Push: `git push origin add-my-skill`
6. Open a PR

## Skill Guidelines

### Do

- Make instructions actionable and specific
- Include success criteria (how to verify it worked)
- Add common pitfalls (what to avoid)
- Test with Claude Code before submitting
- Use consistent formatting

### Don't

- Include actual code to copy-paste (skills are instructions, not templates)
- Make skills too broad (one skill = one task)
- Assume context (explain prerequisites)
- Leave placeholder text

## Good Skill Examples

**Good**: `nextjs-landing-page` - Specific task, clear steps, success criteria

**Not ideal**: `web-development` - Too broad, not actionable

## Questions?

Open an issue or reach out at [GetSkill](https://getskill.info).
