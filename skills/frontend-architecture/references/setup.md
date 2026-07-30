# Frontend Architecture Setup

Use this guide once to establish or deliberately migrate the project-wide architecture. Do not repeat setup for ordinary feature work.

## 1. Inventory before changing

Record the current:

- Source root and import aliases
- Router and route-definition pattern
- Query/cache library and provider placement
- HTTP client and authentication integration
- Existing page, component, repository, service, and type locations
- Test, typecheck, lint, and build commands
- Import-boundary or circular-dependency rules

Preserve working repository conventions unless the setup task explicitly replaces them. Treat migration as a separate behavior-preserving change from feature implementation.

## 2. Define the project contract

Choose and document these roots only when the application needs them:

```text
src/
├── client/
├── components/
├── constants/
├── pages/
├── queries/
├── repositories/
├── services/
└── types/
```

Establish:

- `@/` as the cross-root source alias
- Local `./` imports within a feature
- Dash-case homonymous folders for pages and components
- One React component per `*.view.tsx`
- Colocated `*.controller.ts`, `*.route.tsx`, and tests when required
- Resource-grouped query and mutation hooks
- No barrel files

Do not create every root or placeholder layer up front. Create only the shared infrastructure and one representative feature required to prove the contract.

## 3. Establish shared infrastructure

Configure one singleton API client that owns only:

- Base URL and transport configuration
- Shared headers
- Request and response interceptors
- Per-request access to the current authentication token

Configure the query client and its root provider once. Keep resource endpoints out of the API client and put them in repositories. Keep cache behavior in query and mutation hooks.

Define route constants once. Keep each route object beside its page and assemble route objects at the router root.

## 4. Enforce dependency direction

Configure the repository's existing lint tool to reject:

- Child-to-parent component imports
- Sibling-to-sibling component imports
- Views importing repositories or the API client
- Repositories importing React, controllers, or views
- Upward relative traversal such as `../` inside component subtrees
- Circular dependencies and barrel imports

Prefer an existing boundary plugin already used by the repository. Express rules in terms of the actual source alias and folders instead of adding a second overlapping lint system.

If complete automated enforcement is not practical, enforce the highest-risk boundaries first:

1. No upward or sibling component imports
2. No view-to-data-access imports
3. No inward layer depending on React or transport details

Make each rule fail against a temporary violating example before accepting it. Remove the violation after proving the rule.

## 5. Build one vertical slice

Create or migrate one small real behavior through:

```text
route → page view → controller → query/mutation → repository → API client
```

Include only the layers that behavior needs. Add:

- A behavior-focused test
- A nested child component to prove recursive ownership when relevant
- A server read or write to prove the data boundary when relevant
- Explicit cache invalidation or update after a mutation
- A pure service only if the slice contains a business rule

Do not use a synthetic example if a small existing feature can prove the same structure.

## 6. Verify setup

Confirm:

- A new developer can locate the route, view, orchestration, and data access from the feature name.
- The representative behavior passes.
- A deliberate boundary violation fails lint.
- Typecheck, lint, tests, and build pass.
- No empty layers or speculative shared abstractions remain.
- Existing unaffected behavior still passes.

After these checks, consider setup complete. Handle subsequent features and reviews with the default workflow in `SKILL.md`.
