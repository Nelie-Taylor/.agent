# Back-end Style

Rules specific to the Bun / Hono / Drizzle API codebase (`id-api/`).
These rules supplement `CODE-STYLE.md` which still applies.

## Controller context variables: Always read context into typed variables at the top

Applies to back-end controller functions.

Do not repeatedly access Hono context values inline such as `c.get('user').id` or `c.get('params').id` throughout the controller.

Always read the needed context values once at the top of the controller into typed variables, then use those variables later.

When the type comes from a Drizzle model, do not create local type aliases like `type IUser = typeof UserModel.$inferSelect` or `type IUserToken = typeof UserTokenModel.$inferSelect`.

Use the direct model infer type in the variable annotation.

```ts
// Good
const user: typeof UserModel.$inferSelect = c.get('user');
const userToken: typeof UserTokenModel.$inferSelect = c.get('userToken');
const params: IParams = c.get('params');
const query: IQuery = c.get('query');
const body: IBody = c.get('body');

await DBCore.update(UserModel)
  .set({
    fullName: body.fullName
  })
  .where(eq(UserModel.id, user.id));

// Bad — do not read context values inline many times
await DBCore.update(UserModel)
  .set({
    fullName: c.get('body').fullName
  })
  .where(eq(UserModel.id, c.get('user').id));

// Bad — do not create local aliases for model infer types
type IUser = typeof UserModel.$inferSelect;
type IUserToken = typeof UserTokenModel.$inferSelect;

const userBad: IUser = c.get('user');
const userTokenBad: IUserToken = c.get('userToken');
```

## Controller structure: Typed parameters with span, try/catch, and standard envelope

Every controller and middleware function must type the parameters inline as `(c: Context)` or `(c: Context, next: Next)` instead of annotating with `: MiddlewareHandler`. Include `next: Next` only when the function actually calls `next()`. Retrieve the OpenTelemetry span at the top, wrap logic in `try/catch`, annotate steps with `span.addEvent()`, and return the standard response envelope.

```ts
// Good — controller (no next needed)
const Profile = async (c: Context) => {
  const span: Span = c.get('span');
  const user: typeof UserModel.$inferSelect = c.get('user');
  try {
    span.addEvent('Find user');
    const [profile] = await DBCore.select()
      .from(UserModel)
      .where(eq(UserModel.id, user.id))
      .limit(1);
    if (!profile) {
      return c.json({
        code: 10,
        mess: 'User not exist'
      });
    }

    return c.json({
      code: 0,
      mess: 'Success',
      data: profile
    });
  } catch (e) {
    console.error(e);
    return c.json({
      code: 10,
      mess: 'error from server'
    });
  }
};

// Bad — do not use MiddlewareHandler type annotation
const Profile: MiddlewareHandler = async (c) => {
  // ...
};

// Bad — missing parameter types
const Profile = async (c) => {
  const [profile] = await DBCore.select()
    .from(UserModel)
    .where(eq(UserModel.id, c.get('user').id))
    .limit(1);
  return c.json(profile);
};
```

## Response envelope: Always use `{ code, mess, data }` shape

All JSON responses must use the standard envelope. Never return raw data or non-standard shapes from `/website` or `/link` routes.

Response code meanings:
- `0` — success
- `-1` — validation error (Joi message in `mess`)
- `-2` — authentication / captcha failure
- `1–9` — domain-specific errors (e.g. `1` = not found, `2` = already exists, `3` = not active, `4` = is primary)
- `10` — generic server error
- `11` — page not found

```ts
// Good
return c.json({
  code: 0,
  mess: 'Success',
  data: user
});

return c.json({
  code: 1,
  mess: 'Username exist'
});

return c.json({
  code: 10,
  mess: 'error from server'
});

// Bad — raw data without envelope
return c.json(user);

// Bad — non-standard key names
return c.json({
  status: 'ok',
  message: 'Success',
  result: user
});
```

## Request validation: Joi schema with early return pattern

Request validation middleware must define a Joi schema, parse the body with `.catch(() => ({}))`, validate, return early on error with `code: -1`, then set the validated value on context.

Export the TypeScript interface from the same request file so controllers can import it.

```ts
// Good
export interface ICreateBody {
  email: string;
}

const Create = async (c: Context, next: Next) => {
  const schema = Joi.object<ICreateBody>({
    email: Joi.string().email().required()
  });

  const raw = await c.req.json().catch(() => ({}));
  const validateBody = schema.validate(raw);
  if (validateBody.error) {
    return c.json({
      code: -1,
      mess: validateBody.error.message
    });
  }

  c.set('body', validateBody.value);
  await next();
};

// Bad — do not use MiddlewareHandler type annotation
const Create: MiddlewareHandler = async (c, next) => {
  // ...
};

// Bad — no early return, no interface export, no .catch on json parse
const Create = async (c: Context, next: Next) => {
  const body = await c.req.json();
  c.set('body', body);
  await next();
};
```

For path params, validate using `c.req.param()` and set with `c.set('params', ...)`.
For query strings, validate using `c.req.query()` and set with `c.set('query', ...)`.

## Export pattern: Group handlers as default object

Controllers and request validators export a default object grouping all handlers. Do not use named exports for individual handlers.

```ts
// Good
export default {
  Create,
  List,
  Destroy,
  SetPrimary
};

// Bad — individual named exports
export const Create = ...;
export const List = ...;
```

## Drizzle queries: Use chained builder pattern

Use the Drizzle builder chain directly on `DBCore` or `tx` (in transactions). Use array destructuring `[row]` for single-row results.

```ts
// Good — select single row
const [user] = await DBCore.select()
  .from(UserModel)
  .where(eq(UserModel.id, userId))
  .limit(1);

// Good — select specific columns
const [user] = await DBCore.select({
    id: UserModel.id,
    username: UserModel.username
  })
  .from(UserModel)
  .where(eq(UserModel.id, userId))
  .limit(1);

// Good — insert with returning
const [created] = await DBCore.insert(UserEmailModel)
  .values({
    userId: user.id,
    email: body.email,
    domain: body.email.replace(/.*@/, '')
  })
  .returning({
    id: UserEmailModel.id
  });

// Good — update
await DBCore.update(UserModel)
  .set({
    firstName: body.firstName,
    updatedAt: new Date()
  })
  .where(eq(UserModel.id, user.id));

// Good — delete
await DBCore.delete(UserEmailModel)
  .where(eq(UserEmailModel.id, emailId));

// Good — transaction
await DBCore.transaction(async (tx) => {
  const [user] = await tx.insert(UserModel)
    .values({...})
    .returning({
      id: UserModel.id
    });
  await tx.insert(UserEmailModel)
    .values({
      userId: user.id
    });
});

// Good — parallel independent writes
await Promise.all([
  DBCore.update(UserEmailModel)
    .set({
      activatedAt: new Date()
    })
    .where(eq(UserEmailModel.id, emailId)),
  DBCore.update(UserCodeModel)
    .set({
      activatedAt: new Date()
    })
    .where(eq(UserCodeModel.id, codeId))
]);
```

## Route composition: Request → Security → Controller order

Routes are composed as positional middleware arguments. Always follow the order: request validation, then security middleware (Turnstile / Auth / Password), then controller.

```ts
// Good
router.post('/login', AuthRequest.Login, TurnstileMiddleware, AuthController.Login);
router.delete('/email/:id', EmailRequest.Destroy, AuthMiddleware, PasswordMiddleware, EmailController.Destroy);

// Bad — controller before auth
router.post('/login', AuthController.Login, TurnstileMiddleware, AuthRequest.Login);
```

## Import aliases: Always use path aliases

Use the configured `tsconfig.json` aliases. Never use relative paths when an alias exists.

Available aliases: `@config/*`, `@controller/*`, `@middleware/*`, `@model/*`, `@request/*`, `@router/*`, `@utils/*`.

```ts
// Good
import DBCore from '@config/db-core';
import { UserModel } from '@model/user';
import AuthMiddleware from '@middleware/auth';

// Bad — relative path when alias exists
import DBCore from '../../config/db-core';
import { UserModel } from '../../model/user';
```

## Naming conventions

- Controller / middleware functions: `PascalCase` — `Profile`, `SendCode`, `SetPrimary`
- Database models: `PascalCase` + `Model` suffix — `UserEmailModel`, `ConnectCodeModel`
- Context keys: `camelCase` strings — `'user'`, `'userToken'`, `'body'`, `'params'`, `'query'`, `'span'`
- File names: `kebab-case` — `forgot-password.ts`, `upload-avatar.ts` (follows CODE-STYLE.md)
- Interfaces for request bodies/params/queries: `I` prefix + `PascalCase` — `ICreateBody`, `IUpdateParams`, `ISendCodeBody`
