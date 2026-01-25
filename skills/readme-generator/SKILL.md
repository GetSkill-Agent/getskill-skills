# README Generator

> Create professional, comprehensive README files for any project.

## Trigger

- User asks to create or improve a README
- User needs project documentation
- New project needs a README.md

## Structure Template

```markdown
# Project Name

> One-line description (hook)

Brief 2-3 sentence description of what the project does and why it matters.

## Features

- Feature 1
- Feature 2
- Feature 3

## Quick Start

[Installation and basic usage in < 5 steps]

## Installation

[Detailed installation instructions]

## Usage

[Code examples]

## API Reference (if applicable)

[Endpoints or function documentation]

## Configuration

[Environment variables, config files]

## Contributing

[How to contribute]

## License

[License type]
```

## Instructions

### 1. Project Name & Description

**Name**: Use the actual project name, not a description
- ✅ `next-auth`
- ❌ `Next.js Authentication Library`

**One-liner**: Complete this sentence: "This project helps you ___"
- ✅ "Authentication for Next.js made simple"
- ❌ "A library for authentication"

**Description** (2-3 sentences):
1. What it does
2. Who it's for
3. Why it's better/different

### 2. Features Section

List 3-6 key features with concise descriptions:

```markdown
## Features

- **Fast** - Loads in under 100ms
- **Type-safe** - Full TypeScript support
- **Lightweight** - Only 2kb gzipped
```

### 3. Quick Start

Get users running in < 5 steps:

```markdown
## Quick Start

1. Install: `npm install project-name`
2. Import: `import { thing } from 'project-name'`
3. Use:
   ```js
   const result = thing.doSomething();
   ```
```

### 4. Installation Section

Cover all package managers:

```markdown
## Installation

npm:
\`\`\`bash
npm install project-name
\`\`\`

yarn:
\`\`\`bash
yarn add project-name
\`\`\`

pnpm:
\`\`\`bash
pnpm add project-name
\`\`\`
```

### 5. Usage Section

Provide progressive examples:

1. **Basic usage** - Simplest case
2. **Common patterns** - Frequent use cases
3. **Advanced usage** - Power user features

Each example should be copy-paste ready.

### 6. API Reference (for libraries)

Document each public function:

```markdown
### `functionName(param1, param2)`

Description of what it does.

**Parameters:**
- `param1` (string): Description
- `param2` (number, optional): Description. Default: `10`

**Returns:** `Promise<Result>`

**Example:**
\`\`\`js
const result = await functionName('value', 20);
\`\`\`
```

### 7. Configuration

Document all config options:

```markdown
## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `port` | number | `3000` | Server port |
| `debug` | boolean | `false` | Enable debug mode |
```

Or for environment variables:

```markdown
## Environment Variables

Create a `.env` file:

\`\`\`env
DATABASE_URL=postgresql://localhost/mydb
API_KEY=your-api-key
DEBUG=false
\`\`\`
```

### 8. Contributing Section

```markdown
## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing`)
5. Open a Pull Request
```

### 9. License

```markdown
## License

MIT © [Your Name](https://github.com/yourname)
```

## Badges (Optional)

Add at the top, after the title:

```markdown
![npm version](https://img.shields.io/npm/v/package-name)
![license](https://img.shields.io/npm/l/package-name)
![downloads](https://img.shields.io/npm/dm/package-name)
```

## Success Criteria

- [ ] Clear one-liner description
- [ ] Quick Start gets users running in < 5 steps
- [ ] All code examples are copy-paste ready
- [ ] Installation covers major package managers
- [ ] License is specified

## Common Pitfalls

- Don't assume knowledge (explain prerequisites)
- Don't skip the Quick Start
- Don't use outdated code examples
- Don't forget to specify the license
- Don't leave placeholder text
