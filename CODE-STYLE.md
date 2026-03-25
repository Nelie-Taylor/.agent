# CODE STYLE

## Object: Item luôn xuống hàng

```ts
const obj = {
  key1: 'value1',
  key2: 'value2',
  key3: 'value3',
}
```

## JSX Tag: Attributes luôn trên 1 hàng

```tsx
// ✅ Đúng
<div className='flex h-screen w-full items-center justify-center overflow-hidden bg-content1 p-2 sm:p-4 lg:p-8'>

// ❌ Sai - không xuống hàng attributes
<div
  className='flex h-screen w-full items-center justify-center overflow-hidden bg-content1 p-2 sm:p-4 lg:p-8'
>
```

## If: Luôn có `{}`

```ts
// ✅ Đúng
if (condition) {
  return value;
}

// ❌ Sai - không được bỏ {}
if (condition) return value;
```
