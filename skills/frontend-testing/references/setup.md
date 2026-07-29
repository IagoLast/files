# Vitest Browser Setup

Configure the smallest browser harness that can run a Vite SPA or a representative application fragment.

## Dependencies

Adapt the package manager and versions to the project:

```bash
npm install --save-dev \
  vitest \
  @vitest/browser-playwright \
  playwright \
  vitest-browser-react \
  msw \
  @frontend-testing/server-stubs \
  @frontend-testing/vitest-browser-navigate
```

Use WebdriverIO instead of Playwright only when the project already standardizes on it.

## Vitest configuration

Merge the browser block into the existing Vite/Vitest configuration:

```ts
// vitest.config.ts
import { navigate } from "@frontend-testing/vitest-browser-navigate";
import { playwright } from "@vitest/browser-playwright";
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    setupFiles: ["./src/test/setup.browser.ts"],
    include: ["src/**/*.browser.test.{ts,tsx}"],
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [{ browser: "chromium" }],
      commands: { navigate },
    },
  },
});
```

Keep the project's existing plugins, aliases, environment, and Vite options. If browser and Node tests coexist, separate them by project or include pattern rather than forcing both into one environment.

## MSW worker setup

Configure the worker once:

```ts
// src/test/setup.browser.ts
import "@frontend-testing/vitest-browser-navigate";
import { serverManager } from "@frontend-testing/server-stubs";
import { setupWorker } from "msw/browser";
import { afterAll, afterEach, beforeAll } from "vitest";
import { cleanup } from "vitest-browser-react";

const worker = setupWorker();

serverManager.setDefaultServerLoader(() => worker);

beforeAll(async () => {
  await worker.start({ onUnhandledRequest: "error" });
});

afterEach(async () => {
  await cleanup();
  worker.resetHandlers();
});

afterAll(async () => {
  await worker.stop();
  serverManager.reset();
});
```

If the application intentionally performs requests that cannot be intercepted, understand them before changing `onUnhandledRequest` to `"warn"`. Do not globally bypass application API calls.

Reset any singleton state owned by the application:

```ts
afterEach(() => {
  queryClient.clear();
  localStorage.clear();
  sessionStorage.clear();
});
```

Prefer the application's dedicated reset helpers when they exist. Do not clear unrelated host state in a shared harness.

## Navigation

The custom command updates browser history, dispatches `popstate`, and waits for the SPA router:

```ts
import { commands } from "vitest/browser";

await commands.navigate("/bookings/42?tab=history");
```

Register `commands: { navigate }` in Vitest and import the package once for TypeScript augmentation. Prefer the real SPA router when routing contributes to the behavior; a smaller router harness is acceptable for a fragment whose use case does not depend on the full route tree.

## File naming

Follow the repository's convention. When browser and Node tests need an explicit split, use:

```text
feature.browser.test.tsx
pure-service.test.ts
```

Keep a browser test beside the route or feature it specifies unless the repository has an established integration-test tree.

## Setup verification

Before writing a complex test, prove that the harness can:

1. Render the real application or a representative fragment.
2. Navigate to a public route.
3. Intercept one API request.
4. Fail on an unexpected request.
5. Clean state between two tests.
