## `signIn()`

- Client Side: **Yes**
- Server Side: No

Using the `signIn()` method ensures the user ends back on the page they started on after completing a sign in flow. It will also handle CSRF Tokens for you automatically when signing in with email.

The `signIn()` method can be called from the client in different ways, as shown below.

### Redirect to sign in page

```js
import { signIn } from "next-auth/react"

export default () => <button onClick={() => signIn()}>Sign in</button>
```

### Start OAuth sign-in flow

By default, when calling the `signIn()` method with no arguments, you will be redirected to the NextAuth.js sign-in page. If you want to skip that and get redirected to your provider's page immediately, call the `signIn()` method with the provider's `id`.

For example to sign in with Google:

```js
import { signIn } from "next-auth/react"

export default () => (
  <button onClick={() => signIn("google")}>Sign in with Google</button>
)
```

### Start Email sign-in flow

When using it with the email flow, pass the target `email` as an option.

```js
import { signIn } from "next-auth/react"

export default ({ email }) => (
  <button onClick={() => signIn("email", { email })}>Sign in with Email</button>
)
```

### Specifying a callback URL

The `callbackUrl` specifies to which URL the user will be redirected after signing in. It defaults to the current URL of a user.

You can specify a different `callbackUrl` by specifying it as the second argument of `signIn()`. This works for all providers.

e.g.

- `signIn(undefined, { callbackUrl: '/foo' })`
- `signIn('google', { callbackUrl: 'http://localhost:3000/bar' })`
- `signIn('email', { email, callbackUrl: 'http://localhost:3000/foo' })`

The URL must be considered valid by the [redirect callback handler](/configuration/callbacks#redirect-callback). By default it requires the URL to be an absolute URL at the same host name, or a relative url starting with a slash. If it does not match it will redirect to the homepage. You can define your own [redirect callback](/configuration/callbacks#redirect-callback) to allow other URLs.

### Preventing default redirect

:::note
The redirect option is only available for `credentials` and `email` providers.
:::

In some cases, you might want to deal with the sign in response on the same page and disable the default redirection. For example, if an error occurs (like wrong credentials given by the user), you might want to handle the error on the same page. For that, you can pass `redirect: false` in the second parameter object.

e.g.

- `signIn('credentials', { redirect: false, password: 'password' })`
- `signIn('email', { redirect: false, email: 'bill@fillmurray.com' })`

`signIn` will then return a Promise, that resolves to the following:

```ts
{
  /**
   * Will be different error codes,
   * depending on the type of error.
   */
  error: string | undefined
  /**
   * HTTP status code,
   * hints the kind of error that happened.
   */
  status: number
  /**
   * `true` if the signin was successful
   */
  ok: boolean
  /**
   * `null` if there was an error,
   * otherwise the url the user
   * should have been redirected to.
   */
  url: string | null
}
```

### Additional parameters

It is also possible to pass additional parameters to the `/authorize` endpoint through the third argument of `signIn()`.

See the [Authorization Request OIDC spec](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) for some ideas. (These are not the only possible ones, all parameters will be forwarded)

e.g.

- `signIn("identity-server4", null, { prompt: "login" })` _always ask the user to re-authenticate_
- `signIn("auth0", null, { login_hint: "info@example.com" })` _hints the e-mail address to the provider_

:::note
You can also set these parameters through [`provider.authorizationParams`](/configuration/providers/oauth#options).
:::

:::note
The following parameters are always overridden server-side: `redirect_uri`, `state`
:::

---

## `signOut()`

- Client Side: **Yes**
- Server Side: No

In order to logout, use the `signOut()` method to ensure the user ends back on the page they started on after completing the sign out flow. It also handles CSRF tokens for you automatically.

It reloads the page in the browser when complete.

```js
import { signOut } from "next-auth/react"

export default () => <button onClick={() => signOut()}>Sign out</button>
```

### Specifying a callback URL

As with the `signIn()` function, you can specify a `callbackUrl` parameter by passing it as an option.

e.g. `signOut({ callbackUrl: 'http://localhost:3000/foo' })`

The URL must be considered valid by the [redirect callback handler](/configuration/callbacks#redirect-callback). By default, it requires the URL to be an absolute URL at the same host name, or you can also supply a relative URL starting with a slash. If it does not match it will redirect to the homepage. You can define your own [redirect callback](/configuration/callbacks#redirect-callback) to allow other URLs.

### Preventing default redirect

If you pass `redirect: false` to `signOut`, the page will not reload. The session will be deleted, and the `useSession` hook is notified, so any indication about the user will be shown as logged out automatically. It can give a very nice experience for the user.

:::tip
If you need to redirect to another page but you want to avoid a page reload, you can try:
`const data = await signOut({redirect: false, callbackUrl: "/foo"})`
where `data.url` is the validated URL you can redirect the user to without any flicker by using Next.js's `useRouter().push(data.url)`
:::

---

## `getSession()`

- Client Side: **Yes**
- Server Side: **Yes**

NextAuth.js provides a `getSession()` method which can be called client or server side to return a session.

It calls `/api/auth/session` and returns a promise with a session object, or null if no session exists.

#### Client Side Example

```js
async function myFunction() {
  const session = await getSession()
  /* ... */
}
```

#### Server Side Example

```js
import { getSession } from "next-auth/react"

export default async (req, res) => {
  const session = await getSession({ req })
  /* ... */
  res.end()
}
```

:::note
When calling `getSession()` server side, you need to pass `{req}` or `context` object.
:::

The tutorial [securing pages and API routes](/tutorials/securing-pages-and-api-routes) shows how to use `getSession()` in server side calls.

---

## `getCsrfToken()`

- Client Side: **Yes**
- Server Side: **Yes**

The `getCsrfToken()` method returns the current Cross Site Request Forgery Token (CSRF Token) required to make POST requests (e.g. for signing in and signing out).

You likely only need to use this if you are not using the built-in `signIn()` and `signOut()` methods.

#### Client Side Example

```js
async function myFunction() {
  const csrfToken = await getCsrfToken()
  /* ... */
}
```

#### Server Side Example

```js
import { getCsrfToken } from "next-auth/react"

export default async (req, res) => {
  const csrfToken = await getCsrfToken({ req })
  /* ... */
  res.end()
}
```

---

## `getProviders()`

- Client Side: **Yes**
- Server Side: **Yes**

The `getProviders()` method returns the list of providers currently configured for sign in.

It calls `/api/auth/providers` and returns a list of the currently configured authentication providers.

It can be useful if you are creating a dynamic custom sign in page.

---

#### API Route

```jsx title="pages/api/example.js"
import { getProviders } from "next-auth/react"

export default async (req, res) => {
  const providers = await getProviders()
  console.log("Providers", providers)
  res.end()
}
```

:::note
Unlike `getSession()` and `getCsrfToken()`, when calling `getProviders()` server side, you don't need to pass anything, just as calling it client side.
:::
