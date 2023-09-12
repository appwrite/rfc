# Server Side Rendering

- Implementation Owner: @loks0n
- Start Date: 31-08-2023
- Target Date: Unknown

## Summary

[summary]: #summary

Appwrite Auth is currently tailored towards client side rendered (CSR) applications. This RFC proposes to add support for server side rendered (SSR) applications.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Many popular web frameworks, such as Next.js, Nuxt.jsm and SvelteKit, support and recommend SSR, by default. When developers attempt to use Appwrite Auth with these frameworks, they run into issues with the current implementation.

SSR is popular for a number of reasons:

- It enables search engines to crawl the site and index the content
- It improves performance by reducing the time to first render, and reducing the amount of JavaScript required to render the page
- It improves accessibility by ensuring that the page is usable even if JavaScript is disabled
- It prevents flash of unauthenticated content.

**Problem #1: Accessing basic sessions**

Basic sessions, such as anonymous sessions, email-password sessions and magic URL sessions, are not accessible from SSR applications using the SDK.
When a client uses the SDK to make a POST request to Appwrite, the SDK obscures the session cookie value with the [call method](https://github.com/appwrite/sdk-for-node/blob/aaea14a36d5b7daac859eaa8dc44d2253fbcbcef/lib/client.js#L120C86-L120C86) and only returns the response body.

A popular hacky workaround for this involves making a manual fetch request, bypassing the SDK and parsing the cookie from the response headers.

```js
import { parse } from "set-cookie-parser";

const response = await fetch(
  `https://cloud.appwrite.io/v1/account/sessions/anonymous`,
  {
    method: "POST",
    headers: {
      "x-appwrite-project": "PROJECT_ID",
    },
  }
);

const cookies = response.headers.get("set-cookie") || "";
const sessionCookie = parse(cookies).find((cookie) =>
  cookie.name.startsWith("a_session")
);
const sessionToken = sessionCookie?.value;
```

> SDK for Flutter has a similiar problem, but bypasses it by adding an 'interceptor' to requests. With any response containing a `set-cookie` header, the interceptor stores the cookies within a Flutter implementation of cookie storage. Before making any request, the session cookie is retrieved from storage and added to the request headers.

**Problem #2: Accessing OAuth2 sessions**

Here's a visualisation of the current OAuth2 flow:

![CSR OAuth2 Flow Sequence Diagram](csr-oauth-flow.png)

Step by step:

1. Client makes a 'Create OAuth2 Session' request to the Appwrite,containing the provider, and a success authentication redirect URL.
2. Appwrite returns a URL for a page to authenticate with the OAuth2 provider.
3. User is redirected to the authentication URL, and authenticates with the OAuth2 provider.
4. OAuth2 provider redirects the browser back to Appwrite.
5. Appwrite sets the session cookie on the Appwrite domain, and redirects the browser to the success URL.

This is incompatible with SSR applications, because the session cookie is set on the Appwrite domain, and not the SSR domain.

> There is an undocumented workaround for SSR. To use it, when creating an OAuth2 session, set success parameter is set to `{SSR_DOMAIN}/auth/oauth2/success`. 
> Now, Appwrite will append the session secret as a query parameter when redirecting to this URL. You can find the source code for this [here](https://github.com/appwrite/appwrite/blob/3f3d518f3664bcab281ee00b45dd2f2d387ffc72/app/controllers/api/account.php#L870).
> Although this workaround has good developer experience, it is not secure. The session secret is exposed in the URL, and can be intercepted.

**Problem #3: Using session tokens with the SDK**

After acquiring a session token, you want to make authenticated requests. For example, getting the user's account details. Currently, to set a session token on the server-side, we have to set the `X-Fallback-Cookies` header.

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

const serialisedCookies = JSON.stringify({
  a_session_PROJECT_ID: sessionToken,
});

client.headers["X-Fallback-Cookies"] = serialisedCookies;

const account = new Account(client);
const currentUser = await account.get();
```

This is not intuitive, and requires the developer to understanding details of Appwrite that don't need to be exposed.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

**Problem #1: Solution A - Return session token in response body**

Return the token within the session object, resulting in the follow example code:

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

const account = new Account(client);
const session = await account.createAnonymousSession();
const sessionToken = session.token;
```

**Problem #1: Solution B - Include headers within the response**

Return the session cookie in the response headers.

```js
import { parse } from "set-cookie-parser";
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

const account = new Account(client);
const session = await account.createAnonymousSession();

const sessionToken = parse(session.headers["set-cookie"]).find((cookie) =>
  cookie.name.startsWith("a_session")
).value;
```

**Problem #2: Solution A - Temporary OAuth2 token exchange for session**

- Modify the oauth2 flow to include a temporary token in the final redirect.
- Add a new endpoint, to exchange the temporary token for a session token.

Here's a visualisation of the new flow:

![SSR OAuth2 Flow Sequence Diagram](ssr-oauth-flow.png)

Step by step:

1. Client makes a 'Create OAuth2 Session' request to the Server, containing the user specified provider.
2. Server makes a 'Create OAuth2 Session' request to Appwrite, containing the provider, and a success authentication redirect URL.
3. Appwrite returns a URL for a page to authenticate with the OAuth2 provider.
4. Server returns the authentication URL to the Client.
5. User is redirected to the authentication URL, and authenticates with the OAuth2 provider.
6. OAuth2 provider redirects the browser back to Appwrite.
7. Appwrite sets the session cookie on the Appwrite domain, and redirects the browser to the success URL, **which now includes a userId & temporary token.** e.g. `myssrapp.com/oauth2/success?userId=387asdf7rh42346&token=adfh38khjasd83j`
8. Server exchanges the temporary token for a session token, using the new exchange endpoint, and sets the session cookie on the Server domain.

The SSR application must set up the success page to call the exchange endpoint. The page can then set session cookie on the SSR domain.
`

**Problem #3: Solution A - setSession helper method**

A new SDK helper method, called `setSession`, `setToken`, `setSecret`, or `setSessionSecret` to set the session token of future requests.

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

client.setSession(sessionToken);

const account = new Account(client);
const currentUser = await account.get();
```

### New API Endpoints

**PUT /v1/account/sessions/oauth/exchange** - Exchange a temporary OAuth2 token for a session token.

**Request**

| Name   | Type   | Description             |
| ------ | ------ | ----------------------- |
| token  | String | Temporary OAuth2 token. |
| userId | String | User ID.                |

**Response**

Session object.

### Data Structure

**Session Object Additions**

| Name   | Type   | Description    |
| ------ | ------ | -------------- |
| secret | String | Session token. |

### Documentation & Content

#### What **docs** would support this feature?

- SDK docs and examples for the new helper method.
- SDK docs and examples for the new token exchange endpoint.

#### What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?

- Generic Tutorial for using Appwrite for SSR
- Tutorial for using Appwrite and Next.js for SSR
- Tutorial for using Appwrite and SvelteKit for SSR

#### What **demo applications** can help us demonstrate this feature APIs and capabilities?

- Update Almost SSR examples.

### Prior art

[prior-art]: #prior-art

- https://supabase.com/docs/guides/auth/server-side-rendering
- https://firebase.nuxtjs.org/tutorials/ssr.html
- https://github.com/gladly-team/next-firebase-auth

### Unresolved questions

[unresolved-questions]: #unresolved-questions

**Problem #1**

1.1. Which solution should we implement? A or B?

**Problem #2**

2.1. Should we remove the undocumented workaround?

2.2. Do we need to add a new endpoint to exchange the temporary token for a session token? Can we reuse the existing magic URL exchange endpoint?

2.3. The Create OAuth2 Session endpoints are not included with Server Side SDKs. Should we include them? If so, server side apps need the Location header to forward to the user. How should we make this accessible from the SDK?

**Problem #3**

3.1. Should the helper method be available in Server Side SDKs? How do we seperate requests that are authenticated with a session token, and requests that are authenticated with an API key?

**General**

4.1. Incorporating some of these SSR changes reduces the distinction between the Server Side and Client Side SDKs. Which SDKs would be recommended for SSR applications?

### Future possibilities

[future-possibilities]: #future-possibilities

**Cookie helper methods for SSR frameworks**

On the web, SSR applications will need to set cookies under the SSR domain, but the cookie returned by Appwrite is set under the Appwrite domain.

Each SSR framework has an interface for setting cookies in the response.
For example, in Next.js, you can use the [setCookie](https://nextjs.org/docs/api-reference/next/cookies#setcookie) method.
SvelteKit has a [cookie](https://kit.svelte.dev/docs#modules-cookie) module.

For Next.js, the code would look like this:

```js
import { cookies } from "next/headers";

const cookieList = cookies();
cookieList.set({
  name: "my_cookie_name",
  value: session.token,
  path: "/",
  sameSite: "none",
  secure: true,
  httpOnly: true,
  maxAge: session.expire,
});
```

We can provide a helper method for popular SSR frameworks to set the cookie in the response.

```js
import { cookies } from "next/headers";
import { createSessionCookie } from "appwrite/nextjs";

const cookieList = cookies();
cookieList.set(createSessionCookie(session, "my_cookie_name"));
```

Popular JS frameworks worth including:

- Next
- SvelteKit
- Nuxt
- Remix
