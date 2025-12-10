# Authorization Guidelines

## Authentication vs Authorization

- **Authentication** (`auth` module): "Who is this user?" - Verifies JWT tokens
- **Authorization** (`authorization` module): "Can this user do this?" - Checks resource access

## Service-Level Authorization

**Services are responsible for low-level authorization checks.**

Controllers check permissions at the endpoint level (can user call this endpoint?), but services must check resource access (can user access this specific record?).

Example: "Can Pablo edit this company?"

```typescript
@Injectable()
export class UpdateCompanyService {
  constructor(private readonly authorizationService: AuthorizationService) {}

  async execute(params: { companyId: string; user: AuthenticatedUser }) {
    const { companyId, user } = params;

    // Service checks: Can this user access this specific company?
    if (!this.authorizationService.canAccessCompany(user, companyId)) {
      throw new ForbiddenException("No tienes acceso a esta empresa");
    }

    // Business logic...
  }
}
```

The `AuthorizationService` provides utility functions. Each service calls these checks when needed.

## Checklist for New Endpoints

- [ ] Service checks auth for the user , the action and the resource
