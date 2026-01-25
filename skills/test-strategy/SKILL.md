# Test Strategy

> Design and implement effective testing strategies for software projects.

## Trigger

- User needs to design a testing approach
- User needs to improve test coverage
- User needs to set up testing in CI/CD
- User needs to balance test types

## Instructions

### Step 1: The Test Pyramid

```
                    /\
                   /  \
                  / E2E\         Slow, Expensive, Few
                 /------\
                /        \
               / Integration\    Medium Speed, Medium Cost
              /--------------\
             /                \
            /    Unit Tests    \  Fast, Cheap, Many
           /____________________\
```

**Recommended Ratio:**
- Unit Tests: 70%
- Integration Tests: 20%
- E2E Tests: 10%

### Step 2: Test Types and When to Use

| Test Type | Scope | Speed | Cost | When to Use |
|-----------|-------|-------|------|-------------|
| Unit | Single function/class | Fast (ms) | Low | Pure logic, utilities, transformations |
| Integration | Multiple units together | Medium (s) | Medium | API routes, database queries, services |
| E2E | Full user flow | Slow (min) | High | Critical paths, happy paths |
| Smoke | Basic functionality | Fast | Low | After deployment, quick sanity check |
| Regression | Previously broken | Varies | Medium | Bug fixes, prevent recurrence |
| Performance | Load/stress | Slow | High | Before major releases |

### Step 3: What to Test at Each Level

**Unit Tests:**
```typescript
// Test pure functions
describe('calculatePrice', () => {
  it('applies discount correctly', () => {
    expect(calculatePrice(100, 0.1)).toBe(90);
  });

  it('handles edge cases', () => {
    expect(calculatePrice(0, 0.5)).toBe(0);
    expect(calculatePrice(100, 0)).toBe(100);
    expect(calculatePrice(100, 1)).toBe(0);
  });

  it('throws on invalid discount', () => {
    expect(() => calculatePrice(100, 1.5)).toThrow();
    expect(() => calculatePrice(100, -0.1)).toThrow();
  });
});

// Test class methods
describe('UserService', () => {
  it('validates email format', () => {
    expect(UserService.isValidEmail('test@example.com')).toBe(true);
    expect(UserService.isValidEmail('invalid')).toBe(false);
  });
});

// Test transformations
describe('formatUser', () => {
  it('returns formatted user object', () => {
    const input = { first_name: 'John', last_name: 'Doe' };
    expect(formatUser(input)).toEqual({
      fullName: 'John Doe',
      initials: 'JD',
    });
  });
});
```

**Integration Tests:**
```typescript
// Test API routes with database
describe('POST /api/users', () => {
  beforeEach(async () => {
    await db.user.deleteMany();
  });

  it('creates user in database', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Test' });

    expect(response.status).toBe(201);

    const user = await db.user.findUnique({
      where: { email: 'test@example.com' },
    });
    expect(user).toBeTruthy();
  });

  it('returns 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'invalid', name: 'Test' });

    expect(response.status).toBe(400);
  });
});

// Test service with external dependencies
describe('PaymentService', () => {
  it('processes payment through Stripe', async () => {
    // Use test API keys
    const result = await paymentService.charge({
      amount: 1000,
      currency: 'usd',
      source: 'tok_visa',
    });

    expect(result.status).toBe('succeeded');
  });
});
```

**E2E Tests:**
```typescript
// Test critical user flows
describe('Checkout Flow', () => {
  test('user can complete purchase', async ({ page }) => {
    // Add product to cart
    await page.goto('/products');
    await page.getByRole('button', { name: 'Add to Cart' }).first().click();

    // Go to checkout
    await page.getByRole('link', { name: 'Cart' }).click();
    await page.getByRole('button', { name: 'Checkout' }).click();

    // Fill shipping
    await page.getByLabel('Address').fill('123 Main St');
    await page.getByLabel('City').fill('New York');
    await page.getByRole('button', { name: 'Continue' }).click();

    // Fill payment
    await page.getByLabel('Card Number').fill('4242424242424242');
    await page.getByRole('button', { name: 'Pay' }).click();

    // Verify success
    await expect(page.getByText('Order Confirmed')).toBeVisible();
  });
});
```

### Step 4: Coverage Strategy

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/types/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Higher thresholds for critical code
    './src/services/payment/**': {
      branches: 95,
      functions: 95,
      lines: 95,
    },
    // Lower thresholds for UI components
    './src/components/**': {
      branches: 60,
      functions: 60,
      lines: 60,
    },
  },
};
```

**What to Focus Coverage On:**

| Area | Target | Priority | Reason |
|------|--------|----------|--------|
| Business logic | 95%+ | High | Core value, must be correct |
| API routes | 90%+ | High | User-facing, many edge cases |
| Utilities | 90%+ | Medium | Widely used, easy to test |
| Services | 85%+ | High | Integration points |
| Components | 60-80% | Medium | Visual, harder to test |
| Config/types | 0% | Skip | No logic to test |

### Step 5: Test Organization

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx      # Unit tests
│   │   └── Button.stories.tsx   # Storybook (visual tests)
│   └── Form/
│       ├── Form.tsx
│       └── Form.test.tsx
├── services/
│   ├── user.service.ts
│   └── user.service.test.ts
├── utils/
│   ├── format.ts
│   └── format.test.ts
└── __tests__/                    # Integration tests
    ├── api/
    │   ├── users.test.ts
    │   └── posts.test.ts
    └── setup.ts

tests/                            # E2E tests (separate folder)
├── fixtures/
│   ├── index.ts
│   └── auth.ts
├── pages/
│   ├── LoginPage.ts
│   └── DashboardPage.ts
├── auth.spec.ts
├── checkout.spec.ts
└── playwright.config.ts
```

### Step 6: CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run test:unit -- --coverage

      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test

      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps

      - run: npm run build
      - run: npm run test:e2e

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### Step 7: Test Prioritization

**Run Tests in Order of Speed:**

```json
// package.json
{
  "scripts": {
    "test": "npm run test:unit && npm run test:integration && npm run test:e2e",
    "test:unit": "jest --testPathPattern='\\.test\\.ts'",
    "test:integration": "jest --testPathPattern='__tests__'",
    "test:e2e": "playwright test",
    "test:watch": "jest --watch",
    "test:ci": "npm run test:unit -- --coverage && npm run test:integration"
  }
}
```

**PR Checks vs Deploy Checks:**

| Stage | Tests | Time Budget |
|-------|-------|-------------|
| Pre-commit | Linting, type check | < 30s |
| PR checks | Unit + Integration | < 5 min |
| Pre-deploy | All tests | < 15 min |
| Post-deploy | Smoke tests | < 2 min |
| Nightly | Full E2E + performance | Unlimited |

### Step 8: Testing Checklist by Feature Type

**New Feature:**
- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints
- [ ] E2E test for happy path
- [ ] Edge case tests (empty, null, max values)
- [ ] Error handling tests

**Bug Fix:**
- [ ] Regression test reproducing bug
- [ ] Fix the bug
- [ ] Verify regression test passes
- [ ] Add edge case tests if missing

**Refactoring:**
- [ ] Ensure existing tests pass before refactoring
- [ ] Run tests after each change
- [ ] No new tests needed (behavior unchanged)
- [ ] Update tests if interface changes

**API Change:**
- [ ] Contract tests (request/response schema)
- [ ] Backward compatibility tests
- [ ] Integration tests with consumers
- [ ] Documentation updates

### Step 9: Test Data Strategy

```typescript
// factories/user.factory.ts
import { faker } from '@faker-js/faker';

export function createUser(overrides?: Partial<User>): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    createdAt: faker.date.past(),
    ...overrides,
  };
}

export function createUsers(count: number): User[] {
  return Array.from({ length: count }, () => createUser());
}

// Usage in tests
describe('UserService', () => {
  it('finds user by email', async () => {
    const user = createUser({ email: 'specific@example.com' });
    await db.user.create({ data: user });

    const found = await userService.findByEmail('specific@example.com');
    expect(found).toMatchObject(user);
  });
});
```

### Step 10: When NOT to Test

**Skip Testing:**
- Third-party library internals
- Framework behavior (React rendering, Next.js routing)
- Simple getters/setters with no logic
- Type definitions
- Constants
- Configuration files

**Test Sparingly:**
- UI layout/styling (use visual regression)
- External API responses (mock them)
- Randomized behavior (seed random)
- Time-dependent code (mock time)

## Testing Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Testing implementation | Breaks on refactor | Test behavior/outcomes |
| Shared test state | Flaky tests | Isolate each test |
| Testing private methods | Coupling to internals | Test through public API |
| Too many mocks | Tests prove nothing | Use real dependencies when possible |
| Slow unit tests | Long feedback loop | Move slow tests to integration |
| No assertions | False confidence | Always assert expected outcomes |

## Success Criteria

- [ ] Test pyramid is balanced (70/20/10)
- [ ] Critical paths have E2E coverage
- [ ] Business logic has 90%+ unit coverage
- [ ] Tests run in CI on every PR
- [ ] Test failures block merges

## Common Pitfalls

- Don't aim for 100% coverage (diminishing returns)
- Don't write tests after the fact only (TDD helps design)
- Don't ignore flaky tests (fix or remove)
- Don't mock everything (lose confidence)
- Don't skip testing error paths
- Don't forget to test edge cases
