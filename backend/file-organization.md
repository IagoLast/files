# Backend File Organization

## Co-locate by feature, not by kind

Group files by the **operation** they belong to, not by their technical role. Each atomic operation (see [Services](./services.md)) keeps its controller, service, DTOs, repository, schema and tests **together** in the same folder.

We do **not** want top-level `services/`, `controllers/`, `repositories/`, `dto/` buckets that mix unrelated features together. Those buckets force you to jump across four folders to read one feature, and they hide the real shape of the module.

### Don't (organized by kind)

```
modules/onboarding/
  controllers/
    onboarding.controller.ts
  services/
    get-tutorial-first-steps.service.ts
    update-tutorial-first-steps.service.ts
    get-post-registration-onboarding.service.ts
    update-post-registration-onboarding.service.ts
  repositories/
    tutorial-first-steps.repository.ts
    tutorial-first-steps-progress.repository.ts
    post-registration-onboarding.repository.ts
  dto/
    update-post-registration-onboarding.dto.ts
    post-registration-onboarding-state-response.dto.ts
    tutorial-first-steps-response.dto.ts
  db/schemas/
    tutorial-first-steps-progress.schema.ts
```

Problems:
- To understand `get-tutorial-first-steps` you open 4 folders.
- The `services/` folder mixes unrelated features (tutorial + post-registration).
- DTOs drift from the service that owns them.
- Deleting a feature means hunting files across the module.

### Do (co-located by feature)

```
modules/onboarding/
  onboarding.module.ts
  tutorial-first-steps/
    tutorial-first-steps.controller.ts
    tutorial-first-steps.schema.ts
    tutorial-first-steps.repository.ts
    get-tutorial-first-steps/
      get-tutorial-first-steps.service.ts
      get-tutorial-first-steps.response.dto.ts
      get-tutorial-first-steps.service.spec.ts
    update-tutorial-first-steps/
      update-tutorial-first-steps.service.ts
      update-tutorial-first-steps.dto.ts
      update-tutorial-first-steps.service.spec.ts
  post-registration-onboarding/
    post-registration-onboarding.controller.ts
    post-registration-onboarding.schema.ts
    post-registration-onboarding.repository.ts
    get-post-registration-onboarding/
      get-post-registration-onboarding.service.ts
      get-post-registration-onboarding.response.dto.ts
    update-post-registration-onboarding/
      update-post-registration-onboarding.service.ts
      update-post-registration-onboarding.dto.ts
```

Wins:
- Everything an operation needs lives next to it: service, DTOs, tests.
- Shared-per-feature files (controller, schema, repository) live at the feature root.
- Deleting an operation is one folder.
- The folder tree mirrors the public API surface.

## Rules

1. **One folder per operation.** The folder name matches the service: `create-lead/`, `update-lead/`, `archive-lead/`.
2. **DTOs live with the service that uses them.** Request DTOs in the operation that consumes them; response DTOs in the operation that produces them. Do not create a shared `dto/` bucket.
3. **Tests sit next to the code they test.** `*.spec.ts` and `*.e2e-spec.ts` next to the file, not in a parallel `__tests__/` tree.
4. **Promote only when shared.** If a DTO, type, or helper is used by two or more operations, lift it to the nearest common feature folder — never higher than needed.
5. **Controllers and repositories are feature-scoped**, not module-scoped. One controller per feature, grouping its operations' routes. One repository per Mongo collection, owned by the feature it models.
6. **Schemas live with their feature**, named `*.schema.ts`. No global `db/schemas/` folder.
7. **No `services/` or `dto/` folders.** If you see one, that is a refactor signal.

## When in doubt

Ask: *"if I delete this feature tomorrow, can I `rm -rf` one folder?"* If the answer is no, the files are in the wrong place.
