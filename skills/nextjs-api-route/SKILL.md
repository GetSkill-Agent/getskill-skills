# Next.js API Route

> Build type-safe API routes with Next.js App Router.

## Trigger

- User needs to create an API endpoint
- User needs CRUD operations for a resource
- User wants to handle form submissions server-side
- User needs webhook handlers

## Prerequisites

- Next.js 14+ (App Router)
- TypeScript

## Instructions

### Step 1: Basic Route Handler Structure

```
app/
├── api/
│   ├── users/
│   │   ├── route.ts          # GET /api/users, POST /api/users
│   │   └── [id]/
│   │       └── route.ts      # GET/PUT/DELETE /api/users/:id
│   └── health/
│       └── route.ts          # GET /api/health
```

### Step 2: Create Route Handler

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/users
export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');

    const users = await db.user.findMany({
      skip: (page - 1) * limit,
      take: limit,
    });

    return NextResponse.json({
      data: users,
      pagination: { page, limit },
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch users' },
      { status: 500 }
    );
  }
}

// POST /api/users
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    // Validate input (see zod-validation skill)
    const validated = createUserSchema.parse(body);

    const user = await db.user.create({
      data: validated,
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Failed to create user' },
      { status: 500 }
    );
  }
}
```

### Step 3: Dynamic Route Parameters

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface RouteParams {
  params: Promise<{ id: string }>;
}

// GET /api/users/:id
export async function GET(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;

  const user = await db.user.findUnique({
    where: { id },
  });

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}

// PUT /api/users/:id
export async function PUT(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;
  const body = await request.json();

  try {
    const user = await db.user.update({
      where: { id },
      data: body,
    });
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
}

// DELETE /api/users/:id
export async function DELETE(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;

  try {
    await db.user.delete({ where: { id } });
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
}
```

### Step 4: Request Helpers

```typescript
// lib/api/request.ts
import { NextRequest } from 'next/server';

export async function parseBody<T>(request: NextRequest): Promise<T | null> {
  try {
    return await request.json();
  } catch {
    return null;
  }
}

export function getSearchParams(request: NextRequest) {
  const { searchParams } = request.nextUrl;

  return {
    get: (key: string) => searchParams.get(key),
    getNumber: (key: string, defaultValue: number) => {
      const value = searchParams.get(key);
      return value ? parseInt(value, 10) : defaultValue;
    },
    getBoolean: (key: string) => searchParams.get(key) === 'true',
    getArray: (key: string) => searchParams.getAll(key),
  };
}

export function getHeaders(request: NextRequest) {
  return {
    authorization: request.headers.get('authorization'),
    contentType: request.headers.get('content-type'),
    userAgent: request.headers.get('user-agent'),
  };
}
```

### Step 5: Response Helpers

```typescript
// lib/api/response.ts
import { NextResponse } from 'next/server';

interface ApiResponse<T> {
  data?: T;
  error?: string;
  details?: unknown;
}

export function success<T>(data: T, status = 200) {
  return NextResponse.json({ data }, { status });
}

export function created<T>(data: T) {
  return NextResponse.json({ data }, { status: 201 });
}

export function noContent() {
  return new NextResponse(null, { status: 204 });
}

export function badRequest(message: string, details?: unknown) {
  return NextResponse.json(
    { error: message, details },
    { status: 400 }
  );
}

export function unauthorized(message = 'Unauthorized') {
  return NextResponse.json({ error: message }, { status: 401 });
}

export function forbidden(message = 'Forbidden') {
  return NextResponse.json({ error: message }, { status: 403 });
}

export function notFound(resource = 'Resource') {
  return NextResponse.json(
    { error: `${resource} not found` },
    { status: 404 }
  );
}

export function serverError(message = 'Internal server error') {
  return NextResponse.json({ error: message }, { status: 500 });
}
```

### Step 6: Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from 'next/server';

type Handler = (
  request: NextRequest,
  context: { params: Promise<Record<string, string>> }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

// Compose multiple middleware
export function withMiddleware(...middlewares: Middleware[]) {
  return (handler: Handler): Handler => {
    return middlewares.reduceRight(
      (next, middleware) => middleware(next),
      handler
    );
  };
}

// Error handling middleware
export const withErrorHandling: Middleware = (handler) => async (req, ctx) => {
  try {
    return await handler(req, ctx);
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
};

// Auth middleware
export const withAuth: Middleware = (handler) => async (req, ctx) => {
  const token = req.headers.get('authorization')?.replace('Bearer ', '');

  if (!token) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  // Verify token and attach user to request
  try {
    const user = await verifyToken(token);
    // Attach user to headers for downstream use
    req.headers.set('x-user-id', user.id);
    return handler(req, ctx);
  } catch {
    return NextResponse.json(
      { error: 'Invalid token' },
      { status: 401 }
    );
  }
};

// Rate limiting middleware
export const withRateLimit = (limit: number, window: number): Middleware => {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, ctx) => {
    const ip = req.headers.get('x-forwarded-for') || 'unknown';
    const now = Date.now();

    const record = requests.get(ip);
    if (record && record.resetAt > now) {
      if (record.count >= limit) {
        return NextResponse.json(
          { error: 'Too many requests' },
          { status: 429 }
        );
      }
      record.count++;
    } else {
      requests.set(ip, { count: 1, resetAt: now + window });
    }

    return handler(req, ctx);
  };
};

// Usage
export const GET = withMiddleware(
  withErrorHandling,
  withAuth,
  withRateLimit(100, 60000)
)(async (request, { params }) => {
  // Handler logic
  return NextResponse.json({ message: 'Success' });
});
```

### Step 7: Validation Integration

```typescript
// app/api/users/route.ts
import { z } from 'zod';
import { NextRequest } from 'next/server';
import { success, badRequest, serverError } from '@/lib/api/response';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  role: z.enum(['user', 'admin']).default('user'),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const result = createUserSchema.safeParse(body);

    if (!result.success) {
      return badRequest('Validation failed', result.error.flatten());
    }

    const user = await db.user.create({
      data: result.data,
    });

    return success(user, 201);
  } catch (error) {
    console.error('Create user error:', error);
    return serverError();
  }
}
```

### Step 8: File Upload Handling

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { writeFile } from 'fs/promises';
import { join } from 'path';

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const file = formData.get('file') as File | null;

  if (!file) {
    return NextResponse.json(
      { error: 'No file provided' },
      { status: 400 }
    );
  }

  // Validate file type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
  if (!allowedTypes.includes(file.type)) {
    return NextResponse.json(
      { error: 'Invalid file type' },
      { status: 400 }
    );
  }

  // Validate file size (5MB)
  if (file.size > 5 * 1024 * 1024) {
    return NextResponse.json(
      { error: 'File too large' },
      { status: 400 }
    );
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  const filename = `${Date.now()}-${file.name}`;
  const path = join(process.cwd(), 'public/uploads', filename);

  await writeFile(path, buffer);

  return NextResponse.json({
    url: `/uploads/${filename}`,
  });
}
```

## Route Handler Summary

| Method | Purpose | Status Codes |
|--------|---------|--------------|
| GET | Retrieve resource(s) | 200, 404 |
| POST | Create resource | 201, 400 |
| PUT | Update resource | 200, 404 |
| PATCH | Partial update | 200, 404 |
| DELETE | Remove resource | 204, 404 |

## Success Criteria

- [ ] Routes follow RESTful conventions
- [ ] Input is validated before processing
- [ ] Errors return appropriate status codes
- [ ] Responses have consistent structure
- [ ] Authentication/authorization is implemented
- [ ] Rate limiting is considered

## Common Pitfalls

- Don't forget to await params in dynamic routes (Next.js 15+)
- Don't expose internal error details in production
- Don't skip input validation
- Don't forget CORS headers for external access
- Don't return 200 for errors
- Don't forget to handle empty body cases
