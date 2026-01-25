# Recursive Component Structure

Each component lives in its own **homonymous folder** (folder name matches component name). This structure repeats recursively at every level of the component tree.

**One component per file**: Each `.view.tsx` file must export exactly one React component. Never define multiple components in the same file.

## Folder Structure

```
component-name/
├── component-name.view.tsx      # View (presentation)
├── component-name.controller.ts # Controller (logic/state) - optional
├── component-name.spec.tsx      # Tests - optional
└── components/                  # Child components - optional
    └── child-component/         # Same recursive structure
        ├── child-component.view.tsx
        ├── child-component.controller.ts
        └── components/
            └── grandchild/
```

Only the view file is typically required. Controller, tests, and `components/` folder are optional depending on component complexity.

## Import Rules

A component can **only** import from:

1. Its own controller (`./component-name.controller.ts`)
2. Its child components (`./components/child/`)
3. Global shared components (`@/components`)

### Forbidden Imports

```
parent/
├── parent.view.tsx
└── components/
    ├── child-a/
    │   └── child-a.view.tsx  ← Cannot import from ../child-b/
    └── child-b/
        └── child-b.view.tsx  ← Cannot import from ../child-a/
```

- **No upward imports**: Never use `../` to reach parent components
- **No sibling imports**: Never import from `../sibling-component/`

## Why This Matters

Changes in any branch of the tree are **isolated**. When you modify a component:

- Parent components are unaffected (no reverse dependencies)
- Sibling components are unaffected (no cross-dependencies)
- Only child components could potentially be affected

This guarantees predictable impact scope for any change.

## Quick Reference

| Import Type | Allowed |
| --------------------- | ------- | .
| Own controller | Yes |
| Child components | Yes |
| Global `@/components` | Yes |
| Parent component | No |
| Sibling component | No |
| Cousin component | No |

## Notes

DO NOT USE BARREL EXPORTS FILES `index.ts`
