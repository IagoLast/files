# Backend Services Guidelines

## Atomic Services Pattern

Use **one service per operation** instead of generic services with multiple methods. This makes the code easier to maintain and test.

### Structure

```
src/modules/auth/
  └── services/
      ├── login.service.ts
      ├── logout.service.ts
      └── refresh-token.service.ts
```

### Implementation

Each service exposes a single `execute` method that receives one object with all required parameters:

```typescript
interface LoginParams {
  email: string;
  password: string;
}

interface LoginResult {
  token: string;
}

@Injectable()
export class LoginService {
  async execute(params: LoginParams): Promise<LoginResult> {
    // implementation
  }
}
```

### Usage in Controllers

```typescript
@Controller("auth")
export class AuthController {
  constructor(private readonly loginService: LoginService) {}

  @Post("login")
  async login(@Body() dto: LoginDto) {
    return this.loginService.execute(dto);
  }
}
```