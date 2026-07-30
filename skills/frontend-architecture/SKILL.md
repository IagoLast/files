---
name: frontend-architecture
description: Set up, design, implement, refactor, or review client-rendered React SPA architecture using React Router, route-aligned pages, recursive components, controllers, TanStack Query hooks, mutations, repositories, services, and a singleton API client. Use setup mode once when establishing project-wide structure and dependency enforcement. By default, use feature/review mode when adding or reviewing a feature, deciding file or state placement, wiring router-to-API data flow, splitting components, measuring change impact, or checking dependency boundaries and naming. Warn before applying it to projects that are not client-rendered React SPAs using React Router.
---

# Frontend Architecture

Build the smallest feature that preserves the dependency direction and the repository's existing conventions.

## Compatibility warning

This architecture is intentionally designed for client-rendered React SPAs using React Router. Inspect the rendering and routing model before applying it. If the project uses a different model, warn the user that the route and page conventions may not fit, explain the mismatch briefly, and follow the repository's conventions instead of imposing this architecture.

## Choose a mode

- Use **default mode** for feature implementation, refactoring, and architecture review. Assume the project setup is intentional, inspect it, and follow it unless the task is explicitly to change it.
- Use **setup mode** only when the user asks to establish the architecture or the repository lacks the required project-wide conventions and enforcement.
- Do not run setup merely because an individual feature is missing a file or violates a rule. Fix that feature through default mode.

## Default mode: feature work and review

1. Inspect the existing router, aliases, query library, HTTP client, and nearby feature before creating files.
2. Define the user-visible behavior with a colocated test.
3. Identify the relevant state variables and the concrete change the design must absorb.
4. Trace the feature through `router → page → controller → query or mutation → repository → API client`.
5. Add only the layers the behavior needs. Do not create empty controllers, services, or folders.
6. Keep views presentational and move orchestration, navigation, state, and async coordination into controllers.
7. Run the relevant test, typecheck, lint, and build after each code change.
8. Review imports and delete accidental abstractions or cross-feature coupling.

For a read-only review, do not modify code. Report concrete dependency or state problems, their likely change impact, and the smallest useful correction.

Read [references/architecture.md](references/architecture.md) for the dependency map, file responsibilities, import boundaries, and review checklist. Read [references/decision-method.md](references/decision-method.md) when choosing between designs, placing state, or explaining a review finding.

## Setup mode: establish project conventions

Read [references/setup.md](references/setup.md) completely before changing project-wide structure, aliases, lint rules, shared clients, providers, or templates. Use [references/architecture.md](references/architecture.md) as the target architecture.

Finish setup with one working vertical slice, automated boundary enforcement, and passing tests, typecheck, lint, and build. Then return to default mode for all feature work.

## Non-negotiable rules

- Use one file per component: every `*.view.tsx` exports exactly one React component.
- Give every page or component a homonymous dash-case folder.
- Put every child component or subpage in the owner's `components/` folder. Give that child its own homonymous folder and repeat the same structure recursively at every depth.
- Allow a component to import only its own controller, its descendants, and global shared components.
- Do not import a parent, sibling, or cousin component. Do not use barrel `index.ts` files.
- Keep one concern per file and use descriptive suffixes: `.view.tsx`, `.controller.ts`, `.route.tsx`, `.query.ts`, `.mutation.ts`, `.repository.ts`, `.service.ts`, and `.spec.tsx`.
- Use route constants. Keep each route object beside its page and let the root router assemble route objects.
- Keep the API client a singleton concerned only with transport, configuration, and interceptors.
- Put endpoint calls and server-to-domain mapping in repositories; never call the API client from a view.
- Put server reads in query hooks and writes in mutation hooks. Mutation success must update or invalidate the affected cache deliberately.
- Group query and mutation hooks by resource under `src/queries/<resource>/`.
- Name hooks `use<Action><Resource>Query` or `use<Action><Resource>Mutation`.
- Keep domain services pure unless their explicit responsibility is a browser boundary such as token storage.
- Keep state at the lowest common owner that needs to read or change it. Derive values instead of storing redundant state.
- Prefer `@/` aliases for cross-root imports and local `./` imports within a feature. Try to avoid climbing the component tree with `../`.

## Choose file placement

- Route entry point or layout: `src/pages/<page>/`
- Page-owned component or nested subpage: `src/pages/<page>/components/<child>/`
- Reused, domain-neutral UI: `src/components/<component>/`
- HTTP transport: `src/client/api-client.ts`
- Resource data access and mapping: `src/repositories/<resource>.repository.ts`
- Cached read: `src/queries/<resource>/<action>-<resource>.query.ts`
- Server write: `src/queries/<resource>/<action>-<resource>.mutation.ts`
- Pure business rule: `src/services/<concept>.service.ts`
- Shared domain shape: `src/types/`

Promote code to a shared location only after two real consumers need it.
