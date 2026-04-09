# Back-end style

## Controller context variables

In controllers, do not access Hono context values inline many times like `c.get('user').id`.

Always create typed variables at the top of the controller function and use those variables later.

When the type comes from a Drizzle model, do not create local type aliases like `type IUser = typeof UserModel.$inferSelect` or `type IUserToken = typeof UserTokenModel.$inferSelect`.

Use the direct model infer type in the variable annotation.

Example:

```ts
const user: typeof UserModel.$inferSelect = c.get('user');
const userToken: typeof UserTokenModel.$inferSelect = c.get('userToken');
const params: IParams = c.get('params');
const query: IQuery = c.get('query');
```

Then use `user.id`, `userToken.token`, `params.id`, `query.redirect`, etc.

Avoid patterns like:

```ts
c.get('user').id
c.get('userToken').token
c.get('params').id
c.get('query').redirect

type IUser = typeof UserModel.$inferSelect;
type IUserToken = typeof UserTokenModel.$inferSelect;
```
