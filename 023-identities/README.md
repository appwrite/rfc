# Identities <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @stnguyne90
- Start Date: 12-06-2023
- Target Date: 01-07-2023
- Appwrite Issue:
  * https://github.com/appwrite/appwrite/issues/3924
  * https://github.com/appwrite/appwrite/issues/3680
  * https://github.com/appwrite/appwrite/issues/1192

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Allow users to connect multiple accounts to their Appwrite account regardless of the email address in the external system.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

At the moment, external accounts from an OAuth2 provider are connected to Appwrite via email. When a user creates an OAuth2 session, the email provided by the OAuth2 provider is looked up in Appwrite to find the the existing related user. If the user is not found, a new user is created. The problem with this is a person may have different emails in different OAuth2 providers which would result in a new user being created.

Another problem with our current design is the OAuth2 information is integrated into the session. Because of that, once a session is deleted, there is no information about the OAuth2 provider that was used to create the session (and possibly user).

## Design proposal (Step 2)

[design-proposal]: #design-proposal

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

A user will be able to have multiple identities where each identity is a connection to an OAuth2 provider.

A user should be able to:

1. Connect their account to an OAuth2 provider, creating an identity
2. List their identities
3. Update their identity
   * This will refresh the access token
4. Disconnect their account from an OAuth2 provider, deleting the identity
5. Create an OAuth2 session with an existing account without a connected OAuth2 provider
   * This will automatically create an identity
6. Create an OAuth2 session without creating an account (backwards compatibility)

A developer should be able to:

1. List a user's identities
2. Delete a user's identity

### API Endpoints

<!--
List the new API routes or endpoints that we might need to add for supporting the new feature.
Keep in mind to stay very strict to the API protocol and method, whether your new
changes are for the REST, WebSocket or any other API protocol Appwrite supports.

For example:

**POST /v1/coffee ** - an endpoint for creating coffee.
**DELETE /v1/coffee ** - an endpoint for deleting coffee.
-->

#### Account Service

##### Create Identity

The existing Create OAuth2 Session endpoints will be re-used to create an identity

**GET /v1/account/sessions/oauth2/{provider}**

##### List Identities

**GET /v1/account/identities**

Params:

* queries: it should be possible to filter the queries by provider

Response:

List of [Identities](#identity)

##### Get an Identity

**GET /v1/account/identities/{id}**

Response:

[Identity](#identity)

##### Refresh Identity

**PATCH /v1/account/identities/{id}**

Response:

[Identity](#identity)

##### Delete Identity

**DELETE /v1/account/identities/{id}**

Response:

None

#### Users Service

##### List Identities

**GET /v1/users/identities**

Params:

* queries: it should be possible to filter the queries by provider and userId

Response:

List of [Identities](#identity)

##### Refresh Identity

**PATCH /v1/users/identities/{id}**

Response:

[Identity](#identity)

##### Delete Identity

**DELETE /v1/users/identities/{id}**

Response:

None

### Data Structure

<!--
What kind of changes or additions are required for the Appwrite base collections
to support this feature. Explain which entities should be added or updated, what new attributes they
need to have and why. Please think well about the naming conventions and how well they play with other
Appwrite conventions. Try and stay as consistent with existing patterns as much as possible.
-->

#### Identity

##### Private

* userInternalId: internal id of the user to prevent linking recreated users with old identities

##### Public

* provider: OAuth2 provider
* providerUid: identifier from the provider
* userId: user id from the provider
* providerEmail: email from provider
* status: connected, disconnected
* accessToken: access token from the provider
* accessTokenExpiry: the date when the access token expires in ISO 8601 format
* providerRefreshToken: refresh token from the provider

### Flow

#### User

##### Connecting to an OAuth2 provider

1. User makes authenticated API call to `GET /v1/account/sessions/oauth2/{provider}`
2. User is redirected to OAuth2 Provider
3. User signs in to OAuth2 Provider
4. User is redirected back to Appwrite at `GET /v1/account/sessions/oauth2/callback/{provider}/{projectId}`
5. User is redirected to the handler at `GET /v1/account/sessions/oauth2/{provider}/redirect`
6. Appwrite exchanges the code for a refresh and access token
7. Appwrite looks up the user Id and email from the provider
8. Appwrite looks up to see if there is already an identity for the provider
9. If there is an identity, the access and refresh tokens are refreshed
10. If there is no identity, a new identity is created
11. User is redirected to the success URL

### Key Considerations

1. The email from the OAuth2 provider must not match the email of another user in Appwrite.
2. Provider and the user ID from the OAuth2 provider must be unique so that an identity will only be for one user.
3. Migration will be required to create the identities collection.
4. Events should fire for various actions on identities.

### Supporting Libraries

<!--
Which different libraries do we need to support the new features?
Please describe the new library's potential API?
Avoid using 3rd party libraries when possible, if required - explain why.
-->

The SDKs will need to be updated to support the new endpoints.

### Breaking Changes

<!--
Do we break any API or SDK backward compatibility?
If possible, explain what actions we can take to avoid that.
-->

The existing OAuth2 APIs will still work as they do now. The only difference is an identity will be created if one does not exist.

A migration will be required to create the [Identity](#identity) collection.

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

#### Authentication Docs

The [Authentication Docs](https://appwrite.io/docs/authentication) should be updated to include a section on identities.

The [OAuth Authentication Docs](https://appwrite.io/docs/authentication#oauth) should be updated to explain an identity will be created if there is already a session.

#### API Docs

The [Create OAuth2 Session](https://appwrite.io/docs/client/account#accountCreateOAuth2Session) description shold be updated to link to the [OAuth Authentication Docs](https://appwrite.io/docs/authentication#oauth).

#### Demo Applications

The Playgrounds should be updated to demo identities.

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

#### Auth.js (NextAuth)

Auth.js has 2 separate models: [Users and Accounts](https://authjs.dev/reference/adapters#models). A User is a person that can have multiple Accounts (from a particular OAuth provider).

#### Auth0

Auth0 has 2 separate models: [Profiles and Identites](https://auth0.com/docs/manage-users/user-accounts/user-account-linking). A Profile can be linked/merged and have multiple Identities (including SMS).

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

1. How to connect an account on mobile?
   - We should explore passing a short lived token with limited access rights.
2. Should we support a way to get all identities from a particular provider?
   - This should be supported via queries
3. What to do after an identities' refresh token no longer works?
4. What should happen to related sessions when an identity is deleted?
   - We should let the developer decide whether to delete sessions. We should make sure to fire an event so that developers can take action accordingly.
5. What should happen to an identity when the related session is deleted?
   - Nothing as they should be decoupled.
6. Should we make the identities collection generic with a data attribute to put custom data for different providers?

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

1. Support multiple OAuth2 configurations per provider
