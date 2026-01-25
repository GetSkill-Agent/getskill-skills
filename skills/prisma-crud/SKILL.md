# Prisma CRUD

> Database operations with Prisma ORM.

## Trigger

- User needs to create database queries
- User needs CRUD operations
- User needs to query related data
- User needs database transactions

## Prerequisites

```bash
npm install prisma @prisma/client
npx prisma init
```

## Instructions

### Step 1: Schema Definition

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  posts     Post[]
  profile   Profile?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

model Profile {
  id     String  @id @default(cuid())
  bio    String?
  avatar String?
  userId String  @unique
  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  tags      Tag[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([published])
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
  MODERATOR
}
```

### Step 2: Prisma Client Setup

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### Step 3: Create Operations

```typescript
// Single create
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
    role: 'USER',
  },
});

// Create with relations
const userWithProfile = await prisma.user.create({
  data: {
    email: 'bob@example.com',
    name: 'Bob',
    profile: {
      create: {
        bio: 'Hello world',
      },
    },
    posts: {
      create: [
        { title: 'First post' },
        { title: 'Second post' },
      ],
    },
  },
  include: {
    profile: true,
    posts: true,
  },
});

// Create many
const users = await prisma.user.createMany({
  data: [
    { email: 'user1@example.com', name: 'User 1' },
    { email: 'user2@example.com', name: 'User 2' },
  ],
  skipDuplicates: true, // Ignore duplicates
});

// Create or connect existing
const post = await prisma.post.create({
  data: {
    title: 'New post',
    author: {
      connect: { email: 'alice@example.com' },
    },
    tags: {
      connectOrCreate: [
        {
          where: { name: 'typescript' },
          create: { name: 'typescript' },
        },
      ],
    },
  },
});
```

### Step 4: Read Operations

```typescript
// Find unique (by unique field)
const user = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
});

// Find unique or throw
const userOrThrow = await prisma.user.findUniqueOrThrow({
  where: { id: 'abc123' },
});

// Find first matching
const firstAdmin = await prisma.user.findFirst({
  where: { role: 'ADMIN' },
});

// Find many with filters
const users = await prisma.user.findMany({
  where: {
    role: 'USER',
    email: { contains: '@example.com' },
    createdAt: { gte: new Date('2024-01-01') },
  },
  orderBy: { createdAt: 'desc' },
  take: 10,
  skip: 0,
});

// Select specific fields
const userEmails = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
  },
});

// Include relations
const userWithPosts = await prisma.user.findUnique({
  where: { id: 'abc123' },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
    },
    profile: true,
  },
});

// Nested select
const postsWithAuthorName = await prisma.post.findMany({
  select: {
    id: true,
    title: true,
    author: {
      select: {
        name: true,
        email: true,
      },
    },
  },
});
```

### Step 5: Filter Operators

```typescript
const filtered = await prisma.user.findMany({
  where: {
    // Equals
    email: 'alice@example.com',
    // Or explicit
    role: { equals: 'USER' },

    // Not equals
    name: { not: null },

    // Contains (case-sensitive)
    email: { contains: '@example' },

    // Case-insensitive
    name: { contains: 'alice', mode: 'insensitive' },

    // Starts/ends with
    email: { startsWith: 'alice' },
    email: { endsWith: '.com' },

    // In list
    role: { in: ['USER', 'ADMIN'] },
    role: { notIn: ['MODERATOR'] },

    // Comparison
    createdAt: { gt: new Date('2024-01-01') },
    createdAt: { gte: new Date('2024-01-01') },
    createdAt: { lt: new Date('2024-12-31') },
    createdAt: { lte: new Date('2024-12-31') },

    // AND (implicit)
    AND: [
      { role: 'USER' },
      { email: { contains: '@example' } },
    ],

    // OR
    OR: [
      { role: 'ADMIN' },
      { email: { endsWith: '@company.com' } },
    ],

    // NOT
    NOT: {
      role: 'MODERATOR',
    },

    // Relation filters
    posts: {
      some: { published: true },
    },
    posts: {
      every: { published: true },
    },
    posts: {
      none: { published: false },
    },
  },
});
```

### Step 6: Update Operations

```typescript
// Update single record
const user = await prisma.user.update({
  where: { email: 'alice@example.com' },
  data: {
    name: 'Alice Smith',
    role: 'ADMIN',
  },
});

// Update many
const result = await prisma.user.updateMany({
  where: { role: 'USER' },
  data: { role: 'MODERATOR' },
});
// result.count = number of updated records

// Upsert (update or create)
const user = await prisma.user.upsert({
  where: { email: 'alice@example.com' },
  update: { name: 'Alice Updated' },
  create: {
    email: 'alice@example.com',
    name: 'Alice New',
  },
});

// Increment/decrement
const post = await prisma.post.update({
  where: { id: 'abc123' },
  data: {
    viewCount: { increment: 1 },
    // decrement, multiply, divide also available
  },
});

// Update relations
const user = await prisma.user.update({
  where: { id: 'abc123' },
  data: {
    posts: {
      // Create new post
      create: { title: 'New post' },
      // Connect existing
      connect: { id: 'post123' },
      // Disconnect
      disconnect: { id: 'post456' },
      // Set (replace all)
      set: [{ id: 'post789' }],
      // Update nested
      update: {
        where: { id: 'post123' },
        data: { published: true },
      },
      // Delete nested
      delete: { id: 'post456' },
    },
  },
});
```

### Step 7: Delete Operations

```typescript
// Delete single
const user = await prisma.user.delete({
  where: { email: 'alice@example.com' },
});

// Delete many
const result = await prisma.user.deleteMany({
  where: {
    role: 'USER',
    createdAt: { lt: new Date('2023-01-01') },
  },
});
// result.count = number of deleted records

// Delete all (dangerous!)
await prisma.user.deleteMany({});
```

### Step 8: Transactions

```typescript
// Interactive transaction
const [user, post] = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'new@example.com', name: 'New User' },
  });

  const post = await tx.post.create({
    data: {
      title: 'First post',
      authorId: user.id,
    },
  });

  // If any operation fails, all are rolled back
  return [user, post];
});

// Sequential transaction (array)
const [user, posts] = await prisma.$transaction([
  prisma.user.findUnique({ where: { id: 'abc123' } }),
  prisma.post.findMany({ where: { authorId: 'abc123' } }),
]);

// Transaction with isolation level
await prisma.$transaction(
  async (tx) => {
    // operations
  },
  {
    isolationLevel: 'Serializable',
    maxWait: 5000, // max wait to acquire lock
    timeout: 10000, // max transaction time
  }
);
```

### Step 9: Aggregations

```typescript
// Count
const count = await prisma.user.count({
  where: { role: 'USER' },
});

// Aggregate
const stats = await prisma.post.aggregate({
  _count: true,
  _avg: { viewCount: true },
  _sum: { viewCount: true },
  _min: { createdAt: true },
  _max: { createdAt: true },
  where: { published: true },
});

// Group by
const usersByRole = await prisma.user.groupBy({
  by: ['role'],
  _count: true,
  _avg: { posts: { _count: true } },
  orderBy: { _count: { role: 'desc' } },
});
```

### Step 10: Pagination Patterns

```typescript
// Offset pagination
async function getUsers(page: number, pageSize: number) {
  const [users, total] = await prisma.$transaction([
    prisma.user.findMany({
      skip: (page - 1) * pageSize,
      take: pageSize,
      orderBy: { createdAt: 'desc' },
    }),
    prisma.user.count(),
  ]);

  return {
    data: users,
    pagination: {
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
    },
  };
}

// Cursor pagination (better for large datasets)
async function getUsersCursor(cursor?: string, take = 10) {
  const users = await prisma.user.findMany({
    take: take + 1, // Fetch one extra to check if more exist
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { id: 'asc' },
  });

  const hasMore = users.length > take;
  const data = hasMore ? users.slice(0, -1) : users;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  return { data, nextCursor, hasMore };
}
```

## Query Patterns Summary

| Operation | Method | Returns |
|-----------|--------|---------|
| Create one | `create` | Created record |
| Create many | `createMany` | `{ count: number }` |
| Find unique | `findUnique` | Record or null |
| Find first | `findFirst` | Record or null |
| Find many | `findMany` | Array of records |
| Update one | `update` | Updated record |
| Update many | `updateMany` | `{ count: number }` |
| Upsert | `upsert` | Created or updated record |
| Delete one | `delete` | Deleted record |
| Delete many | `deleteMany` | `{ count: number }` |

## Success Criteria

- [ ] Schema has proper relations and indexes
- [ ] Queries select only needed fields
- [ ] Pagination is implemented for lists
- [ ] Transactions used for multi-step operations
- [ ] Error handling for not found cases

## Common Pitfalls

- Don't use `findMany` without pagination for large tables
- Don't forget indexes on frequently queried fields
- Don't select all fields when you only need a few
- Don't skip error handling for `findUnique` (can return null)
- Don't forget to handle cascade deletes properly
- Don't use `$transaction` for read-only queries (unnecessary)
