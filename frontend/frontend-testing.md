# Frontend testing

- Test should be as close as a real use case as possible.
  - Do not test implementation.
- Only server calls are mocked (avoid side effects).
- Render the entire app and navigate using the router to test real user flows.

## Test Setup

Tests use Vitest with React Testing Library. The test setup includes:

- **`src/test/setup.ts`**: Global test configuration including cleanup and jest-dom matchers
- jsdom environment virtualizes the browser, including the router

## Test Structure

Each page should have a `*.spec.tsx` file alongside the view and route files:

```
pages/
└── home-page/
    ├── home-page.route.tsx
    ├── home-page.view.tsx
    └── home-page.spec.tsx
```

## Testing Pattern

### Basic Test Example

```tsx
import { describe, it, expect } from "vitest";
import { screen, waitFor } from "@testing-library/react";
import { render } from "@testing-library/react";
import App from "@/App";

describe("HomePage", () => {
  it("should display the home page content when navigating to root path", async () => {
    render(<App />);

    await waitFor(() => {
      expect(screen.getByText("Home Page")).toBeVisible();
    });
  });
});
```

### Complete Test Example with Server Mocking

```tsx
import { describe, it, expect } from "vitest";
import { screen, waitFor } from "@testing-library/react";
import { render } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "@/App";

describe("TodoListPage", () => {
  it("should display a list of todos when the server returns a valid todo list", async () => {
    // Mock server response
    const mockTodos = [
      { id: "1", text: "Buy groceries", completed: false },
      { id: "2", text: "Walk the dog", completed: true },
    ];

    // TODO: Set up server mocking here when HTTP client is configured
    // stubJsonResponse({
    //   path: 'api/v1/todos',
    //   method: 'get',
    //   response: mockTodos,
    // });

    // Render app - jsdom virtualizes the router
    render(<App />);

    // Wait for content to appear
    await waitFor(() => {
      expect(screen.getByText("Buy groceries")).toBeVisible();
    });

    // Interact with the page like a user would
    const user = userEvent.setup();
    const completeButton = screen.getByRole("button", { name: /complete/i });
    await user.click(completeButton);

    // Assert like a human would
    await waitFor(() => {
      expect(screen.getByText("Buy groceries")).toHaveClass("completed");
    });
  });
});
```

## Key Principles

1. **Render the entire app**: Use `render(<App />)` directly - jsdom virtualizes the router
2. **Use descriptive test descriptions**: Test names should describe what the user sees or does
3. **Assert like a human**: Check for visible text, roles, and user-visible elements
4. **Wait for async operations**: Use `waitFor()` when content loads asynchronously
5. **Mock only server calls**: Mock API responses, not component logic or React hooks
6. **Test real use cases**: Test the full user journey, not implementation details

## Running Tests

```bash
# Run tests in watch mode
npm run test

# Run tests once
npm run test:run

# Run tests with UI
npm run test:ui
```
