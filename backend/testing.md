# Backend Testing Guidelines

## E2E Tests

- Use **real external services** in e2e tests - no mocks.
- Use **testcontainers** for databases (PostgreSQL).
- Use **Clerk development credentials** for authentication tests.
- **Colocate tests** with the modules they test (e.g., `src/modules/auth/auth.e2e.spec.ts`).
- **Never assume a clean database**. Tests should work with existing data in the DB. This makes tests more realistic and allows concurrent execution.
- Use **@faker-js/faker** to generate unique test data (names, emails, etc.).
- Use **timestamps or UUIDs** in test data names to ensure uniqueness (e.g., `Company_${Date.now()}`).
- **Do not clean up data** before tests. Each test should create its own unique data and verify only that data.
- If a role/entity already exists, use `findOne` + create only if not found, or use unique names per test run.

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

**Requirements for the test user:**

1. Must exist in Clerk with the email/password defined in `test/constants.ts`
2. Must have the `SUPER_ADMIN` role assigned in the local database
3. The `SUPER_ADMIN` role must exist in the `roles` table with all permissions
