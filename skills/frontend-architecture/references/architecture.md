# Frontend Architecture Reference

## Dependency map

```text
router
  └─ page route
      └─ page view
          └─ controller
              ├─ query/mutation hook
              │   └─ repository
              │       └─ API client
              └─ pure service
```

Dependencies point downward. Data and handlers return upward. Views do not know HTTP details, repositories do not know React, and the API client does not know domain resources.

## Canonical tree

```text
src/
├── router.tsx
├── client/
│   └── api-client.ts
├── components/
│   └── button/
│       ├── button.view.tsx
│       ├── button.spec.tsx
│       └── components/
│           └── button-icon/
│               └── button-icon.view.tsx
├── constants/
│   └── routes.ts
├── pages/
│   └── booking-list/
│       ├── booking-list.route.tsx
│       ├── booking-list.view.tsx
│       ├── booking-list.controller.ts
│       ├── booking-list.spec.tsx
│       └── components/
│           ├── booking-item/
│           │   ├── booking-item.view.tsx
│           │   ├── booking-item.spec.tsx
│           │   └── components/
│           │       └── booking-actions/
│           │           ├── booking-actions.view.tsx
│           │           └── components/
│           │               └── grandchild-component/
│           │                   └── grandchild-component.view.tsx
│           └── booking-details/              # nested subpage
│               ├── booking-details.route.tsx
│               ├── booking-details.view.tsx
│               ├── booking-details.controller.ts
│               └── components/
│                   └── booking-summary/
│                       ├── booking-summary.view.tsx
│                       └── components/
│                           └── booking-status/
│                               └── booking-status.view.tsx
├── queries/
│   └── bookings/
│       ├── list-bookings.query.ts
│       └── create-booking.mutation.ts
├── repositories/
│   └── bookings.repository.ts
├── services/
│   └── booking-policy.service.ts
└── types/
    └── booking.ts
```

Do not reproduce the entire tree for every feature. Create only files required by current behavior.

The structure repeats recursively at every depth for both components and subpages. Every owner may have a `components/` folder; every child inside it receives its own folder and may in turn own another `components/` folder. A nested page follows the same rule as any other component and keeps its route, view, controller, test, and children together.

```text
owner/
├── owner.view.tsx
└── components/
    └── child/
        ├── child.view.tsx
        └── components/
            └── grandchild/
                ├── grandchild.view.tsx
                └── components/
                    └── ...
```

## Layer responsibilities

### Router and route files

- Define paths once in `constants/routes.ts`.
- Keep `*.route.tsx` beside its page.
- Compose route objects in `router.tsx`.
- Use nested routes for nested page ownership.
- Put route-level redirects, authentication gates, and required preloading in loaders.
- Keep domain data fetching in query hooks unless the router API specifically owns that data.

### Views and components

- Render props and controller results.
- Contain markup, accessibility attributes, and presentation-only derivations.
- Export one component per view file.
- Place every owned child component or subpage under the owner's `components/` folder.
- Give each child its own folder, including its optional controller, test, route, and `components/` descendants.
- Apply this convention recursively with no depth exception.
- Keep state at the closest owner; add a controller when state or orchestration obscures rendering.
- Extract a child when it has a distinct responsibility, independent test surface, or meaningful reuse.

### Controllers

- Coordinate router params, navigation, forms, local state, queries, and mutations.
- Expose view-ready values and named handlers.
- Do not duplicate repository or domain logic.
- Remain optional for genuinely simple components.

### Queries and mutations

Group hooks by resource:

```text
queries/
└── bookings/
    ├── list-bookings.query.ts
    ├── get-booking.query.ts
    ├── create-booking.mutation.ts
    └── cancel-booking.mutation.ts
```

Use a query for a read and a mutation for a write. The resource folder owns only async cache hooks; endpoint paths and response mapping remain in the corresponding repository.

```ts
export function useListBookingsQuery() {
  return useQuery({
    queryKey: ["bookings", "list"],
    queryFn: bookingsRepository.list,
  });
}

export function useCreateBookingMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: bookingsRepository.create,
    onSuccess: () =>
      queryClient.invalidateQueries({ queryKey: ["bookings", "list"] }),
  });
}
```

- Give query keys stable resource-to-detail hierarchy.
- Keep query functions free of UI behavior.
- Handle cache invalidation or direct cache updates explicitly after writes.
- Surface transport failures; translate them to user-facing state at the controller/view boundary.

### Repositories

```ts
async function list(): Promise<Booking[]> {
  const response = await client.get<BookingResponse[]>("/bookings");
  return response.data.map(mapBooking);
}

export default { list };
```

- Own endpoint paths, HTTP verbs, request DTOs, response DTOs, and mapping.
- Return domain-friendly data rather than raw response envelopes.
- Do not own caching, notifications, navigation, or React state.

### API client

- Export one configured instance.
- Read the base URL from environment configuration.
- Apply shared headers and request/response interceptors.
- Ask the authentication service for the current token per request.
- Do not expose resource-specific methods from the generic client.

### Services

- Hold reusable business rules as pure functions.
- Depend on domain types and constants, not React or the HTTP client.
- Group closely related functions in a named/default service object when that matches the codebase.

## Import boundaries

Within a recursive component subtree, allow:

```text
own view → own controller
parent view → child view
any view → global shared component
controller → query/mutation/service
query/mutation → repository
repository → API client
```

Reject:

```text
child → parent
sibling → sibling
view → repository or API client
repository → query/controller/view
API client → repository/domain resource
```

If siblings need the same behavior, lift it to their nearest common owner. If unrelated features need it, promote a stable abstraction to the appropriate root folder.

## Review checklist

- Does a test define the requested behavior?
- Does the route live beside the page and use a centralized path constant?
- Does each view file export exactly one component?
- Are child components nested under their owner?
- Does every nested component or subpage repeat the same `components/<child>/` structure recursively?
- Is rendering separate from orchestration?
- Are server reads queries and writes mutations?
- Are endpoint and mapping details confined to repositories?
- Is all HTTP transport routed through the singleton client?
- Are cache effects after mutations explicit?
- Do imports follow the dependency direction without barrels or upward traversal?
- Were tests, typecheck, lint, and build run?
