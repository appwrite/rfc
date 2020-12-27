# JWT Support for Server API Authentication <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux
- Start Date: 26-12-2021
- Target Date: Unknown
- Appwrite Issue:
  https://github.com/appwrite/appwrite/issues/511

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

JSON Web Tokens are an open, industry-standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties. By adding support for JWT authentication, Appwrite developers will be able to use their APIs in the user scope. APIs that are currently restricted to the client-side could be useful on the server-side as well. This will allow developers to create interesting new use-cases for Appwrite.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

Appwrite Server API is limited and can't access the different services in the user scope and perform an action on his/her behalf. The only current way to use the server API is in admin mode combined with an API key that has relevant permissions scopes. This is limiting the usage in Account, Database, Storage, and other Appwrite APIs.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

The current implementation of server API scopes functionality and flexibility is limited. The server API is especially limited when trying to integrate with a custom server or 3rd party databases.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

A new authentication method will be introduced to the Appwrite server API. The new authentication method will allow developers to act as a user from their own servers.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Create a new API Endpoint

Create a new API endpoint called 'createJWT' `POST /v1/account/jwt`. The new endpoint should be available to an authenticated user. The number of API calls should be limited and protected by the abuse object. We can use [adhocore/php-jwt](https://github.com/adhocore/php-jwt) for token creation and validation. Include the UserID and SessionId as part of the JWT payload.

According to [Hasura](https://hasura.io/blog/best-practices-of-using-jwt-with-graphql/#:~:text=This%20is%20why%20JWTs%20have,JWTs%20don't%20get%20leaked.), the best practice for the expiry value of the JWT is around 15 minutes. This combined with the fact the JWT auth is depnedent on a valid auth session should give a good level of security for the Appwrite project users.

### Create a New JWT Model

Create a new JWT response model to be returned by the new API endpoint. New model should be located at: `src/Appwrite/Utopia/Response/Model`.

### Allow New Auth Method

Allow a new auth method using a header that will contain the JWT secret. If valid, the API will allow the server to perform all actions under the relevant user. The server will also grant access to any resources (files, documents, etc...) belonging to the user. Verify the sessionId and userId are valid when verifing the JWT auth header.

### Update the Server SDKs

Add all missing API that used to be relevant only for client integrations. Add a new method for attaching JWT secret to the SDK HTTP client. 

### Update Documentation

List all the new server endpoints that are now available on the server API. Add a JWT as a new authentication method for all the relevant API endpoints.

### Tutorial

Create a short tutorial that explains how to generate a new JWT on the Web/Flutter/both. Create a server example for consuming the API (Node/Python/PHP/Ruby/Deno) and authenticating a user, and making actions on his behalf.

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is reasonably clear how the contribution would be implemented
- Dependencies on libraries, tools, projects or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables
- Changes to documentation
- Ways to scale the solution

Ensure that you include examples, code-snippets etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal. A
few examples of what this can include are:

- Does this functionality exist in other software and what experience has their
  community had?
- For other teams: What lessons can we learn from what other communities have
  done here?
- Papers: Are there any published papers or great posts that discuss this? If
  you have some relevant papers to refer to, this can serve as a more detailed
  theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other software.

Write your answer below.
-->

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

Not sure what header name we should use for the new authentication method. I would probably go for X-Appwrite-JWT, although Authorization', "Bearer $token" seems to be the go-to option doe most public APIs. The idea to use a custom header is because we have multiple authentication methods (api-key,session,jwt). As far as I can tell there is no standard way to distinguish between them on the server-side. I would avoid making any wrong usage in a common header that will force us to make API breaking changes.

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

This feature will create a lot of new use-cases and will probably force us to make the API even more flexible, and new ideas come from the community.
