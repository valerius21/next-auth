The NextAuth.js client library makes it easy to interact with sessions from React applications.

## The Session Object

```ts
{
  user: {
    name: string
    email: string
    image: string
  },
  expires: Date // This is the expiry of the session, not any of the tokens within the session
}
```

:::tip
The session data returned to the client does not contain sensitive information such as the Session Token or OAuth tokens. It contains a minimal payload that includes enough data needed to display information on a page about the user who is signed in for presentation purposes (e.g name, email, image).

You can use the [session callback](/configuration/callbacks#session-callback) to customize the session object returned to the client if you need to return additional data in the session object.
:::

:::note
The `expires` value is rotated, meaning whenever the session is retrieved from the [REST API](/getting-started/rest-api), this value will be updated as well, to avoid session expiry.
:::
