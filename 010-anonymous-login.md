# Anonymous Login

- Implementation Owner: @torstendittmann
- Start Date: 15-02-2021
- Target Date: 16-02-2021
- Appwrite Issue: https://github.com/appwrite/appwrite/discussions/907

## Summary

[summary]: #summary

This RFC will introduce authentication of anonymous users in Appwrite. These temporary anonymous users can be used to allow users who haven't yet signed up to an app to work with data protected by Appwrite's permission and security rules. If an anonymous user decides to sign up to an app, they can convert their account to an E-Mail or OAuth2 authenticated one, so that they can continue to work with their protected data in future sessions.

## #Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

When you develop an application, there might be times when you want to let a user interact with pieces of Appwrite before they're signed in.

**What is the context or background in which this problem exists?**

For example, say you're an online retailer, you might want to let users browse your inventory and place items in a shopping cart. As the retailer, you don't know who the user is, so they're anonymous. A user might remain anonymous if they never choose to sign in. If they do choose to sign in, then the anonymous becomes a known user and their anonynmous account is converted from anonymous to a user account.

**Once the proposal is implemented, how will the system change?**

Appwrite will offer a project wide feature to enable anonymous logins.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

An anonymous user will be treated like a unverified normal user with following limitations:

- No Name
- No E-Mail address

The user can convert their account to one, that is authenticated via E-Mail or an OAuth2 provider. To convert an account to one that is authenticated with an E-Mail address, we should offer an endpoint similiar to the `/v1/account/verification` one. Meaning, an accouunt can only be converted by clicking a verification link form an E-Mail. This way we ensure an anonymous account doesn't get lost with a typo in the E-Mail address. The flow should look like this:

- Anonymous user initiate the E-Mail verification process
- Verifies his E-Mail via the link provided by the E-mail
- Will only be converted to a normal user at this point.

The process for OAuth2 should be very simple, once a user has linked an OAuth2 authentification to an anonymous account - it will be converted.

For data consistency, the user should be provided the same ID at any point.

### Endpoints

#### `GET: /v1/account/sessions/anonymous`

Allow a user to create and login to an anonymous user account.

#### `POST: /v1/account/verification/anonymous`

Allow a user to initiate converting their account to one that is authenticated via E-Mail. This will have the same functionality as the `POST/v1/account/verification` endpoint, except that an E-Mail address needs to be provided in the payload.

#### `PUT: /v1/account/verification/anonymous`

This will have the same functionality as the `PUT/v1/account/verification` endpoint to complete converting the anonymous account. Maybe one endpoint could serve both use cases, but I would prefer the single responsibility.

### Prior art

[prior-art]: #prior-art

Luckily we have a lot of resources and examples around the internet that we can take as an example. I don't think I need to name them here, since a simple Google search will satisfy everyone's curiosity.

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

**What happens if the user wants to convert his anonymous account to his already existing user account?**

To be fair, I have no idea how to handle this properly. We could convert his data to his old account and also transfer all his stuff to the old user id. But this would break consistency, if there are any references in the database to the anonymous user id.

**What happens to unused anonymous accounts?**

Should we add a TTL for anonymous accounts? Let's say no login to an anonymous user happened in 90 days, we should probably delete that user. This option should be configurable.

**If we add one more way of authentication - it will get messy!**

As mentioned in our meeting before, we might want to decouple the user accounts from email authentication. Meaning, that an account can have multiple ways of authentication. This way we could also disable users from abusing the endpoint of creating an account with an E-Mail address if this is not wanted from the owner of an Appwrite project. 

For example:

I create a new project and in the settings I can choose what authentication methods I want to use. Let's say I want to use E-Mail and Google OAuth2. A user creates an account via the the `POST /v1/account/email` endpoint. This will create an user with that E-Mail and Password as one possible way of logging into this account. Now the user also wants to connect his Google Account via the `GET /v1/account/oauth2/{provider}` endpoint. This endpoint will recognize a already logged in user due to cookies and will add that authentication method to the account. Now all of a sudden the owner needs to add GitHub as an OAuth2 provider to attract more developers. This would also allow multiple E-Mail addresses for a single account.

Also, if we disable E-Mail authentication - this way we can ensure users not creating such accounts over the REST API.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
