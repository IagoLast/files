# Writing Frontend Browser Tests

Describe complete user behavior while replacing only external boundaries.

## Choose the render scope

Prefer the complete application when the scenario involves routing, layouts, providers, authentication, caching, or collaboration across features:

```tsx
await commands.navigate("/bookings");
await render(<App />);
```

Render a representative fragment when the application root cannot reasonably run in the harness or when the behavior is genuinely local:

```tsx
await render(
  <AppProviders>
    <BookingForm />
  </AppProviders>,
);
```

Keep the fragment's real children, providers, form logic, and server interaction. The fragment is a pragmatic boundary, not an excuse to mock every collaborator. Choose the scope that gives confidence in the real use case with the least incidental setup.

## Canonical test shape

```tsx
import { stubJsonResponse } from "@frontend-testing/server-stubs";
import { commands, page } from "vitest/browser";
import { render } from "vitest-browser-react";
import { describe, expect, test } from "vitest";
import App from "@/App";

describe("Bookings page", () => {
  test("shows the new booking after the user creates it", async () => {
    stubJsonResponse({
      path: "*/api/bookings",
      response: { data: [] },
    });

    stubJsonResponse({
      path: "*/api/bookings",
      method: "POST",
      response: {
        id: "booking-1",
        customerName: "Ana",
      },
      status: 201,
    });

    await commands.navigate("/bookings");
    await render(<App />);

    await page.getByRole("button", { name: "Create booking" }).click();
    await page.getByLabelText("Customer name").fill("Ana");
    await page.getByRole("button", { name: "Save" }).click();

    await expect
      .element(page.getByText("Booking created for Ana"))
      .toBeVisible();
  });
});
```

Adjust imports to the installed Vitest and renderer versions. Preserve the sequence: stub, authenticate if needed, navigate, render, interact, assert.

## Decide what to stub

Stub a request when the real application would cross the server boundary during the scenario:

- Initial page data
- Form submissions
- Follow-up requests after mutations
- Error responses
- Retry or polling responses
- File-upload responses

Usually keep these real:

- Components or layouts
- Router behavior
- Form libraries or validation
- Query/mutation hooks
- Repositories or the HTTP client
- State stores
- Browser APIs that the real browser supports

Replace an internal collaborator only when the normal boundary is impractical in the test harness and the replacement does not bypass the behavior being specified. If the browser contacts a third-party API directly, intercept that network request too. The principle is about useful behavioral confidence, not rigid isolation.

## Use `stubJsonResponse`

Static response:

```ts
stubJsonResponse({
  path: "*/api/bookings",
  response: { data: [] },
});
```

Error:

```ts
stubJsonResponse({
  path: "*/api/bookings",
  method: "POST",
  response: { message: "Booking overlaps an existing booking" },
  status: 409,
});
```

Dynamic response:

```ts
stubJsonResponse({
  path: "*/api/bookings",
  method: "POST",
  response: ({ body }) => ({ id: "booking-1", ...body }),
});
```

Sequential responses:

```ts
stubJsonResponse({
  path: "*/api/bookings",
  responses: [
    { response: { message: "Unavailable" }, status: 503 },
    { response: { data: [] }, status: 200 },
  ],
});
```

The returned spy captures `url`, `method`, parsed `body`, `headers`, route `params`, and the original request. Use it when the request itself is a required client-server contract, but also assert the resulting user-visible behavior. For ordinary use cases, the rendered result is stronger evidence than an internal call.

## Authentication decision

Use this order:

1. If testing login, log in through the UI and stub the login/session endpoints.
2. If testing an authenticated feature, use an existing application-level auth test helper.
3. Otherwise initialize the real token, cookie, or session storage contract before rendering.
4. Mock an auth boundary only when the earlier options are impractical.

Example storage precondition:

```ts
beforeEach(() => {
  authTestHelper.authenticate({
    id: "user-1",
    permissions: ["bookings:write"],
  });
});

afterEach(() => {
  authTestHelper.reset();
});
```

Prefer a helper that uses the same public storage/session contract as production. Avoid replacing the route guard, auth hook, page, or permission-dependent UI because doing so removes behavior from the test. When authorization affects server data, make stubbed responses consistent with the chosen identity.

## Query as a user

Prefer, in order:

1. `getByRole` with an accessible name
2. `getByLabelText`
3. Visible text
4. Placeholder text when it is meaningful
5. Test ID as a last resort

Use locator assertions:

```ts
await expect.element(page.getByRole("alert")).toHaveTextContent(
  "Booking overlaps an existing booking",
);
```

Avoid querying CSS classes or inspecting React state. A failing accessible query can reveal a real accessibility defect.

## Async behavior

- Await every `fill`, `click`, navigation, render, and async assertion.
- Use Vitest browser locator retrying rather than manual polling.
- Assert transient state only if it is a requirement and the response timing makes it observable deterministically.
- For retries, use sequential stub responses instead of timers or sleeps.
- For debounce or long time-based pure logic, prefer a focused unit test unless browser integration is the requirement.

## Test design

- Name the behavior, initial condition, and expected result.
- Keep Arrange, Act, and Assert visually clear.
- Make one scenario fail for one behavioral reason.
- Use concrete values and explicit expected results.
- Cover a representative success, meaningful failure, and boundary case; do not repeat equivalent browser flows.
- Prove test sensitivity by seeing it fail before implementing the behavior.

## Review checklist

- Does the test run in a real browser?
- Does it render the widest practical boundary for the use case?
- If it renders a fragment, does that fragment retain the real collaborators relevant to the behavior?
- Are all required HTTP stubs registered before navigation/render?
- Is only the external boundary replaced?
- Does the test interact through accessible user-facing locators?
- Are assertions about visible behavior rather than implementation?
- Is authentication prepared through the narrowest stable boundary?
- Is there a user-visible outcome rather than only a function or spy assertion?
- Are request-spy assertions limited to actual API-contract requirements and supported by an observable result?
- Are state and handlers reset after the test?
- Does the focused test fail without the behavior and pass with it?
- Were the browser suite, typecheck, lint, and build run?
