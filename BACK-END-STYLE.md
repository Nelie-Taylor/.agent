# Back-end Style

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
