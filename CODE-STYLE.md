# Code Style

## Objects: Always put properties on separate lines

```ts
// Good
const obj = {
  key1: 'value1',
  key2: 'value2',
  key3: 'value3',
}

// Bad — do not inline properties
const obj = { key1: 'value1', key2: 'value2', key3: 'value3' }
```

## JSX: Keep all attributes on a single line

```tsx
// Good
<div className='flex h-screen w-full items-center justify-center overflow-hidden bg-content1 p-2 sm:p-4 lg:p-8'>

// Bad — do not break attributes onto multiple lines
<div
  className='flex h-screen w-full items-center justify-center overflow-hidden bg-content1 p-2 sm:p-4 lg:p-8'
>
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

## Never extract object properties into intermediate variables

```ts
// Good — use property access inline
eq(Model.id, params.id)
StorageHDG.Upload(mediaPath(user.id, params.id), ...)

// Bad — do not extract into a new variable
const vehicleId = params.id;
eq(Model.id, vehicleId)
StorageHDG.Upload(mediaPath(user.id, vehicleId), ...)
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
