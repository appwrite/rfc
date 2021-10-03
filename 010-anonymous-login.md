# Anonymous Login

- Implementation Owner: @TorstenDittmann
- Start Date: 15-02-2021
- Target Date: 16-02-2021
- Appwrite Issue: https://github.com/appwrite/appwrite/discussions/907

## Summary

[summary]: #summary

This RFC will introduce authentication of anonymous users in Appwrite. These temporary anonymous users can be used to allow users who haven't yet signed up to an app to work with data protected by Appwrite's permission and security rules. If an anonymous user decides to sign up to an app, they can convert their account to an E-Mail or OAuth2 authenticated one, so that they can continue to work with their protected data in future sessions.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

When you develop an application, there might be times when you want to let a user interact with pieces of Appwrite before they're signed in. This also increases the conversion rate of applications, since the hurdle of registering is very high.

**What is the context or background in which this problem exists?**

For example, say you're an online retailer, you might want to let users browse your inventory and place items in a shopping cart. As the retailer, you don't know who the user is, so they're anonymous. A user might remain anonymous if they never choose to sign in. If they do choose to sign in, then the anonymous becomes a known user and their anonymous account is converted from anonymous to a user account.

**Once the proposal is implemented, how will the system change?**

Appwrite will offer a project wide feature to enable anonymous logins.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

An anonymous user will be treated like a unverified normal user with following limitations:

- No Name
- No E-Mail address
- No Password

The user can convert their account to one, that is authenticated via E-Mail or an OAuth2 provider. To convert an account to one that is authenticated with an E-Mail address, we will use the `PATCH:/v1/account/email` endpoint which requires an email and password for the request. Normally This endpoint is used to update the users E-mail address and requires a password. Since both is not set on an anonymous account, we can use this existing endpoint to save both values to the anonymous account - therefore converting it to a user account. The flow should look like this:

- Anonymous user gets created and uses Appwrite features without providing any personal informations.
- Passes E-mail address and password to the `PATCH:/v1/account/email` endpoint.
- A successful request will convert this account.

The process for OAuth2 should be very simple, once a user has linked an OAuth2 authentification to an anonymous account - it will be converted.

For data consistency, the user should be provided the same ID at any point.

### Endpoints

#### `POST: /v1/account/sessions/anonymous`

Allow a user to create and login to an anonymous user account.

#### `PATCH: /v1/account/email`

This endpoint will be adapted, to behave different on an account that doesn't have E-Mail and Password set (`null` in this case). If the endpoint turns out to be requested from an anonymous user, it will do the following:

- Checks if the E-Mail address is already registered to an account.
  - If no account was found with provided E-Mail Address:
    - Save E-Mail Address
    - Save Password
  - If an account was found:
    - Throws an error

### Prior art

[prior-art]: #prior-art

Luckily we have a lot of resources and examples around the internet that we can take as an example. I don't think I need to name them here, since a simple Google search will satisfy everyone's curiosity.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

**What happens to unused anonymous accounts?**

> As commented, this will remain untouched and is in the owners responsibility.

Should we add a TTL for anonymous accounts? Let's say no login to an anonymous user happened in 90 days, we should probably delete that user. This option should be configurable.

**If we add one more way of authentication - it will get messy!**

> This will be covered in a separate RFC.

As mentioned in our meeting before, we might want to decouple the user accounts from email authentication. Meaning, that an account can have multiple ways of authentication. This way we could also disable users from abusing the endpoint of creating an account with an E-Mail address if this is not wanted from the owner of an Appwrite project. 

For example:

I create a new project and in the settings I can choose what authentication methods I want to use. Let's say I want to use E-Mail and Google OAuth2. A user creates an account via the the `POST /v1/account/email` endpoint. This will create an user with that E-Mail and Password as one possible way of logging into this account. Now the user also wants to connect his Google Account via the `GET /v1/account/oauth2/{provider}` endpoint. This endpoint will recognize a already logged in user due to cookies and will add that authentication method to the account. Now all of a sudden the owner needs to add GitHub as an OAuth2 provider to attract more developers. This would also allow multiple E-Mail addresses for a single account.

Also, if we disable E-Mail authentication in a project - this way we can ensure users not creating such accounts over the REST API.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
