# Backend Services

Services are NestJS providers (`@Injectable()` classes) that encapsulate a single business operation. Every service exposes one public method, `execute`, which receives a single object argument with all parameters. Before doing any work, `execute` MUST call the private `_checkPermissions()` method.

This shape is mandatory. It exists so agents (and humans) can reason about a service without reading its implementation: same entry point, same contract, same guarantee that authorization runs first.

## One service per operation

One file, one exported service, one domain action. Filename: `foo-bar.service.ts`.

```
services/
  create-user.service.ts
  charge-subscription.service.ts
  send-welcome-email.service.ts
```

Not this:

```
services/
  user.service.ts          // 2000 lines, 30 methods, untouchable
```

### Why

The main reason is **AI context**. An agent asked to touch `chargeSubscription` should be able to load `charge-subscription.service.ts` and have the full picture in a few hundred tokens. A god-service like `user.service.ts` forces the agent to read thousands of lines of unrelated logic, blowing the context window and degrading reasoning quality on every task that touches the file.

The same property helps humans:

- **Search = find.** `grep charge-subscription` → one file.
- **Clean diffs.** Changing billing touches one file, not a shared monster.
- **Honest imports.** Each file imports only what it needs.
- **Trivial tests.** Mock the deps of one action, not of thirty.
- **Cheap refactors.** Move = `git mv`. Rename = rename file + function.
- **Built-in ceiling.** Past ~150 lines, the action hides sub-actions → split. God-services have no such signal.

### Trade-off

More files. Fine — "many files" is linear friction solved by fuzzy find. "One service does everything" is exponential friction solved only by rewriting (and unloadable into any context window, human or AI).

## The contract

1. One public method: `execute({ ... })`.
2. One argument: an object holding every parameter (named arguments — see [naming-conventions.md](../naming-conventions.md#use-an-object-if-your-function-takes-more-than-2-parameters)).
3. The first statement inside `execute` is `await this._checkPermissions(args)`.
4. `_checkPermissions` throws (`ForbiddenException` / `UnauthorizedException`) when the caller is not allowed. It never returns `false`.
5. No other public methods. Helpers are `private` and prefixed with `_` (`_verbNoun`).
6. Dependencies (repositories, other services) come in via the constructor.

```typescript
// do this — create-user.service.ts
interface ICreateUserExecuteArgs {
  actor: IActor;
  email: string;
  role: IUserRole;
}

@Injectable()
export class CreateUserService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async execute(args: ICreateUserExecuteArgs): Promise<IUser> {
    await this._checkPermissions(args);
    const user = await this.usersRepository.create({ email: args.email, role: args.role });
    return user;
  }

  private async _checkPermissions({ actor, role }: ICreateUserExecuteArgs): Promise<void> {
    if (!actor.canManageUsers) {
      throw new ForbiddenException('cannot create users');
    }
    if (role === 'admin' && !actor.isAdmin) {
      throw new ForbiddenException('only admins can grant admin');
    }
  }
}
```

The `_checkPermissions` line is the whole point: it's a visible reminder that authorization is the service's responsibility, not the route's.

## Naming

- File: `<action>-<resource>.service.ts` (e.g. `create-user.service.ts`, `archive-project.service.ts`).
- One service = one verb. If you need two actions on the same resource, write two services.
- Args type: `I<ServiceName>ExecuteArgs`.
- Return type: `I<ServiceName>ExecuteReturnValue` when not a domain entity.

## Anti-patterns

- **No `_checkPermissions` call.** If the action is truly public, the method is still required — it just resolves to `void`. The presence of the call is the signal.
- **Permission checks in controllers/routes only.** Routes can do coarse checks, but the authoritative check lives in the service. Services get called from queues, crons and other services too.
- **Multiple public methods.** Split into separate services.
- **Positional arguments.** Always one object.
- **Returning permission errors instead of throwing.** Throw — callers should not have to inspect a result envelope.
- **God-services.** `user.service.ts` with thirty methods. Split into one file per action.

## Why this shape

- Agents have a single, predictable entry point to call and to mock.
- Putting `_checkPermissions` as the first line of `execute` makes authorization a visible, reviewable rule.
- Named arguments make call sites self-documenting and reorder-safe.
- One service per action keeps blast radius small and the file loadable into any context window.
