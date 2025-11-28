# Backend Testing Guidelines

## E2E Tests

- Use **real external services** in e2e tests - no mocks.
- Use **testcontainers** for databases (PostgreSQL).
- Use **Clerk development credentials** for authentication tests.
- **Colocate tests** with the modules they test (e.g., `src/modules/auth/auth.e2e.spec.ts`).

## Test File Location

Tests are colocated with their modules:
```
src/modules/auth/
  ├── auth.module.ts
  ├── auth.controller.ts
  ├── auth.service.ts
  └── auth.e2e.spec.ts    # Tests for this module

src/modules/users/
  ├── users.module.ts
  ├── users.controller.ts
  └── users.e2e.spec.ts   # Tests for this module
```

## Test Setup

Environment variables for e2e tests are configured in:
- `test/setup-files/jest.setup.ts` - Global setup (runs before all tests)
- `test/setup-files-after-env/jest.setup.ts` - Per-test setup (sets env vars)
- `test/constants.ts` - Shared test constants (test user credentials, etc.)

Database container config is shared via `.test-db-config.json` file between the global setup process and test processes.

## Test User for Auth

A test user must exist in the Clerk development environment with known credentials. Credentials are defined in `test/constants.ts`.
