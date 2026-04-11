# Code Style

## Objects: Always put properties on separate lines

Applies to TypeScript/JavaScript and Dart/Flutter source.

This applies everywhere: top-level declarations, inline payloads, object/map literals inside functions, callbacks, widget trees, helper returns, and nested object literals passed as arguments.

```ts
// Good
const obj = {
  key1: 'value1',
  key2: 'value2',
  key3: 'value3'
}

timestamp('created_at', {
  withTimezone: true
})

// Bad — do not inline properties
const obj = { key1: 'value1', key2: 'value2', key3: 'value3' }

timestamp('created_at', { withTimezone: true })
```

```dart
// Good
final payload = {
  'name': name,
  'phone': phone,
  'email': email
};

void submit() {
  api.post(
    '/profile',
    data: {
      'name': name,
      'phone': phone
    },
  );
}

// Bad — do not inline properties
final payload = {'name': name, 'phone': phone, 'email': email};

void submit() {
  api.post('/profile', data: {'name': name, 'phone': phone});
}
```

## Never use destructuring assignment

```ts
// Good
const param = c.get('param') as IParamId;
const body = await c.req.json<{ name: string }>();
console.log(param.id, body.name);

// Bad — do not destructure
const { id } = c.get('param') as IParamId;
const { name } = await c.req.json<{ name: string }>();
```

## Never break import specifiers onto multiple lines

```ts
// Good
import type { ICreateBody, ISearchTenantQuery, IUploadTenantImageParams, IUploadVehicleMediaParams } from '@request/mobile/owner/contract';

// Bad — do not wrap import items onto multiple lines
import type {
  ICreateBody,
  ISearchTenantQuery,
  IUploadTenantImageParams,
  IUploadVehicleMediaParams
} from '@request/mobile/owner/contract';
```

## Always use single quotes

```ts
// Good
import { IUpdateParams, IUpdateBody } from '@request/web/system/connect';
const mess = 'error from server';

// Bad — do not use double quotes when single quotes are possible
import { IUpdateParams, IUpdateBody } from "@request/web/system/connect";
const mess = "error from server";
```

## Never use trailing commas in objects or arrays

Applies to TypeScript/JavaScript source.

Exception: Do not enforce this rule for Dart/Flutter code because `dart format` may add trailing commas automatically for multiline layouts.

```ts
// Good
const obj = {
  key1: 'value1',
  key2: 'value2'
}

const arr = [
  'a',
  'b'
]

// Bad — do not leave a trailing comma on the last item
const obj = {
  key1: 'value1',
  key2: 'value2',
}

const arr = [
  'a',
  'b',
]
```

## Controller params/body: Always use interfaces from request middleware

```ts
// Good — import and use the interface from @request/
import { IUpdateParams, IUpdateBody } from "@request/web/system/connect";

const params: IUpdateParams = c.get('params');
const body: IUpdateBody = c.get('body');

// Bad — do not inline types
const params: { id: string } = c.get('params');
const body: { name: string } = c.get('body');
```

## Never break function arguments onto multiple lines

```ts
// Good
StorageHDG.DeleteMany(media.map((m) => `${mediaPath(user.id, params.id)}/${m.fileName}`)).catch(console.error);

// Bad — do not wrap arguments across lines
StorageHDG.DeleteMany(
  media.map((m) => `${mediaPath(user.id, params.id)}/${m.fileName}`)
).catch(console.error);
```

## Only break chained calls from the second dot onward

Keep the first method call on the same line as the base expression. If a chain must wrap, only the second `.` and onward may start on new lines.

```ts
// Good
await DBCore.select()
  .from(Account.AccountOAuth)
  .where(eq(Account.AccountOAuth.userId, userId));

// Bad — do not break before the first chained call
await DBCore
  .select()
  .from(Account.AccountOAuth)
  .where(eq(Account.AccountOAuth.userId, userId));
```

## Never extract object properties into intermediate variables

Applies to TypeScript/JavaScript and Dart/Flutter source.

This also includes assigning object members to new variables before use. Do not write `const ConnectCodeModel = ID.Connect.ConnectCodeModel;` or similar. Use `ID.Connect.ConnectCodeModel` inline.

```ts
// Good — use property access inline
eq(Model.id, params.id)
StorageHDG.Upload(mediaPath(user.id, params.id), ...)

// Bad — do not extract into a new variable
const vehicleId = params.id;
eq(Model.id, vehicleId)
StorageHDG.Upload(mediaPath(user.id, vehicleId), ...)
```

```dart
// Good — use property access inline
Text(user.fullName)
service.updateProfile(user.id, form.name)

// Bad — do not extract into a new variable when it is only used immediately
final fullName = user.fullName;
final userId = user.id;
Text(fullName)
service.updateProfile(userId, form.name)
```

## Extract function calls used more than once into a variable

When a function call (not a simple property access) is used more than once, store its result in a variable. This avoids duplicate calls and improves readability.

This rule is about **function/method calls** like `c.req.header('app')`, `Date.now()`, `arr.filter(...)`. It does NOT apply to simple property access like `user.id` or `params.name` — those follow the "Never extract object properties" rule above.

```ts
// Good — function called twice, extract to variable
const app = c.req.header('app');
if (app && app !== '') {
  const [connect] = await DBCore.select()
    .from(ConnectModel)
    .where(eq(ConnectModel.secret, app));
}

// Bad — same function call repeated inline
if (c.req.header('app') && c.req.header('app') !== '') {
  const [connect] = await DBCore.select()
    .from(ConnectModel)
    .where(eq(ConnectModel.secret, c.req.header('app')));
}
```

## If statements: Always use braces

```ts
// Good
if (condition) {
  return value;
}

// Bad — do not omit braces
if (condition) return value;
```

## Never break short ternary expressions onto multiple lines

```ts
// Good
const vehicleMedia = vehicles.length > 0 ? await DBCore.select()
  .from(Account.AccountTenantVehicleMediaModel) : [];

// Bad — do not wrap short ternary across lines
const vehicleMedia = vehicles.length > 0
  ? await DBCore.select().from(Account.AccountTenantVehicleMediaModel)
  : [];
```

## Arrays: Always put items on separate lines

Each item in `[]` must be on its own line — primitives, expressions, and objects alike.

```ts
// Good
const ids = [
  'abc',
  'def',
  'ghi'
]

const names = [
  getFirst(),
  getSecond()
]

// Bad — do not inline items
const ids = ['abc', 'def', 'ghi']
const names = [getFirst(), getSecond()]
```

## Array of objects: close `}, {` on the same line

When an array contains object literals, each object's opening `{` stays on the line with `[` or `}, {`, and the final `}]` closes together.

```ts
// Good
const users = [{
  name: 'Alice',
  age: 30
}, {
  name: 'Bob',
  age: 25
}]

await db.insert(TokenModel).values([{
  userId: user.id,
  token: code
}, {
  userId: user.id,
  token: backup
}])

// Bad — do not put `{` or `}` on their own line inside an array
const users = [
  {
    name: 'Alice',
    age: 30
  },
  {
    name: 'Bob',
    age: 25
  }
]

// Bad — do not separate `}` and `]` onto two lines
const users = [{
  name: 'Alice',
  age: 30
}, {
  name: 'Bob',
  age: 25
}
]
```

## File names: Always use lowercase with hyphens

All file names must be lowercase and use hyphens to separate words.

```bash
# Good
notification-service.ts
user-profile-card.tsx
home-screen.dart

# Bad — do not use camelCase, PascalCase, or underscores
NotificationService.ts
user_profile_card.tsx
HomeScreen.dart
```
