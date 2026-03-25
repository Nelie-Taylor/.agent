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

## If statements: Always use braces

```ts
// Good
if (condition) {
  return value;
}

// Bad — do not omit braces
if (condition) return value;
```
