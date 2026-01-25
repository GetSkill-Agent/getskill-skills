# Jest Unit Test

> Write reliable unit tests with Jest.

## Trigger

- User needs to write unit tests
- User needs to mock dependencies
- User needs to test async code
- User wants to improve test coverage

## Prerequisites

```bash
npm install -D jest @types/jest ts-jest
npx ts-jest config:init
```

## Instructions

### Step 1: Basic Test Structure

```typescript
// utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function divide(a: number, b: number): number {
  if (b === 0) throw new Error('Cannot divide by zero');
  return a / b;
}
```

```typescript
// utils/math.test.ts
import { add, divide } from './math';

describe('Math utilities', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    it('should handle negative numbers', () => {
      expect(add(-1, 1)).toBe(0);
    });

    it('should handle zero', () => {
      expect(add(0, 5)).toBe(5);
    });
  });

  describe('divide', () => {
    it('should divide two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    it('should throw on division by zero', () => {
      expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
    });
  });
});
```

### Step 2: Common Matchers

```typescript
describe('Jest Matchers', () => {
  // Equality
  it('toBe - strict equality (===)', () => {
    expect(2 + 2).toBe(4);
  });

  it('toEqual - deep equality', () => {
    expect({ a: 1 }).toEqual({ a: 1 });
    expect([1, 2]).toEqual([1, 2]);
  });

  it('toStrictEqual - strict deep equality', () => {
    // Fails if properties are undefined vs missing
    expect({ a: 1 }).toStrictEqual({ a: 1 });
  });

  // Truthiness
  it('truthiness matchers', () => {
    expect(null).toBeNull();
    expect(undefined).toBeUndefined();
    expect(1).toBeDefined();
    expect(true).toBeTruthy();
    expect(0).toBeFalsy();
  });

  // Numbers
  it('number matchers', () => {
    expect(10).toBeGreaterThan(5);
    expect(10).toBeGreaterThanOrEqual(10);
    expect(5).toBeLessThan(10);
    expect(0.1 + 0.2).toBeCloseTo(0.3); // For floats
  });

  // Strings
  it('string matchers', () => {
    expect('Hello World').toMatch(/World/);
    expect('Hello World').toContain('World');
  });

  // Arrays
  it('array matchers', () => {
    expect([1, 2, 3]).toContain(2);
    expect([{ a: 1 }, { a: 2 }]).toContainEqual({ a: 1 });
    expect([1, 2, 3]).toHaveLength(3);
  });

  // Objects
  it('object matchers', () => {
    expect({ a: 1, b: 2 }).toHaveProperty('a');
    expect({ a: 1, b: 2 }).toHaveProperty('a', 1);
    expect({ a: { b: 1 } }).toHaveProperty('a.b', 1);
    expect({ a: 1 }).toMatchObject({ a: 1 });
  });

  // Exceptions
  it('exception matchers', () => {
    expect(() => { throw new Error('fail'); }).toThrow();
    expect(() => { throw new Error('fail'); }).toThrow('fail');
    expect(() => { throw new Error('fail'); }).toThrow(Error);
  });

  // Negation
  it('negation with .not', () => {
    expect(5).not.toBe(10);
    expect([1, 2]).not.toContain(3);
  });
});
```

### Step 3: Async Testing

```typescript
// services/api.ts
export async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}
```

```typescript
// services/api.test.ts
describe('API Service', () => {
  // Using async/await (preferred)
  it('should fetch user', async () => {
    const user = await fetchUser('123');
    expect(user).toHaveProperty('id', '123');
  });

  // Using .resolves/.rejects
  it('should fetch user with resolves', async () => {
    await expect(fetchUser('123')).resolves.toHaveProperty('id');
  });

  it('should reject for invalid user', async () => {
    await expect(fetchUser('invalid')).rejects.toThrow('User not found');
  });

  // Testing callbacks (if needed)
  it('should work with callbacks', (done) => {
    fetchUserCallback('123', (err, user) => {
      expect(err).toBeNull();
      expect(user.id).toBe('123');
      done();
    });
  });
});
```

### Step 4: Mocking Functions

```typescript
describe('Mocking', () => {
  // Mock function
  it('should track calls', () => {
    const mockFn = jest.fn();

    mockFn('arg1', 'arg2');
    mockFn('arg3');

    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
    expect(mockFn).toHaveBeenLastCalledWith('arg3');
  });

  // Mock return values
  it('should mock return values', () => {
    const mockFn = jest.fn()
      .mockReturnValue('default')
      .mockReturnValueOnce('first')
      .mockReturnValueOnce('second');

    expect(mockFn()).toBe('first');
    expect(mockFn()).toBe('second');
    expect(mockFn()).toBe('default');
  });

  // Mock implementation
  it('should mock implementation', () => {
    const mockFn = jest.fn((x: number) => x * 2);

    expect(mockFn(5)).toBe(10);
  });

  // Mock async functions
  it('should mock async functions', async () => {
    const mockFn = jest.fn().mockResolvedValue({ id: '123' });

    const result = await mockFn();
    expect(result).toEqual({ id: '123' });
  });

  // Mock rejected promise
  it('should mock rejected promises', async () => {
    const mockFn = jest.fn().mockRejectedValue(new Error('Failed'));

    await expect(mockFn()).rejects.toThrow('Failed');
  });
});
```

### Step 5: Mocking Modules

```typescript
// __mocks__/axios.ts (manual mock)
export default {
  get: jest.fn(),
  post: jest.fn(),
};
```

```typescript
// services/userService.test.ts
import axios from 'axios';
import { getUser } from './userService';

// Auto-mock entire module
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should fetch user', async () => {
    mockedAxios.get.mockResolvedValue({
      data: { id: '123', name: 'John' },
    });

    const user = await getUser('123');

    expect(mockedAxios.get).toHaveBeenCalledWith('/api/users/123');
    expect(user).toEqual({ id: '123', name: 'John' });
  });
});

// Partial module mock
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  fetchData: jest.fn(),
}));
```

### Step 6: Spying on Methods

```typescript
describe('Spies', () => {
  it('should spy on object method', () => {
    const video = {
      play: () => true,
      pause: () => false,
    };

    const spy = jest.spyOn(video, 'play');

    video.play();

    expect(spy).toHaveBeenCalled();
    expect(spy).toHaveReturnedWith(true);

    spy.mockRestore(); // Restore original implementation
  });

  it('should spy and mock implementation', () => {
    const spy = jest.spyOn(console, 'log').mockImplementation(() => {});

    console.log('test');

    expect(spy).toHaveBeenCalledWith('test');

    spy.mockRestore();
  });
});
```

### Step 7: Setup and Teardown

```typescript
describe('Database Tests', () => {
  let db: Database;

  // Run once before all tests
  beforeAll(async () => {
    db = await Database.connect();
  });

  // Run once after all tests
  afterAll(async () => {
    await db.disconnect();
  });

  // Run before each test
  beforeEach(async () => {
    await db.clear();
    await db.seed();
  });

  // Run after each test
  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should create user', async () => {
    const user = await db.createUser({ name: 'John' });
    expect(user).toHaveProperty('id');
  });
});
```

### Step 8: Testing Error Handling

```typescript
describe('Error Handling', () => {
  it('should throw specific error', () => {
    expect(() => {
      throw new Error('Specific error');
    }).toThrow('Specific error');
  });

  it('should throw error type', () => {
    class ValidationError extends Error {}

    expect(() => {
      throw new ValidationError('Invalid');
    }).toThrow(ValidationError);
  });

  it('should handle async errors', async () => {
    const asyncFn = async () => {
      throw new Error('Async error');
    };

    await expect(asyncFn()).rejects.toThrow('Async error');
  });

  it('should catch and verify error properties', async () => {
    try {
      await failingFunction();
      fail('Should have thrown');
    } catch (error) {
      expect(error).toBeInstanceOf(CustomError);
      expect((error as CustomError).code).toBe('INVALID');
    }
  });
});
```

### Step 9: Snapshot Testing

```typescript
describe('Snapshot Tests', () => {
  it('should match object snapshot', () => {
    const user = {
      id: '123',
      name: 'John',
      createdAt: new Date('2024-01-01'),
    };

    expect(user).toMatchSnapshot();
  });

  it('should match inline snapshot', () => {
    const result = formatUser({ id: '1', name: 'John' });

    expect(result).toMatchInlineSnapshot(`
      {
        "displayName": "John",
        "id": "1",
      }
    `);
  });

  // Ignore dynamic values
  it('should match with property matchers', () => {
    const user = createUser({ name: 'John' });

    expect(user).toMatchSnapshot({
      id: expect.any(String),
      createdAt: expect.any(Date),
    });
  });
});
```

### Step 10: Test Organization

```typescript
// __tests__/userService.test.ts
describe('UserService', () => {
  describe('getUser', () => {
    describe('when user exists', () => {
      it('should return user data', async () => {});
      it('should include profile', async () => {});
    });

    describe('when user does not exist', () => {
      it('should throw NotFoundError', async () => {});
    });
  });

  describe('createUser', () => {
    describe('with valid data', () => {
      it('should create user', async () => {});
      it('should send welcome email', async () => {});
    });

    describe('with invalid data', () => {
      it('should throw ValidationError', async () => {});
    });
  });
});

// Use .only for focused testing during development
describe.only('Focused tests', () => {
  it.only('runs only this test', () => {});
});

// Use .skip to skip tests
describe.skip('Skipped tests', () => {
  it('is skipped', () => {});
});
```

## Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/*.test.ts'],
  collectCoverageFrom: ['src/**/*.ts', '!src/**/*.d.ts'],
  coverageThreshold: {
    global: { branches: 80, functions: 80, lines: 80, statements: 80 },
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};
```

## Success Criteria

- [ ] Tests are isolated (no shared state)
- [ ] Mocks are reset between tests
- [ ] Async code is properly awaited
- [ ] Edge cases are covered
- [ ] Error cases are tested
- [ ] Coverage meets threshold

## Common Pitfalls

- Don't forget to await async assertions
- Don't share state between tests
- Don't forget jest.clearAllMocks() in beforeEach
- Don't test implementation details, test behavior
- Don't forget to mockRestore() after spying
- Don't use real network/database in unit tests
