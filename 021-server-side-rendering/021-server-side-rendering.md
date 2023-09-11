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

To authenticate using anonymous sessions, email-password sessions and magic URL sessions, the Appwrite SDKs make a POST request to the relevant endpoint. The response body contains the session ID, but not the

The SDK currently obscures the session cookie value with the [call method](https://github.com/appwrite/sdk-for-node/blob/aaea14a36d5b7daac859eaa8dc44d2253fbcbcef/lib/client.js#L120C86-L120C86) which only returns the response body.

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

The SDK for flutter has a similiar problem, but bypasses it by adding and interceptor to the Client requests. The interceptor sets cookies (in the apps 'cookie jar') when receiving set-cookie responses, and retrieves the cookie when making requests. For server side SDKs, this is not possible, as there isn't an equivalent method persistent storage.

**Problem #2: Accessing OAuth2 sessions**

The current oauth2 flow looks something like this:

![CSR OAuth2 Flow Sequence Diagram](csr-oauth-flow.png)

1. Browser makes a GET request to the Appwrite API.
2. Appwrite API redirects the browser to the OAuth2 provider.
3. User authenticates with the OAuth2 provider.
4. OAuth2 provider redirects the browser back to the Appwrite API.
5. Appwrite API sets the session cookie on the Appwrite domain.
6. Appwrite API redirects the browser back to the client application.

This is incompatible with SSR applications, because the session cookie is set on the Appwrite domain, and not the SSR domain.

Interestingly, there is an undocumented workaround for this. If the developer sets the success parameter to `{SSR_DOMAIN}/auth/oauth2/success` the Appwrite API will include the session cookie as a query parameter in the redirect URL. [Source here](https://github.com/appwrite/appwrite/blob/3f3d518f3664bcab281ee00b45dd2f2d387ffc72/app/controllers/api/account.php#L870). This method is vunerable to session spying attacks, as the session token is exposed in the URL, and could be used to hijack the session.

**Problem #3: Using session tokens with the SDK**

Now, you've got a session token, you want to make authenticated requests. For example, getting the user's account details. To set a session token on the server-side, we have to use the `X-Fallback-Cookies` header

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
const currentUser = account.get();
```

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
- Implement a new endpoint, or extend the existing functionality of the magic URL endpoint, to exchange the temporary token for a session token.

![SSR OAuth2 Flow Sequence Diagram](ssr-oauth-flow.png)

1. Browser makes a GET request to the SSR application.
2. SSR application makes a GET request to the Appwrite API.
3. Appwrite API redirects the SSR application to the OAuth2 provider.
4. The SSR application redirects the client to the OAuth2 provider.
5. User authenticates with the OAuth2 provider.
6. OAuth2 provider redirects the browser back to the Appwrite API.
7. Appwrite API sets the session cookie on the Appwrite domain.
8. Appwrite API redirects the browser back to the client application. **Now, the redirect URL includes a userId & temporary token.** e.g. `myssrapp.com/oauth2/success?userId=loks0n&token=TEMPORARY_TOKEN`
9. The SSR application makes a POST request to the Appwrite API to exchange the temporary token for a session token.

The SSR application must set up the success page to call the exchange endpoint. The page can then set session cookie on the SSR domain.
`

**Problem #3: Solution A - setSession helper method**

A new SDK helper method, called `setSession`, to set the session token of future requests.

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

client.setSession(sessionToken);

const account = new Account(client);
const currentUser = account.get();
```

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail, keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is reasonably clear how the contribution would be implemented
- Dependencies on libraries, tools, projects, or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables
- Changes to documentation
- Ways to scale the solution

Ensure that you include examples and code snippets to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation to suggest changes early on during the development.**

Write your answer below.
-->

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
