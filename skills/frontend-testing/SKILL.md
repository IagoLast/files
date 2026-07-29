---
name: frontend-testing
description: Write, configure, refactor, or review behavior-focused tests for Vite SPAs using Vitest Browser Mode, real browser navigation, user-visible assertions, and minimal HTTP stubbing through @frontend-testing/server-stubs and MSW. Prefer rendering the whole application and use a representative fragment when necessary. Use for React user flows, routes, dialogs, forms, authentication-dependent pages, server states, browser APIs, or replacing fragile implementation-coupled tests.
---

# Frontend Testing

Test a Vite SPA from the user's point of view in a real browser. Keep as much production behavior as practical and replace boundaries only when they would make the test external, slow, or nondeterministic.

## Core workflow

1. Inspect the existing Vite/Vitest config, test setup, application root, router, query cache, authentication boundary, and a nearby browser test.
2. Choose the widest useful render scope. Prefer rendering the complete application. Render a representative fragment when the whole app is impractical or adds no confidence to the use case.
3. Express one observable behavior with a test that fails for the intended reason.
4. Stub only HTTP responses required by that scenario with `stubJsonResponse`.
5. Set the initial route with `await commands.navigate(path)` when routing is part of the scenario.
6. Render `<App />` or the chosen fragment with the real providers it needs.
7. Interact through accessible labels, roles, names, and visible text as a user would.
8. Assert the result a user can observe, such as a visible dialog, navigation, validation message, disabled action, or updated content.
9. Run the focused browser test after each change, then the affected suite, typecheck, lint, and build.

Default ordering:

```text
stub server → prepare auth if necessary → navigate if relevant → render target → interact → observe
```

## Guiding principles

- Prefer Vitest Browser Mode with Playwright or WebdriverIO for SPA behavior that depends on the browser. Keep focused unit tests for pure logic where a browser adds no value.
- Prefer rendering the complete `<App />` with its real providers and router. Use a fragment when full-app rendering is unavailable, disproportionately expensive, or irrelevant to the behavior; keep the fragment large enough to represent the real use case.
- Keep internal application code real by default: controllers, hooks, repositories, query clients, routers, and child components should normally collaborate as in production.
- Preserve the real path through routing, forms, validation, HTTP client, state, caching, error handling, and CSS whenever those pieces contribute to the behavior.
- Prefer replacing server communication at the network boundary with `stubJsonResponse` instead of mocking `fetch`, Axios, or repository functions.
- Register stubs before the application can issue their requests.
- Treat unexpected network requests as test failures when practical; allow documented harness exceptions.
- Prefer accessible locators: role, label, name, placeholder, and visible text. Use test IDs only when no user-facing selector exists.
- Assert what the user sees or can do. A function being called with a parameter does not prove the use case; a dialog becoming visible, a route changing, or an error appearing does.
- Treat spies and internal-call assertions as supporting evidence only when the call itself is a required external contract. They should not replace an observable outcome.
- Avoid component state, private methods, hook calls, and incidental DOM structure because they couple tests to implementation.
- Await interactions, navigation, async rendering, and locator assertions. Prefer retrying browser assertions over arbitrary sleeps.
- Keep each test independent. Reset MSW handlers, rendered UI, query state, storage, and authentication state between tests.
- Use concrete inputs and expected outputs. Avoid reproducing production logic inside the test.
- Use focused unit tests for pure, combinatorial business logic; use browser tests to prove real user cases and collaboration between pieces.

## Authentication

- If authentication is the behavior under test, perform the login flow through the real UI and stub only its server requests.
- If authentication is merely a precondition, establish it through the narrowest stable application boundary, such as the existing auth test helper or production token/session storage contract.
- If no practical public setup exists, replace the narrowest authentication boundary that unlocks the scenario while keeping route, authorization, and downstream behavior real.
- Use the least privileged representative identity required by the scenario and reset its state after the test.

## Load detailed guidance

- Read [references/setup.md](references/setup.md) when installing or configuring Vitest Browser Mode, MSW, server stubs, or SPA navigation.
- Read [references/writing-tests.md](references/writing-tests.md) before adding or reviewing a test. It contains the canonical test shape, auth choices, stubbing patterns, and review checklist.
