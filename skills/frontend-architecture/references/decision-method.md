# Frontend Architecture Decision Method

Use architecture to reduce the human effort required to understand and change the system. Prefer evidence about change impact over the number of layers, files, or patterns used.

## Change-impact review

1. Name one likely change in concrete terms, such as replacing the HTTP provider, adding a second renderer, changing a validation rule, or sharing a filter across two views.
2. Identify the modules that know about the changing detail.
3. Predict which files must change together to preserve behavior.
4. Assess the dependency using:
   - **Strength:** How difficult is the dependency to detect and refactor?
   - **Locality:** How far apart are the dependent elements?
   - **Degree:** How many elements must agree or change together?
   - **Stability:** If strength, distance, or degree is high, is the shared interface stable enough to justify it?
5. Prefer the design that keeps strong dependencies local, limits their degree, and makes distant dependencies explicit and stable.
6. Verify the prediction with tests and, when useful, an import/dependency diagram.

Do not introduce an interface only because a dependency exists. Introduce a boundary when it isolates a volatile detail, protects stable policy, or gives multiple real consumers a coherent contract.

## Dependency and cohesion heuristics

- Treat A as dependent on B when a change in B can force a change in A to preserve behavior.
- Group code that changes for the same reason.
- Split code when unrelated actors or change reasons are mixed.
- Prefer predictable modules with small, explicit interfaces.
- Keep business and presentation policy independent from React, transport, persistence, analytics, and other replaceable details.
- Let volatile details depend on stable domain types and policies, not the reverse.
- Avoid speculative extensibility. Keep an option open only when the boundary is cheap or the expected change is credible.

For a review, report each issue as:

```text
Change scenario:
Coupled modules:
Why they must change together:
Strength / locality / degree:
Smallest useful boundary:
Behavioral test:
```

## State placement

Treat the frontend as a transformation from relevant state to UI. Start by listing only the variables that can affect the requested behavior; ignore unrelated environment state.

For each variable, decide:

1. **Source of truth:** server, URL, browser storage, user input, system/environment, or a domain constant.
2. **Readers and writers:** which components need the value, and which events may change it.
3. **Lifetime and sharing:** render, component, page, navigation/session, or server-managed lifetime.
4. **Owner:** the lowest common ancestor of all readers and writers.
5. **Persistence:** use the URL when navigation, refresh, sharing, or history must preserve it; browser storage when it is local to the browser and survives reloads; server state when the server is authoritative.

Apply these rules:

- Keep local UI state in the closest component or its controller.
- Lift only the state that multiple descendants actually share to their nearest common owner.
- Pass values downward and report events upward to preserve one-way data flow.
- Use global reactive state only for genuinely cross-cutting state with many distant consumers.
- Keep remote data in the query cache; do not copy it into local or global state without a specific editing or snapshot requirement.
- Compute derived values from their source state. Store them only when they have an independent lifecycle or source of truth.
- Separate state by ownership and lifetime instead of putting all state in one store.

## Tests as state examples

Define behavior with representative state classes:

1. List the state variables relevant to the behavior.
2. Partition their values into cases that produce meaningfully different UI or effects.
3. Test one representative from each case, plus important boundaries and failures.
4. Avoid tests that vary state known to be irrelevant to the behavior.

This makes hidden dependencies visible: if a test needs extensive unrelated setup, reconsider the module boundary or dependency direction.

## Decision check

Before approving a design, answer:

- What user-visible behavior does the test protect?
- Which change becomes cheaper?
- Which dependency becomes weaker, more local, or lower-degree?
- Is the stable policy separated from replaceable details?
- Is each state variable owned by the narrowest correct scope?
- Did the design add only the boundaries required now?
