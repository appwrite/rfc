# Server Side Rendering

- Implementation Owner: @loks0n
- Start Date: 31-08-2023
- Target Date: Unknown

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
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






## Design proposal (Step 2)

[design-proposal]: #design-proposal

**Solution #1: Basic account session cookies are not accessible from SDK**

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
const sessionCookie = parse(cookies).find(
    (cookie) => cookie.name === "a_session"
);
const sessionToken = sessionCookie?.value;
```

The document proposes that the Account service should return the token within the session object, simplifying the code above to:

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

const account = new Account(client);
const session = await account.createAnonymousSession();
const sessionToken = session.token;
```

**Solution #2: OAuth2 session cookies are not accessible from SDK**


**Solution #3: Making authenticated requests with the SDK**

Now, you've got a session token, you want to make authenticated requests. For example, getting the user's account details. To set a session token on the server-side, we have to use the `X-Fallback-Cookies` header


```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

const serialisedCookies = JSON.stringify({
    "a_session_PROJECT_ID": sessionToken,
});

client.headers["X-Fallback-Cookies"] = serialisedCookies;

const account = new Account(client);
const currentUser = account.get();
```

We propose a new SDK helper method to simplify this process.

```js
import { Client, Account } from "appwrite";

const client = new Client();
client.setEndpoint("https://cloud.appwrite.io/v1");
client.setProject("PROJECT_ID");

client.setSession(sessionToken);

const account = new Account(client);
const currentUser = account.get();
```

**Solution #4: Persisting cookie sessions on client**

On the web, SSR applications will need to set cookies under the SSR domain, but the cookie returned by Appwrite is set under the Appwrite domain.

Each SSR framework has an interface for setting cookies in the response. 
For example, in Next.js, you can use the [setCookie](https://nextjs.org/docs/api-reference/next/cookies#setcookie) method.
SvelteKit has a [cookie](https://kit.svelte.dev/docs#modules-cookie) module.

For Next.js, the code would look like this:

```js

import { cookies } from "next/headers";

const cookieList = cookies()
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

We propose a helper method for popular SSR frameworks to set the cookie in the response.

```js
import { cookies } from "next/headers";
import { createSessionCookie } from "appwrite/nextjs";

const cookieList = cookies()
cookieList.set(createSessionCookie(session, 'my_cookie_name'));
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



### API Endpoints

<!--
List the new API routes or endpoints that we might need to add for supporting the new feature.
Keep in mind to stay very strict to the API protocol and method, whether your new
changes are for the REST, WebSocket or any other API protocol Appwrite supports.

For example:

**POST /v1/coffee ** - an endpoint for creating coffee.
**DELETE /v1/coffee ** - an endpoint for deleting coffee.
-->

### Data Structure

<!--
What kind of changes or additions are required for the Appwrite base collections
to support this feature. Explain which entities should be added or updated, what new attributes they
need to have and why. Please think well about the naming conventions and how well they play with other
Appwrite conventions. Try and stay as consistent with existing patterns as much as possible.
-->

### Supporting Libraries

<!--
Which different libraries do we need to support the new features?
Please describe the new library's potential API?
Avoid using 3rd party libraries when possible, if required - explain why.
-->

### Breaking Changes

<!--
Do we break any API or SDK backward compatibility?
If possible, explain what actions we can take to avoid that.
-->

### Reliability (Tests & Benchmarks)

#### Scaling

<!-- Explain how we will scale this new feature. -->

#### Benchmarks

<!-- Explain how we will benchmark the new feature. -->

#### Tests (UI, Unit, E2E)

<!-- 
Explain how we will test the new feature. 
You can use "N/A" if this section is not relevant to your proposal.
-->

### Documentation & Content

<!--

Documentation is vital for making this new feature a success for both developers using Appwrite and the Appwrite maintainers.
Please answer the following questions:

1. What **docs** would support this feature?
2. Do we need to update the **contribution guide** with a new section or a supporting tutorial?
3. What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?
4. What **demo applications** can help us demonstrate this feature APIs and capabilities? 

-->

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Does this functionality exist in other software, and what experience has their community had?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us, whether they are brand new or an adaptation from other software.

Write your answer below.
-->

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
