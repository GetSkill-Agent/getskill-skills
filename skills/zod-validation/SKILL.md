# Zod Validation

> Type-safe schema validation with automatic TypeScript inference.

## Trigger

- User needs to validate form input
- User needs API request/response validation
- User needs runtime type checking
- User needs to parse environment variables

## Prerequisites

```bash
npm install zod
```

## Instructions

### Step 1: Basic Schema Definition

```typescript
import { z } from 'zod';

// Primitives
const stringSchema = z.string();
const numberSchema = z.number();
const booleanSchema = z.boolean();
const dateSchema = z.date();

// Object schema
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().optional(),
  role: z.enum(['user', 'admin', 'moderator']),
  createdAt: z.date(),
});

// Infer TypeScript type from schema
type User = z.infer<typeof userSchema>;
// Result: { id: string; email: string; name: string; age?: number; role: 'user' | 'admin' | 'moderator'; createdAt: Date }
```

### Step 2: String Validations

```typescript
const stringSchemas = {
  // Length constraints
  username: z.string().min(3).max(20),

  // Format validations
  email: z.string().email('Invalid email format'),
  url: z.string().url('Must be a valid URL'),
  uuid: z.string().uuid(),

  // Pattern matching
  slug: z.string().regex(/^[a-z0-9-]+$/, 'Invalid slug format'),
  phone: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),

  // Transformations
  trimmed: z.string().trim(),
  lowercase: z.string().toLowerCase(),
  uppercase: z.string().toUpperCase(),

  // Custom refinement
  noSpaces: z.string().refine(
    (val) => !val.includes(' '),
    { message: 'Cannot contain spaces' }
  ),
};
```

### Step 3: Number Validations

```typescript
const numberSchemas = {
  // Constraints
  positive: z.number().positive(),
  negative: z.number().negative(),
  nonNegative: z.number().nonnegative(),
  integer: z.number().int(),

  // Range
  percentage: z.number().min(0).max(100),
  age: z.number().int().min(0).max(150),

  // From string (form inputs)
  fromString: z.coerce.number(),

  // Safe integer
  safeInt: z.number().int().safe(),

  // Multiple of
  evenNumber: z.number().multipleOf(2),
};
```

### Step 4: Object Patterns

```typescript
// Basic object
const personSchema = z.object({
  name: z.string(),
  age: z.number(),
});

// Nested objects
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  country: z.string(),
  zipCode: z.string(),
});

const userWithAddressSchema = z.object({
  name: z.string(),
  address: addressSchema,
});

// Optional and nullable
const profileSchema = z.object({
  bio: z.string().optional(),        // string | undefined
  avatar: z.string().nullable(),     // string | null
  website: z.string().nullish(),     // string | null | undefined
});

// Pick and omit
const publicUser = userSchema.pick({ name: true, email: true });
const userWithoutId = userSchema.omit({ id: true });

// Extend
const adminSchema = userSchema.extend({
  permissions: z.array(z.string()),
});

// Merge
const fullProfileSchema = userSchema.merge(profileSchema);

// Partial (all fields optional)
const updateUserSchema = userSchema.partial();

// Deep partial
const deepPartialSchema = userSchema.deepPartial();

// Required (make optional fields required)
const requiredSchema = profileSchema.required();

// Strict (reject unknown keys)
const strictSchema = z.object({
  name: z.string(),
}).strict();

// Passthrough (allow unknown keys)
const passthroughSchema = z.object({
  name: z.string(),
}).passthrough();
```

### Step 5: Array and Tuple

```typescript
// Array
const tagsSchema = z.array(z.string());
const numbersSchema = z.array(z.number()).min(1).max(10);
const uniqueTagsSchema = z.array(z.string()).refine(
  (items) => new Set(items).size === items.length,
  { message: 'Tags must be unique' }
);

// Non-empty array
const nonEmptyArray = z.array(z.string()).nonempty();

// Tuple (fixed length, mixed types)
const coordinatesSchema = z.tuple([z.number(), z.number()]);
const recordSchema = z.tuple([z.string(), z.number(), z.boolean()]);

// Tuple with rest
const argsSchema = z.tuple([z.string()]).rest(z.number());
// ['hello', 1, 2, 3] is valid
```

### Step 6: Union and Discriminated Union

```typescript
// Simple union
const stringOrNumber = z.union([z.string(), z.number()]);
// Shorthand
const stringOrNumberShort = z.string().or(z.number());

// Discriminated union (better error messages)
const eventSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('click'),
    x: z.number(),
    y: z.number(),
  }),
  z.object({
    type: z.literal('scroll'),
    direction: z.enum(['up', 'down']),
    amount: z.number(),
  }),
  z.object({
    type: z.literal('keypress'),
    key: z.string(),
  }),
]);

// Type is automatically narrowed
type Event = z.infer<typeof eventSchema>;
```

### Step 7: Transforms and Preprocessing

```typescript
// Transform output
const dateStringSchema = z.string().transform((str) => new Date(str));

// Coerce (auto-convert input)
const coercedNumber = z.coerce.number();  // "123" -> 123
const coercedDate = z.coerce.date();      // "2024-01-01" -> Date
const coercedBoolean = z.coerce.boolean(); // "true" -> true

// Preprocess (transform before validation)
const trimmedEmail = z.preprocess(
  (val) => typeof val === 'string' ? val.trim().toLowerCase() : val,
  z.string().email()
);

// Chain transforms
const processedSchema = z.string()
  .trim()
  .toLowerCase()
  .transform((val) => val.replace(/\s+/g, '-'));
```

### Step 8: Custom Error Messages

```typescript
const userSchema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string',
  }).email({ message: 'Please enter a valid email' }),

  password: z.string()
    .min(8, { message: 'Password must be at least 8 characters' })
    .regex(/[A-Z]/, { message: 'Password must contain uppercase letter' })
    .regex(/[0-9]/, { message: 'Password must contain a number' }),

  age: z.number()
    .int({ message: 'Age must be a whole number' })
    .min(18, { message: 'Must be at least 18 years old' }),
});

// Error formatting
const result = userSchema.safeParse(data);
if (!result.success) {
  // Flat errors
  const flat = result.error.flatten();
  // { formErrors: [], fieldErrors: { email: ['...'], password: ['...'] } }

  // Formatted errors
  const formatted = result.error.format();
  // { email: { _errors: ['...'] }, password: { _errors: ['...'] } }
}
```

### Step 9: Refinements and Super Refine

```typescript
// Simple refinement
const passwordSchema = z.string().refine(
  (val) => /[A-Z]/.test(val) && /[0-9]/.test(val),
  { message: 'Password must contain uppercase and number' }
);

// Super refine for complex validations
const signupSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).superRefine((data, ctx) => {
  if (data.password !== data.confirmPassword) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: 'Passwords do not match',
      path: ['confirmPassword'],
    });
  }
});

// Async refinement
const uniqueEmailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: 'Email already exists' }
);
```

### Step 10: Environment Variables

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  PORT: z.coerce.number().default(3000),
  DEBUG: z.coerce.boolean().default(false),
  ALLOWED_ORIGINS: z.string().transform((val) => val.split(',')),
});

// Parse and validate
export const env = envSchema.parse(process.env);

// Type-safe access
console.log(env.DATABASE_URL); // string
console.log(env.PORT);         // number
console.log(env.DEBUG);        // boolean
```

### Step 11: API Request/Response Validation

```typescript
// schemas/api.ts
import { z } from 'zod';

// Request schemas
export const createPostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1),
  published: z.boolean().default(false),
  tags: z.array(z.string()).default([]),
});

export const updatePostSchema = createPostSchema.partial();

export const querySchema = z.object({
  page: z.coerce.number().positive().default(1),
  limit: z.coerce.number().positive().max(100).default(10),
  sort: z.enum(['newest', 'oldest', 'popular']).default('newest'),
  search: z.string().optional(),
});

// Response schema (for type inference)
export const postResponseSchema = z.object({
  id: z.string(),
  title: z.string(),
  content: z.string(),
  published: z.boolean(),
  createdAt: z.string().datetime(),
});

export type CreatePostInput = z.infer<typeof createPostSchema>;
export type PostResponse = z.infer<typeof postResponseSchema>;
```

## Common Patterns

| Pattern | Schema | Example |
|---------|--------|---------|
| Email | `z.string().email()` | `"user@example.com"` |
| UUID | `z.string().uuid()` | `"550e8400-e29b-..."` |
| URL | `z.string().url()` | `"https://example.com"` |
| Date string | `z.coerce.date()` | `"2024-01-01"` -> Date |
| Enum | `z.enum(['a', 'b'])` | `"a"` or `"b"` |
| Optional | `z.string().optional()` | `string \| undefined` |
| Nullable | `z.string().nullable()` | `string \| null` |
| Default | `z.string().default('x')` | Falls back to `"x"` |

## Success Criteria

- [ ] Schema matches expected data structure
- [ ] Error messages are user-friendly
- [ ] TypeScript types are inferred correctly
- [ ] Edge cases are handled (empty, null, undefined)
- [ ] Transforms produce correct output types

## Common Pitfalls

- Don't forget `.safeParse()` vs `.parse()` - latter throws
- Don't mix up `optional()`, `nullable()`, and `nullish()`
- Don't forget `z.coerce` for form inputs (strings to numbers)
- Don't skip error message customization for user-facing forms
- Don't use `z.any()` - defeats the purpose of validation
- Don't forget async validation needs `parseAsync()`
