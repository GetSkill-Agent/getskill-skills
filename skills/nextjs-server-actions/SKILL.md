# Next.js Server Actions

> Handle form submissions and mutations with Server Actions.

## Trigger

- User needs to handle form submission
- User needs to perform database mutations
- User needs progressive enhancement (works without JS)
- User wants to revalidate cached data

## Prerequisites

- Next.js 14+ (App Router)
- TypeScript

## Instructions

### Step 1: Basic Server Action

```typescript
// app/actions.ts
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  // Database operation
  const user = await db.user.create({
    data: { name, email },
  });

  return user;
}
```

**Usage in form:**
```tsx
// app/page.tsx
import { createUser } from './actions';

export default function Page() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Create User</button>
    </form>
  );
}
```

### Step 2: With Validation and Error Handling

```typescript
// app/actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
});

export type ActionState = {
  success: boolean;
  message: string;
  errors?: Record<string, string[]>;
};

export async function createUser(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  // Validate input
  const result = createUserSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  });

  if (!result.success) {
    return {
      success: false,
      message: 'Validation failed',
      errors: result.error.flatten().fieldErrors,
    };
  }

  try {
    await db.user.create({
      data: result.data,
    });

    // Revalidate the users list
    revalidatePath('/users');

    return {
      success: true,
      message: 'User created successfully',
    };
  } catch (error) {
    return {
      success: false,
      message: 'Failed to create user',
    };
  }
}
```

### Step 3: Form Component with useActionState

```tsx
// components/CreateUserForm.tsx
'use client';

import { useActionState } from 'react';
import { createUser, ActionState } from '@/app/actions';

const initialState: ActionState = {
  success: false,
  message: '',
};

export function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(createUser, initialState);

  return (
    <form action={formAction} className="space-y-4">
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          name="name"
          required
          disabled={isPending}
          className={state.errors?.name ? 'border-red-500' : ''}
        />
        {state.errors?.name && (
          <p className="text-red-500 text-sm">{state.errors.name[0]}</p>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          required
          disabled={isPending}
          className={state.errors?.email ? 'border-red-500' : ''}
        />
        {state.errors?.email && (
          <p className="text-red-500 text-sm">{state.errors.email[0]}</p>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>

      {state.message && (
        <p className={state.success ? 'text-green-500' : 'text-red-500'}>
          {state.message}
        </p>
      )}
    </form>
  );
}
```

### Step 4: Optimistic Updates

```tsx
// components/LikeButton.tsx
'use client';

import { useOptimistic, useTransition } from 'react';
import { toggleLike } from '@/app/actions';

interface LikeButtonProps {
  postId: string;
  initialLikes: number;
  isLiked: boolean;
}

export function LikeButton({ postId, initialLikes, isLiked }: LikeButtonProps) {
  const [isPending, startTransition] = useTransition();

  const [optimisticState, addOptimistic] = useOptimistic(
    { likes: initialLikes, isLiked },
    (state, newIsLiked: boolean) => ({
      likes: state.likes + (newIsLiked ? 1 : -1),
      isLiked: newIsLiked,
    })
  );

  const handleClick = () => {
    startTransition(async () => {
      addOptimistic(!optimisticState.isLiked);
      await toggleLike(postId);
    });
  };

  return (
    <button
      onClick={handleClick}
      disabled={isPending}
      className={optimisticState.isLiked ? 'text-red-500' : 'text-gray-500'}
    >
      {optimisticState.isLiked ? '❤️' : '🤍'} {optimisticState.likes}
    </button>
  );
}
```

### Step 5: Server Action with Redirect

```typescript
// app/actions.ts
'use server';

import { redirect } from 'next/navigation';
import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  const post = await db.post.create({
    data: { title, content },
  });

  // Revalidate and redirect
  revalidatePath('/posts');
  redirect(`/posts/${post.id}`);
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });

  revalidatePath('/posts');
  redirect('/posts');
}
```

### Step 6: Revalidation Strategies

```typescript
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function updateUser(id: string, data: FormData) {
  await db.user.update({
    where: { id },
    data: {
      name: data.get('name') as string,
    },
  });

  // Option 1: Revalidate specific path
  revalidatePath(`/users/${id}`);

  // Option 2: Revalidate path and all sub-paths
  revalidatePath('/users', 'layout');

  // Option 3: Revalidate by cache tag
  revalidateTag('users');
  revalidateTag(`user-${id}`);
}

// In your data fetching:
async function getUser(id: string) {
  const user = await fetch(`/api/users/${id}`, {
    next: { tags: ['users', `user-${id}`] },
  });
  return user.json();
}
```

### Step 7: File Upload with Server Action

```typescript
// app/actions.ts
'use server';

import { writeFile } from 'fs/promises';
import { join } from 'path';

export async function uploadFile(formData: FormData) {
  const file = formData.get('file') as File;

  if (!file || file.size === 0) {
    return { error: 'No file provided' };
  }

  // Validate file type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
  if (!allowedTypes.includes(file.type)) {
    return { error: 'Invalid file type' };
  }

  // Validate size (5MB max)
  if (file.size > 5 * 1024 * 1024) {
    return { error: 'File too large (max 5MB)' };
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  const filename = `${Date.now()}-${file.name}`;
  const path = join(process.cwd(), 'public/uploads', filename);

  await writeFile(path, buffer);

  return { url: `/uploads/${filename}` };
}
```

```tsx
// components/FileUpload.tsx
'use client';

import { useRef, useState } from 'react';
import { uploadFile } from '@/app/actions';

export function FileUpload() {
  const [preview, setPreview] = useState<string | null>(null);
  const [uploading, setUploading] = useState(false);
  const formRef = useRef<HTMLFormElement>(null);

  const handleSubmit = async (formData: FormData) => {
    setUploading(true);
    const result = await uploadFile(formData);
    setUploading(false);

    if (result.url) {
      setPreview(result.url);
      formRef.current?.reset();
    }
  };

  return (
    <form ref={formRef} action={handleSubmit}>
      <input
        type="file"
        name="file"
        accept="image/*"
        disabled={uploading}
      />
      <button type="submit" disabled={uploading}>
        {uploading ? 'Uploading...' : 'Upload'}
      </button>
      {preview && <img src={preview} alt="Preview" className="mt-4 max-w-xs" />}
    </form>
  );
}
```

### Step 8: Binding Arguments

```typescript
// app/actions.ts
'use server';

export async function updateItem(id: string, formData: FormData) {
  const name = formData.get('name') as string;

  await db.item.update({
    where: { id },
    data: { name },
  });

  revalidatePath('/items');
}
```

```tsx
// components/ItemForm.tsx
import { updateItem } from '@/app/actions';

export function ItemForm({ item }: { item: { id: string; name: string } }) {
  // Bind the id to the action
  const updateItemWithId = updateItem.bind(null, item.id);

  return (
    <form action={updateItemWithId}>
      <input name="name" defaultValue={item.name} />
      <button type="submit">Update</button>
    </form>
  );
}
```

## Server Action Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Form Action** | Simple form submission | `<form action={createUser}>` |
| **useActionState** | Form with validation feedback | Form with field errors |
| **useOptimistic** | Instant UI feedback | Like button, todo toggle |
| **bind** | Pass extra arguments | `action.bind(null, id)` |
| **redirect** | Navigate after action | Create then redirect to detail |
| **revalidatePath** | Refresh specific route data | After CRUD operations |
| **revalidateTag** | Refresh tagged cache | Related data updates |

## Success Criteria

- [ ] Action has 'use server' directive
- [ ] Input is validated before database operations
- [ ] Errors are returned, not thrown (for form state)
- [ ] Loading state is shown during submission
- [ ] Data is revalidated after mutations
- [ ] Form works without JavaScript (progressive enhancement)

## Common Pitfalls

- Don't forget 'use server' at the top of action files
- Don't throw errors in form actions (return error state instead)
- Don't forget to revalidate after mutations
- Don't use client-side state in server actions
- Don't skip validation (user can bypass client validation)
- Don't redirect inside try/catch (redirect throws internally)
