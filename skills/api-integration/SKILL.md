# API Integration

> Integrate external APIs with proper error handling, retries, and TypeScript types.

## Trigger

- User needs to fetch data from an external API
- User wants to integrate a third-party service
- User needs to make HTTP requests

## Instructions

### Step 1: Create API Client

```typescript
// lib/api-client.ts

const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'https://api.example.com';

interface FetchOptions extends RequestInit {
  timeout?: number;
}

class ApiError extends Error {
  constructor(
    public status: number,
    public statusText: string,
    public data?: unknown
  ) {
    super(`API Error: ${status} ${statusText}`);
    this.name = 'ApiError';
  }
}

async function fetchWithTimeout(
  url: string,
  options: FetchOptions = {}
): Promise<Response> {
  const { timeout = 10000, ...fetchOptions } = options;

  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      ...fetchOptions,
      signal: controller.signal,
    });
    return response;
  } finally {
    clearTimeout(timeoutId);
  }
}

export async function apiRequest<T>(
  endpoint: string,
  options: FetchOptions = {}
): Promise<T> {
  const url = `${API_BASE}${endpoint}`;

  const response = await fetchWithTimeout(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (!response.ok) {
    const data = await response.json().catch(() => null);
    throw new ApiError(response.status, response.statusText, data);
  }

  return response.json();
}
```

### Step 2: Add Retry Logic

```typescript
// lib/api-client.ts (continued)

interface RetryOptions {
  retries?: number;
  retryDelay?: number;
  retryOn?: number[];
}

export async function fetchWithRetry<T>(
  endpoint: string,
  options: FetchOptions & RetryOptions = {}
): Promise<T> {
  const {
    retries = 3,
    retryDelay = 1000,
    retryOn = [408, 429, 500, 502, 503, 504],
    ...fetchOptions
  } = options;

  let lastError: Error | null = null;

  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await apiRequest<T>(endpoint, fetchOptions);
    } catch (error) {
      lastError = error as Error;

      const shouldRetry =
        error instanceof ApiError &&
        retryOn.includes(error.status) &&
        attempt < retries;

      if (!shouldRetry) {
        throw error;
      }

      // Exponential backoff
      const delay = retryDelay * Math.pow(2, attempt);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}
```

### Step 3: Define TypeScript Types

```typescript
// types/api.ts

// Response types
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  createdAt: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    perPage: number;
    totalPages: number;
  };
}

export interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}
```

### Step 4: Create API Functions

```typescript
// lib/api/users.ts
import { fetchWithRetry } from '../api-client';
import { User, PaginatedResponse } from '@/types/api';

export async function getUsers(page = 1, limit = 10) {
  return fetchWithRetry<PaginatedResponse<User>>(
    `/users?page=${page}&limit=${limit}`
  );
}

export async function getUser(id: string) {
  return fetchWithRetry<User>(`/users/${id}`);
}

export async function createUser(data: Omit<User, 'id' | 'createdAt'>) {
  return fetchWithRetry<User>('/users', {
    method: 'POST',
    body: JSON.stringify(data),
  });
}

export async function updateUser(id: string, data: Partial<User>) {
  return fetchWithRetry<User>(`/users/${id}`, {
    method: 'PATCH',
    body: JSON.stringify(data),
  });
}

export async function deleteUser(id: string) {
  return fetchWithRetry<void>(`/users/${id}`, {
    method: 'DELETE',
  });
}
```

### Step 5: Handle Authentication

```typescript
// lib/api-client.ts (auth extension)

let authToken: string | null = null;

export function setAuthToken(token: string | null) {
  authToken = token;
}

export async function authenticatedRequest<T>(
  endpoint: string,
  options: FetchOptions = {}
): Promise<T> {
  if (!authToken) {
    throw new Error('No auth token set');
  }

  return apiRequest<T>(endpoint, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${authToken}`,
    },
  });
}
```

### Step 6: React Hook Integration

```typescript
// hooks/useApi.ts
import { useState, useEffect, useCallback } from 'react';

interface UseApiState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

export function useApi<T>(
  fetcher: () => Promise<T>,
  deps: unknown[] = []
): UseApiState<T> & { refetch: () => void } {
  const [state, setState] = useState<UseApiState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  const fetchData = useCallback(async () => {
    setState((prev) => ({ ...prev, loading: true, error: null }));
    try {
      const data = await fetcher();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, deps);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { ...state, refetch: fetchData };
}

// Usage:
// const { data, loading, error } = useApi(() => getUsers(1, 10), []);
```

### Step 7: Error Handling in UI

```tsx
// components/UserList.tsx
import { useApi } from '@/hooks/useApi';
import { getUsers } from '@/lib/api/users';

export function UserList() {
  const { data, loading, error, refetch } = useApi(
    () => getUsers(1, 10),
    []
  );

  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return (
      <div>
        <p>Error: {error.message}</p>
        <button onClick={refetch}>Retry</button>
      </div>
    );
  }

  return (
    <ul>
      {data?.data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Environment Setup

```env
# .env.local
NEXT_PUBLIC_API_URL=https://api.example.com
API_KEY=your-secret-key
```

## Success Criteria

- [ ] API client with base URL configuration
- [ ] Timeout handling
- [ ] Retry logic with exponential backoff
- [ ] TypeScript types for all responses
- [ ] Error handling with user-friendly messages
- [ ] Loading states in UI

## Common Pitfalls

- Don't forget timeout handling
- Don't retry on client errors (4xx except 429)
- Don't expose API keys in client-side code
- Don't forget to handle network errors
- Don't skip TypeScript types for API responses
