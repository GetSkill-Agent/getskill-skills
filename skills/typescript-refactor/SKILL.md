# TypeScript Refactoring

> Refactor code for better type safety, readability, and maintainability.

## Trigger

- User wants to improve code quality
- User needs to refactor JavaScript to TypeScript
- User wants better type safety

## Instructions

### 1. Add Proper Types (Remove `any`)

```typescript
// Bad
function processData(data: any) {
  return data.map((item: any) => item.name);
}

// Good
interface User {
  id: string;
  name: string;
  email: string;
}

function processData(data: User[]): string[] {
  return data.map((user) => user.name);
}
```

### 2. Use Type Inference

```typescript
// Unnecessary explicit types
const name: string = 'John';
const numbers: number[] = [1, 2, 3];
const user: { name: string } = { name: 'John' };

// Let TypeScript infer
const name = 'John';
const numbers = [1, 2, 3];
const user = { name: 'John' };

// DO add types for function parameters and return types
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

### 3. Use Union Types Instead of Enums

```typescript
// Old style
enum Status {
  Pending = 'pending',
  Active = 'active',
  Completed = 'completed',
}

// Better: Union type (tree-shakeable, simpler)
type Status = 'pending' | 'active' | 'completed';

// Use as const for objects
const STATUS = {
  Pending: 'pending',
  Active: 'active',
  Completed: 'completed',
} as const;

type Status = typeof STATUS[keyof typeof STATUS];
```

### 4. Use Discriminated Unions

```typescript
// Bad: optional properties
interface Response {
  success: boolean;
  data?: User;
  error?: string;
}

// Good: discriminated union
type Response =
  | { success: true; data: User }
  | { success: false; error: string };

// Usage
function handleResponse(res: Response) {
  if (res.success) {
    // TypeScript knows res.data exists
    console.log(res.data.name);
  } else {
    // TypeScript knows res.error exists
    console.error(res.error);
  }
}
```

### 5. Use Generics for Reusability

```typescript
// Bad: duplicate code
function getFirstUser(users: User[]): User | undefined {
  return users[0];
}

function getFirstProduct(products: Product[]): Product | undefined {
  return products[0];
}

// Good: generic
function getFirst<T>(items: T[]): T | undefined {
  return items[0];
}

// Usage
const firstUser = getFirst(users);    // User | undefined
const firstProduct = getFirst(products);  // Product | undefined
```

### 6. Use Utility Types

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Partial - all properties optional
type UpdateUser = Partial<User>;

// Pick - select specific properties
type UserPreview = Pick<User, 'id' | 'name'>;

// Omit - exclude properties
type PublicUser = Omit<User, 'password'>;

// Required - all properties required
type RequiredUser = Required<User>;

// Readonly - immutable
type ImmutableUser = Readonly<User>;

// Record - key-value mapping
type UserMap = Record<string, User>;
```

### 7. Extract Types from Functions

```typescript
// Get return type of a function
type ApiResponse = ReturnType<typeof fetchUsers>;

// Get parameter types
type FetchParams = Parameters<typeof fetchUsers>;

// Get awaited type (for async functions)
type Users = Awaited<ReturnType<typeof fetchUsers>>;
```

### 8. Use `satisfies` for Type Checking

```typescript
// Problem: loses type inference
const config: Record<string, string> = {
  apiUrl: 'https://api.example.com',
  version: '1.0.0',
};
// config.apiUrl is string, not the literal

// Solution: satisfies
const config = {
  apiUrl: 'https://api.example.com',
  version: '1.0.0',
} satisfies Record<string, string>;
// config.apiUrl is 'https://api.example.com' (literal type)
```

### 9. Proper Null Handling

```typescript
// Bad
function getUser(id: string): User | null | undefined {
  // ...
}

// Good: consistent null handling
function getUser(id: string): User | null {
  // ...
}

// Use nullish coalescing
const name = user?.name ?? 'Unknown';

// Use optional chaining
const street = user?.address?.street;
```

### 10. Type Guards

```typescript
// Type guard function
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

// Usage
function process(data: unknown) {
  if (isUser(data)) {
    // TypeScript knows data is User
    console.log(data.name);
  }
}

// Array filter with type guard
const users = items.filter(isUser); // User[]
```

### 11. Branded Types for Type Safety

```typescript
// Problem: can mix up IDs
function getUser(userId: string) { }
function getOrder(orderId: string) { }

getUser(orderId); // No error, but wrong!

// Solution: branded types
type UserId = string & { __brand: 'UserId' };
type OrderId = string & { __brand: 'OrderId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(userId: UserId) { }
function getOrder(orderId: OrderId) { }

const userId = createUserId('123');
const orderId = createOrderId('456');

getUser(userId);  // OK
getUser(orderId); // Error!
```

## Refactoring Checklist

- [ ] Remove all `any` types
- [ ] Add return types to functions
- [ ] Use union types instead of enums
- [ ] Use discriminated unions for variants
- [ ] Apply utility types where appropriate
- [ ] Add type guards for runtime checks
- [ ] Use `satisfies` for config objects
- [ ] Enable strict mode in tsconfig

## tsconfig.json Recommended Settings

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Success Criteria

- [ ] No `any` types
- [ ] All functions have explicit return types
- [ ] Discriminated unions for state variants
- [ ] Type guards for runtime safety
- [ ] Strict mode enabled

## Common Pitfalls

- Don't use `any` as an escape hatch
- Don't over-type (let TypeScript infer when obvious)
- Don't use `as` for type assertions (use type guards)
- Don't ignore TypeScript errors with `@ts-ignore`
- Don't forget to enable strict mode
