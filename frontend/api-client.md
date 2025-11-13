# API Client Configuration

## Overview

The API client is a singleton HTTP client configured with axios that handles all backend communication. It follows a clean separation of concerns where authentication logic is handled by `auth.service.ts`, while the client only manages HTTP requests and responses.

## Configuration

The API client is located at `src/client/api-client.ts` and provides a configured axios instance:

```typescript
import authService from "@/services/auth.service";
import axios from "axios";

const BASE_URL = import.meta.env.VITE_API_URL || "http://localhost:3000";

export const client = axios.create({
  baseURL: BASE_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

client.interceptors.request.use(config => {
  const token = authService.getToken(); // Or something similar
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

## Key Principles

### 1. Singleton Pattern

The client is exported as a singleton instance that can be imported and used throughout the application:

```typescript
import { client } from "@/client/api-client";
```

### 2. Environment-Based Configuration

The base URL is configured via environment variable `VITE_API_URL`, with a fallback to `http://localhost:3000` for local development.

### 3. Automatic Authentication

The request interceptor automatically adds the authentication token to every request by fetching it from `authService` on each call. This ensures:

- Token is always fresh (read from storage on each request)
- No stale tokens in headers
- Clean separation: client doesn't manage token lifecycle

### 4. Separation of Concerns

**Authentication management** (`auth.service.ts`):

- Stores and retrieves tokens from localStorage
- Provides `getToken()`, `setToken()`, `clearToken()`, `isAuthenticated()` methods
- Manages token lifecycle

**HTTP client** (`client/api-client.ts`):

- Only handles HTTP configuration and requests
- Reads token from authService when needed
- Does not store or manage tokens

## Request Interceptor

The request interceptor runs before every HTTP request and automatically adds the Authorization header if a token exists:

```typescript
client.interceptors.request.use(config => {
  const token = authService.getToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

**Benefits:**

- Token is fetched fresh on each request
- No need to manually add headers in repositories
- Consistent authentication across all API calls

## Response Interceptor

You can add a response interceptor to handle common error cases:

```typescript
client.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      authService.clearToken();
      // Optionally redirect to login
    }
    return Promise.reject(error);
  },
);
```

## Usage in Repositories

Repositories use the client for all API interactions:

```typescript
// src/repositories/users.repository.ts
import { client } from "@/client/api-client";

async function list(): Promise<User[]> {
  const response = await client.get("/users");
  return response.data;
}

async function getById(id: string): Promise<User> {
  const response = await client.get(`/users/${id}`);
  return response.data;
}

async function create(user: CreateUserRequest): Promise<User> {
  const response = await client.post("/users", user);
  return response.data;
}

export default { list, getById, create };
```

## Import Path Aliases

Always use the `@/` alias for imports to maintain consistency with the recursive architecture:

```typescript
// ✅ Correct
import { client } from "@/client/api-client";
import authService from "@/services/auth.service";

// ❌ Wrong - relative imports going up are prohibited
import { client } from "../client/api-client";
```

For more information on import rules, see the [Architecture Guide](./architecture.md) and [Tooling Guide](./tooling.md).

## Related Documentation

- [Architecture Guide](./architecture.md) - Learn about the separation between services and client
- [Tooling Guide](./tooling.md) - ESLint rules for import paths
- [Testing Guide](./frontend-testing.md) - How to mock the API client in tests
