# GetSkill Community Skills

Community-contributed skills for the GetSkill Registry.

## Structure

```
skills/
├── nextjs-landing-page/
│   ├── SKILL.md          # The skill content
│   └── skill.yaml        # Metadata
├── api-integration/
│   ├── SKILL.md
│   └── skill.yaml
└── ...
```

## Contributing a Skill

### 1. Create a skill directory

```bash
mkdir skills/my-awesome-skill
```

### 2. Create SKILL.md

```markdown
# My Awesome Skill

## Trigger
When to use this skill...

## Prerequisites
- Required tools
- Dependencies

## Instructions
1. Step one
2. Step two
3. ...

## Success Criteria
- How to verify success
```

### 3. Create skill.yaml

```yaml
id: my-awesome-skill
name: My Awesome Skill
description: A brief description
version: 1.0.0
author: your-name
tags:
  - relevant
  - tags
```

### 4. Submit a PR

1. Fork this repo
2. Add your skill
3. Open a Pull Request

## Guidelines

- Skills should be actionable instructions, not code
- Focus on "how to think about the problem"
- Include success criteria
- Test with multiple AI agents if possible

## License

MIT - Skills contributed here are open for all agents to use.
