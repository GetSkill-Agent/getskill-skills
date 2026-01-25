# TanStack Query

> Powerful data fetching and caching for React applications.

## Trigger

- User needs to fetch data from an API
- User needs client-side caching
- User needs optimistic updates
- User needs to manage server state

## Prerequisites

```bash
npm install @tanstack/react-query
```

## Instructions

### Step 1: Provider Setup

```tsx
// app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState, ReactNode } from 'react';

export function Providers({ children }: { children: ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            gcTime: 5 * 60 * 1000, // 5 minutes (formerly cacheTime)
            retry: 1,
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### Step 2: Basic Query

```tsx
// hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query';

interface User {
  id: string;
  name: string;
  email: string;
}

async function fetchUsers(): Promise<User[]> {
  const response = await fetch('/api/users');
  if (!response.ok) {
    throw new Error('Failed to fetch users');
  }
  return response.json();
}

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
}

// Usage
function UserList() {
  const { data: users, isLoading, isError, error } = useUsers();

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {users?.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Step 3: Query with Parameters

```tsx
// hooks/useUser.ts
import { useQuery } from '@tanstack/react-query';

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('User not found');
  }
  return response.json();
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => fetchUser(id),
    enabled: !!id, // Only run if id exists
  });
}

// With search params
export function useSearchUsers(query: string) {
  return useQuery({
    queryKey: ['users', 'search', query],
    queryFn: () => searchUsers(query),
    enabled: query.length >= 2, // Only search with 2+ chars
    placeholderData: [], // Show empty while loading
  });
}
```

### Step 4: Mutations

```tsx
// hooks/useCreateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

interface CreateUserInput {
  name: string;
  email: string;
}

async function createUser(input: CreateUserInput): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(input),
  });
  if (!response.ok) {
    throw new Error('Failed to create user');
  }
  return response.json();
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createUser,
    onSuccess: (newUser) => {
      // Option 1: Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });

      // Option 2: Update cache directly
      queryClient.setQueryData<User[]>(['users'], (old) => {
        return old ? [...old, newUser] : [newUser];
      });
    },
    onError: (error) => {
      console.error('Failed to create user:', error);
    },
  });
}

// Usage
function CreateUserForm() {
  const createUser = useCreateUser();

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    createUser.mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={createUser.isPending}>
        {createUser.isPending ? 'Creating...' : 'Create'}
      </button>
      {createUser.isError && <p>Error: {createUser.error.message}</p>}
    </form>
  );
}
```

### Step 5: Optimistic Updates

```tsx
// hooks/useUpdateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateUser,
    onMutate: async (updatedUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['users', updatedUser.id] });

      // Snapshot previous value
      const previousUser = queryClient.getQueryData<User>(['users', updatedUser.id]);

      // Optimistically update
      queryClient.setQueryData<User>(['users', updatedUser.id], (old) => ({
        ...old!,
        ...updatedUser,
      }));

      // Return context for rollback
      return { previousUser };
    },
    onError: (err, updatedUser, context) => {
      // Rollback on error
      if (context?.previousUser) {
        queryClient.setQueryData(['users', updatedUser.id], context.previousUser);
      }
    },
    onSettled: (data, error, variables) => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey: ['users', variables.id] });
    },
  });
}
```

### Step 6: Infinite Queries (Pagination)

```tsx
// hooks/usePosts.ts
import { useInfiniteQuery } from '@tanstack/react-query';

interface PostsResponse {
  data: Post[];
  nextCursor: string | null;
}

async function fetchPosts({ pageParam }: { pageParam: string | null }): Promise<PostsResponse> {
  const url = pageParam
    ? `/api/posts?cursor=${pageParam}`
    : '/api/posts';
  const response = await fetch(url);
  return response.json();
}

export function usePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    initialPageParam: null as string | null,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });
}

// Usage
function PostList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = usePosts();

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {data?.pages.map((page) =>
        page.data.map((post) => (
          <PostCard key={post.id} post={post} />
        ))
      )}

      {hasNextPage && (
        <button
          onClick={() => fetchNextPage()}
          disabled={isFetchingNextPage}
        >
          {isFetchingNextPage ? 'Loading more...' : 'Load more'}
        </button>
      )}
    </div>
  );
}
```

### Step 7: Prefetching

```tsx
// Prefetch on hover
import { useQueryClient } from '@tanstack/react-query';

function UserLink({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey: ['users', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 5 * 60 * 1000, // Don't refetch if less than 5 min old
    });
  };

  return (
    <Link href={`/users/${userId}`} onMouseEnter={handleMouseEnter}>
      View User
    </Link>
  );
}

// Prefetch in Server Component (Next.js)
// app/users/page.tsx
import { HydrationBoundary, QueryClient, dehydrate } from '@tanstack/react-query';

export default async function UsersPage() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <UserList />
    </HydrationBoundary>
  );
}
```

### Step 8: Query Keys Factory

```typescript
// lib/queryKeys.ts
export const queryKeys = {
  users: {
    all: ['users'] as const,
    lists: () => [...queryKeys.users.all, 'list'] as const,
    list: (filters: UserFilters) => [...queryKeys.users.lists(), filters] as const,
    details: () => [...queryKeys.users.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.users.details(), id] as const,
  },
  posts: {
    all: ['posts'] as const,
    lists: () => [...queryKeys.posts.all, 'list'] as const,
    list: (filters: PostFilters) => [...queryKeys.posts.lists(), filters] as const,
    details: () => [...queryKeys.posts.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.posts.details(), id] as const,
    byAuthor: (authorId: string) => [...queryKeys.posts.all, 'byAuthor', authorId] as const,
  },
};

// Usage
useQuery({
  queryKey: queryKeys.users.detail(userId),
  queryFn: () => fetchUser(userId),
});

// Invalidate all user queries
queryClient.invalidateQueries({ queryKey: queryKeys.users.all });

// Invalidate specific user
queryClient.invalidateQueries({ queryKey: queryKeys.users.detail(userId) });
```

### Step 9: Error Handling

```tsx
// Global error handler
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Don't retry on 404
        if (error instanceof Error && error.message === 'Not found') {
          return false;
        }
        return failureCount < 3;
      },
    },
    mutations: {
      onError: (error) => {
        // Global error toast
        toast.error(error.message);
      },
    },
  },
});

// Component-level error boundary
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div>
              <p>Error: {error.message}</p>
              <button onClick={resetErrorBoundary}>Retry</button>
            </div>
          )}
        >
          <UserList />
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}
```

### Step 10: Dependent Queries

```tsx
// Fetch user, then fetch their posts
function UserProfile({ userId }: { userId: string }) {
  const userQuery = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
  });

  const postsQuery = useQuery({
    queryKey: ['posts', { authorId: userId }],
    queryFn: () => fetchPostsByAuthor(userId),
    enabled: !!userQuery.data, // Only run when user is loaded
  });

  if (userQuery.isLoading) return <div>Loading user...</div>;

  return (
    <div>
      <h1>{userQuery.data?.name}</h1>
      {postsQuery.isLoading ? (
        <div>Loading posts...</div>
      ) : (
        <PostList posts={postsQuery.data ?? []} />
      )}
    </div>
  );
}
```

## Query States Summary

| State | Description |
|-------|-------------|
| `isLoading` | First fetch, no data yet |
| `isFetching` | Any fetch (including background) |
| `isError` | Query failed |
| `isSuccess` | Query succeeded |
| `isPending` | Mutation in progress |
| `isStale` | Data older than staleTime |

## Success Criteria

- [ ] Provider wraps the app
- [ ] Query keys follow consistent pattern
- [ ] Loading and error states handled
- [ ] Mutations invalidate relevant queries
- [ ] Optimistic updates for critical UX

## Common Pitfalls

- Don't create QueryClient inside component (causes re-renders)
- Don't forget `enabled` for conditional queries
- Don't use object references in queryKey (use primitives)
- Don't forget to invalidate after mutations
- Don't skip error handling
- Don't set staleTime: Infinity without good reason
