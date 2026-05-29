# Backend Testing Guidelines

## Philosophy

**Everything is tested through e2e tests.** We don't write unit tests for services or controllers:
real coverage comes from booting the Nest app, hitting HTTP endpoints with `supertest`, and verifying
the effect in MongoDB. Tests must exercise the same code path as production — controller → service →
repository → database.

The rule is:

- **Our own infrastructure is real**: MongoDB (atlas-local, single-node replica set with Atlas Search),
  S3 (LocalStack), the full Nest app with every module, pipe, filter, interceptor and middleware.
- **Third parties are mocked**: Stripe, Postmark, Mixpanel, GoHighLevel, Twilio, Meta Graph API, FX
  providers, etc. No outbound network calls in tests.
- **Never mock our own code** from inside a test. Needing to "skip" an internal service is almost
  always a sign that a test helper is missing, or that the code is too coupled and needs a refactor.

## Test layout

```
backend/test/
  ├── jest-config-e2e.ts           # Jest config (testMatch *.e2e-spec.ts)
  ├── jest-global-setup.ts         # Boots Mongo + LocalStack ONCE (parent process)
  ├── jest-global-teardown.ts
  ├── setup-tests.ts               # Per worker: starts MSW, mocks for util/dotenv
  ├── msw-server.ts                # Shared MSW server
  ├── __mocks__/                   # Low-level mocks (dd-trace, etc.)
  ├── fixtures/                    # Images and binary assets
  └── src/
      ├── test-app/
      │   ├── setup-mongo.ts       # testcontainers → mongodb-atlas-local
      │   ├── setup-localstack.ts  # testcontainers → S3 local
      │   ├── setup-msw-stubs.ts   # MSW handlers for third-party HTTP
      │   ├── setup-app.ts         # AppModule bootstrap + seeds + syncIndexes
      │   ├── test-app.ts          # TestApp wrapper exposing service helpers
      │   ├── test-coach.service.ts
      │   ├── test-library.service.ts
      │   ├── test-workout.service.ts
      │   └── dummies/             # Response factories for third parties (Stripe, Postmark...)
      └── *.e2e-spec.ts            # e2e specs
```

Specs live in `backend/test/src/*.e2e-spec.ts`. They are **not** colocated with their modules: they sit
in a parallel tree, grouped by functional domain (`programs-copy-paste-sets.e2e-spec.ts`,
`token-refresh.e2e-spec.ts`, `outrank-webhook.e2e-spec.ts`...).

## Commands

```bash
npm run test       # Unit tests (very few). Useful only for pure utilities.
npm run test:e2e   # The important ones. Boots containers and runs the full suite.
```

CI runs with `maxWorkers: 1`; locally we go up to 4 parallel workers.

## How the environment boots

1. **Global setup (once, parent process):**
   - Loads `backend/test/.env.e2e-test`.
   - Forces `NODE_ENV=test` and `TZ=UTC`.
   - In parallel:
     - Starts `mongodb/mongodb-atlas-local:8.0.20` with `testcontainers`. We wait for
       "Waiting for connections" **and** for `mongosh` to answer `ping` (the replica set isn't ready
       just because the log line fired).
     - Starts `localstack/localstack:3.5.0` and seeds S3 with the assets in `default-cdn-resources/`.
   - Exports `MONGODB_URI` with `?directConnection=true` so workers connect to the node directly
     (atlas-local advertises itself as `trainer-db`, unreachable from the host).

2. **Per worker (`setup-tests.ts`):**
   - Starts MSW (`server.listen({ onUnhandledRequest: 'bypass' })`).
   - `afterEach`: `server.resetHandlers()` so a test never leaks into the next.
   - `afterAll`: closes MSW.

3. **Per suite (`TestApp.start()`):**
   - Compiles the real `AppModule` via `Test.createTestingModule()`.
   - Overrides **only** providers that talk to non-HTTP SDKs (Twilio, Meta Graph API client) with
     in-process mocks.
   - `app.listen(0)` → random port, no collisions between workers.
   - **Re-seeds** the DB: `db.dropDatabase()` + seeds (muscle groups, metric units, company,
     exercises). The Mongo container is shared between suites, so dropping avoids collisions with
     unique indexes left over from previous suites.
   - `connection.syncIndexes({ continueOnError: true })`: Mongoose builds indexes in the background
     after first use; without this, dedup tests (idempotencyKey, slug, providerRefs.\*) can race the
     index build and both inserts succeed.

## Mocking third parties (MSW)

Every outbound HTTP call to a third party is intercepted with [MSW](https://mswjs.io). Handlers live
in `setup-msw-stubs.ts` and are loaded as the server's initial handlers, so **they survive
`resetHandlers()` between tests**.

What we already cover:

- **Stripe**: customers, subscriptions, products, prices, coupons (with an in-memory `couponStore` to
  simulate stateful behaviour), payment intents, invoices.
- **Postmark**: `/email` and `/email/withTemplate`.
- **Mixpanel**: `track`, `groups`, `engage`.
- **GoHighLevel**: `contacts/upsert`, custom fields, `PUT contacts/:id`.
- **FX rates** (`open.er-api.com`): fixed rates per currency.
- **Test images**: any `https://test-images.localhost/*` returns the buffer at
  `fixtures/exercise-image.jpg`.

### Rules for adding a new third party

1. If the SDK speaks HTTP (Stripe, Postmark, etc.), **add an MSW handler** — do NOT mock the SDK
   client. That way we also exercise our adapter layer.
2. If the SDK uses a binary channel / WebSocket / SMS (Twilio, Meta Graph), **override the provider
   in `setup-app.ts`** with `.overrideProvider(...).useValue(mock)`.
3. Put response factories in `test-app/dummies/`. No naked object literals inside handlers — they rot
   the moment the shape changes.
4. If the third party is stateful (coupons, subscriptions), implement an in-memory store inside the
   handler. Stripe coupons is the reference example.
5. For per-test overrides, call `server.use(http.post(...))` inside a `beforeEach` or `it`. They are
   cleaned up automatically in `afterEach`.

## Data: never assume a clean DB

Even though `setup-app.ts` drops the DB **at the start of the suite**, suites share the Mongo
container and tests within a suite are not isolated from each other. Therefore:

- **Every test creates its own data** with unique identifiers. Use `Date.now()`, `faker` or
  `ObjectId()` to avoid collisions (`refresh-ok+${timestamp}@example.com`, `Coach-${currentTime}`).
- **Never clean up data at the end of a test.** There is no `afterEach` wiping collections. If you
  add one, you'll break other tests in the suite.
- **Don't rely on execution order.** Jest may parallelize specs; within a suite `it` blocks run in
  order but you must never depend on state left behind — always create what you need.
- **When you need to authenticate**, create a coach with `TestCoachService.createCoach()` and use the
  `accessToken` it returns. Don't reuse tokens across tests.

## Test helpers (`TestApp` + services)

`TestApp` exposes the `HttpServer` and a set of "test services" wrapping full flows so specs don't
repeat setup:

```ts
const testApp = new TestApp();
await testApp.start();

const coachService    = testApp.getCoachService();    // create coach, customers, exercises...
const libraryService  = testApp.getLibraryService();
const workoutService  = testApp.getWorkoutService();

const coach    = await coachService.createCoach();
const customer = await coachService.createCustomer(coach);
```

Rules for test services:

- **Never reach into the DB** to create entities — hit the same HTTP endpoints the frontend hits.
  That way an API change breaks the test (which is what we want).
- Exception: to grant a role that's unreachable via the API (e.g. `SUPERADMIN`), a targeted
  `db.collection('users').updateOne(...)` followed by a fresh login is acceptable.
- Return types derived from the controller (`Awaited<ReturnType<XxxController['method']>>`) so the
  compiler catches drift between API and tests.
- Obvious invariant assertions (token exists, `role` is CUSTOMER, etc.) belong in the helper, not in
  every spec.

## Spec shape

```ts
import request from 'supertest';
import { TestApp } from './test-app/test-app';
import { TestCoachService } from './test-app/test-coach.service';

describe('Token Refresh (e2e)', () => {
  const testApp = new TestApp();
  let coachService: TestCoachService;

  beforeAll(async () => {
    await testApp.start();
    coachService = testApp.getCoachService();
  });

  afterAll(async () => {
    await testApp.stop();
  });

  it('refreshes valid tokens', async () => {
    const ts = Date.now();
    const email = `refresh+${ts}@example.com`;
    const password = 'TestPassword1234';

    await coachService.createCoach({ coachEmail: email, coachPassword: password });

    const login = await request(testApp.getHttpServer())
      .post('/users/login')
      .send({ email, pwd: password, app: 'COACH_DASHBOARD' })
      .expect(201);

    const refreshed = await request(testApp.getHttpServer())
      .post('/users/refresh')
      .send({ oldToken: login.body.accessToken, refreshToken: login.body.refreshToken })
      .expect(201);

    expect(refreshed.body.accessToken).toBeDefined();
    expect(refreshed.body.refreshToken).toBeDefined();
  });
});
```

## Quick rules

- ❌ **Don't** mock a `Service` with `jest.mock()` to "avoid" hitting the DB. If the logic matters,
  it goes through an e2e with a real DB.
- ❌ **Don't** create MSW handlers inside an `it` without restoring them. If they're global, put them
  in `setup-msw-stubs.ts`; if they're per-test, in `beforeEach` + automatic reset.
- ❌ **Don't** drop collections, drop the DB, or close the `MongoClient` from a spec. `setup-app.ts`
  owns that.
- ❌ **Don't** hardcode ports: `app.listen(0)` and `getMappedPort()` avoid collisions.
- ✅ **Do** add a helper in `test-coach.service.ts` (or the right one) when several specs start
  repeating the same HTTP flow.
- ✅ **Do** assert response shape inside the helper to catch accidental API changes.
- ✅ **Do** use `faker` for unique names/emails and `Date.now()` as a suffix for unique identifiers.
- ✅ **Do** model stateful third parties (coupons, subscriptions) with in-memory stores inside the
  handler.
