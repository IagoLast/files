# Browser Testing Setup for React SPAs

## What is this?

This guide explains how to set up **Vitest Browser Mode** for testing React SPAs. Tests run in a **real Chromium browser** (not jsdom), giving high confidence while maintaining fast execution.

The approach:

1. **Mock HTTP requests** with MSW (Mock Service Worker)
2. **Navigate to routes** using `@frontend-testing/vitest-browser-navigate`
3. **Render the full `<App />`** component
4. **Interact and assert** using Testing Library

## Prerequisites

- A React SPA with React Router (or similar)
- Vite as bundler
- TypeScript (recommended)

---

## Step 1: Install Dependencies

```bash
npm install -D vitest @vitest/browser-playwright playwright msw vitest-browser-react @testing-library/react @testing-library/jest-dom @frontend-testing/vitest-browser-navigate
```

| Package                                     | Purpose                              |
| ------------------------------------------- | ------------------------------------ |
| `vitest`                                    | Test runner                          |
| `@vitest/browser-playwright`                | Playwright provider for Vitest       |
| `playwright`                                | Browser automation                   |
| `msw`                                       | Mock Service Worker for HTTP mocking |
| `vitest-browser-react`                      | React renderer for browser mode      |
| `@testing-library/react`                    | DOM queries (screen, waitFor)        |
| `@testing-library/jest-dom`                 | Custom matchers (toBeVisible, etc.)  |
| `@frontend-testing/vitest-browser-navigate` | SPA navigation in browser mode       |

---

## Step 2: Configure Vitest

Create `vitest.config.ts` in the project root:

```typescript
import { navigate } from "@frontend-testing/vitest-browser-navigate";
import { playwright } from "@vitest/browser-playwright";
import { defineConfig } from "vitest/config";
import viteConfig from "./vite.config";

export default defineConfig({
  ...viteConfig,
  test: {
    setupFiles: ["./src/test/setup.ts"],
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [{ browser: "chromium" }],
      commands: {
        navigate,
      },
    },
  },
});
```

---

## Step 3: Create MSW Server

Create `src/test/mocks/server.ts`:

```typescript
import { setupWorker } from "msw/browser";

export const server = setupWorker();
```

---

## Step 4: Create stub-json-response Utility

Create `src/test/stub-json-response.ts`:

```typescript
import { http, HttpResponse, type JsonBodyType } from "msw";
import { vi, type Mock } from "vitest";
import { server } from "./mocks/server";

type HttpMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";

interface StubJsonResponseOptions {
  path: string;
  method?: HttpMethod;
  response: JsonBodyType | unknown;
  status?: number;
}

export interface RequestCall {
  url: string;
  method: string;
  body?: unknown;
  headers: Record<string, string>;
  params?: Record<string, string>;
}

const httpMethods = {
  GET: http.get,
  POST: http.post,
  PUT: http.put,
  PATCH: http.patch,
  DELETE: http.delete,
} as const;

export function stubJsonResponse({
  path,
  method = "GET",
  response,
  status = 200,
}: StubJsonResponseOptions): {
  spy: Mock<(call: RequestCall) => void>;
} {
  const spy = vi.fn<(call: RequestCall) => void>();

  const handler = httpMethods[method](path, async ({ request, params }) => {
    let body: unknown = undefined;
    try {
      body = await request.json();
    } catch {
      // Body might not be JSON or might be empty
    }

    const headers: Record<string, string> = {};
    request.headers.forEach((value, key) => {
      headers[key] = value;
    });

    spy({
      url: request.url,
      method: request.method,
      body,
      headers,
      params: params as Record<string, string>,
    });

    return HttpResponse.json(response as JsonBodyType, { status });
  });

  server.use(handler);

  return { spy };
}
```

---

## Step 5: Create Test Setup File

Create `src/test/setup.ts`:

```typescript
import { server } from "./mocks/server";
import "@frontend-testing/vitest-browser-navigate";
import "@testing-library/jest-dom/vitest";
import { cleanup } from "@testing-library/react";
import { afterAll, afterEach, beforeAll } from "vitest";

beforeAll(() => {
  server.start({ onUnhandledRequest: "warn" });
});

afterEach(() => {
  cleanup();
  server.resetHandlers();
});

afterAll(() => {
  server.stop();
});
```

If using React Query, also reset the query client in `afterEach`:

```typescript
import { queryClient } from "@/queries/query-client";

afterEach(() => {
  cleanup();
  server.resetHandlers();
  queryClient.removeQueries();
});
```

---

## Step 6: Add TypeScript Types (Optional)

Create `src/test/vitest-browser.d.ts`:

```typescript
import "@frontend-testing/vitest-browser-navigate";
```

This ensures TypeScript knows about the `navigate` command.

---

## Writing Tests

### Test File Structure

Tests should use `.spec.tsx` extension and be placed next to the component:

```
src/pages/
└── login-page/
    ├── login-page.tsx
    └── login-page.spec.tsx
```

### Basic Test Pattern

```typescript
import App from "@/App";
import { stubJsonResponse } from "@/test/stub-json-response";
import "@frontend-testing/vitest-browser-navigate";
import { screen, waitFor } from "@testing-library/react";
import { describe, expect, test } from "vitest";
import { render } from "vitest-browser-react";
import { commands, userEvent } from "vitest/browser";

describe("LoginPage", () => {
  test("should login and redirect to dashboard", async () => {
    // 1. Mock API responses BEFORE navigation
    stubJsonResponse({
      path: "*/auth/login",
      method: "POST",
      response: { token: "mock-token" },
    });

    // 2. Navigate to the route
    await commands.navigate("/login");

    // 3. Render the full app (not the page component)
    await render(<App />);

    // 4. Interact like a real user
    await userEvent.type(screen.getByPlaceholderText("Email"), "test@test.com");
    await userEvent.type(
      screen.getByPlaceholderText("Password"),
      "password123"
    );
    await userEvent.click(screen.getByRole("button", { name: "Login" }));

    // 5. Assert with waitFor (async)
    await waitFor(() => {
      expect(window.location.pathname).toBe("/dashboard");
    });
  });
});
```

### Key Points

| Concept                                | Explanation                                                                          |
| -------------------------------------- | ------------------------------------------------------------------------------------ |
| **Render `<App />`**                   | Always render the full app, not individual page components. This tests real routing. |
| **Navigate first**                     | Call `commands.navigate()` before `render()` to set the initial route.               |
| **Mock before navigate**               | Set up API mocks before navigation so they're ready when the page loads.             |
| **Use `waitFor`**                      | All assertions should be wrapped in `waitFor` since everything is async.             |
| **`render` from vitest-browser-react** | Do NOT use render from `@testing-library/react`.                                     |

---

## API Mocking Patterns

### Simple GET

```typescript
stubJsonResponse({
  path: "*/users",
  method: "GET",
  response: [{ id: "1", name: "John" }],
});
```

### POST with spy

```typescript
const { spy } = stubJsonResponse({
  path: "*/users",
  method: "POST",
  response: { id: "new-id", name: "New User" },
});

// Later: verify the request
await waitFor(() => {
  expect(spy).toHaveBeenCalledWith(
    expect.objectContaining({
      body: { name: "New User" },
    })
  );
});
```

### Error response

```typescript
stubJsonResponse({
  path: "*/users/123",
  method: "GET",
  response: { message: "Not found" },
  status: 404,
});
```

### Wildcard paths

Use `*` at the start to match any base URL:

```typescript
stubJsonResponse({
  path: "*/api/v1/users", // Matches http://localhost:3000/api/v1/users
  response: [],
});
```

---

## User Interactions

```typescript
// Type text
await userEvent.type(screen.getByPlaceholderText("Email"), "user@example.com");

// Click
await userEvent.click(screen.getByRole("button", { name: "Submit" }));

// Clear input
await userEvent.clear(screen.getByPlaceholderText("Search"));

// Select option (native select)
await userEvent.selectOptions(screen.getByRole("combobox"), "option-value");
```

---

## Assertions

```typescript
// Element is visible
await waitFor(() => {
  expect(screen.getByText("Success")).toBeVisible();
});

// Element is NOT present
await waitFor(() => {
  expect(screen.queryByText("Error")).not.toBeInTheDocument();
});

// URL changed
await waitFor(() => {
  expect(window.location.pathname).toBe("/dashboard");
});

// Multiple elements
await waitFor(() => {
  expect(screen.getAllByRole("listitem")).toHaveLength(3);
});

// With timeout for slow operations
await waitFor(
  () => {
    expect(screen.getByText("Loaded")).toBeVisible();
  },
  { timeout: 5000 }
);
```

---

## Testing Authenticated Pages

```typescript
import authService from "@/services/auth.service";

describe("DashboardPage", () => {
  beforeEach(() => {
    authService.setToken("mock-token");
  });

  afterEach(() => {
    authService.clearToken();
  });

  test("should display user info", async () => {
    stubJsonResponse({
      path: "*/users/me",
      response: { id: "1", email: "admin@test.com", permissions: ["ADMIN"] },
    });

    await commands.navigate("/dashboard");
    await render(<App />);

    await waitFor(() => {
      expect(screen.getByText("admin@test.com")).toBeVisible();
    });
  });
});
```

---

## Testing Dialogs

```typescript
test("should open dialog and submit", async () => {
  // Setup mocks...

  await commands.navigate("/page");
  await render(<App />);

  // Open dialog
  await userEvent.click(screen.getByRole("button", { name: "Open" }));

  // Verify dialog opened
  await waitFor(() => {
    expect(screen.getByRole("dialog")).toBeVisible();
  });

  // Fill and submit
  await userEvent.type(screen.getByPlaceholderText("Name"), "Test");
  await userEvent.click(screen.getByRole("button", { name: "Save" }));

  // Verify dialog closed
  await waitFor(() => {
    expect(screen.queryByRole("dialog")).not.toBeInTheDocument();
  });
});
```

---

## Running Tests

```bash
npx vitest              # Run all tests
npx vitest --watch      # Watch mode
npx vitest login        # Run tests matching "login"
npx vitest --ui         # Open Vitest UI
```

---

## Troubleshooting

### Test times out

- Increase timeout: `{ timeout: 5000 }`
- Check if API mocks are set up correctly
- Ensure navigation happens before render

### Element not found

- Use `screen.debug()` to see current DOM
- Verify API mocks return expected data
- Check if element requires auth/permissions

### Multiple elements found

Use more specific queries:

```typescript
screen.getByRole("button", { name: "Specific Name" });
screen.getByLabelText("Input label");
screen.getByTestId("unique-id");
```

### MSW not intercepting requests

- Ensure `server.start()` is called in setup
- Check path pattern matches (use `*` prefix for any base URL)
- Verify method matches (GET, POST, etc.)

---

## Project Structure

```
src/
├── test/
│   ├── mocks/
│   │   └── server.ts
│   ├── setup.ts
│   └── stub-json-response.ts
├── pages/
│   └── example-page/
│       ├── example-page.tsx
│       └── example-page.spec.tsx
└── App.tsx
```
