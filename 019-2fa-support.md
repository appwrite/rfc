# 2-factor authentication

- Implementation Owner: @Meldiron
- Start Date: 06-02-2022
- Target Date: x
- Appwrite Issue: x

## Summary

[summary]: #summary

Appwrite currently only provides password verification as a security step to validate a visitor is owner of the account. This RFC will improve this process by allowing user to setup 2FA, and protect session creation with a 6-digit code.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

On internet it's pretty common to have 2 factor authentication for security purposes, and Appwrite does not support it. This RFC will make Appwrite a safer BaaS, since it is currently not possible to have 2FA (impossible even with Functions).

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

The 2FA code generation and verification looks complicated, but I found a library with bunch of downloads that I would like to use: [bacon/bacon-qr-code](https://packagist.org/packages/bacon/bacon-qr-code).

The whole 2FA implementation will kept with-in client-SDK. There will be a new endpoint to start process of enabling 2FA on an account, that will return a `secret` that user has to type into their 2FA application. Developers can use existing `avatars.getQR()` method to convert secret into scanable QR code. Finally, the process is finished by hitting another endpoint to confirm you have scanned the QR code. There, you need to provide your first 6-digit numeric code from the 2FA application. Once the process is finished, the response will also include 10 alpha-numeric codes each 10 characters long. These are backup keys and can be used for authentication, similiar to 6-digit numeric codes, but these never expire.

There will now be optional parameter for `code` when using `account.createSession()` method. This parameter is ignored if 2FA is disabned, but is required if 2FA is enabled. We validate the code before creating the session. Similarly, we will protect all other endpoints for creatign sessions: `createOAuth2Session()`, `updateMagicURLSession()`.

All important account-updaring endpoints should now also require `code` for verification: `updatePassword()`, `updateEmail()`, `delete()`, `updateRecovery()`.

Last but not least, there will be an endpoint to disable 2FA on the account. This action needs to be confirmed with current `password` and 2FA `code`.

A new activity should be created each time 2FA is enabled or disabled. No Realtime changes are required, since this feature does not need realtime functionality. 

Talking about UI, there should be indicator in Appwrite Console saying that account has 2FA enabled, similiar to `Verified` badge. This means user's response model will need new rule `2fa` boolean, saying if 2FA is enabled on this account or not.

To ensure 2FA can be used to it's full power, we need to allow developers to use 2FA for their functionality. This means, there will be a Server-SDK endpoint for verifying 2FA codes, so developers can have their Appwrite Functions protected by 2FA codes.

### API Endpoints

**POST /v1/account/2fa**

- `sdk.account.create2fa()` (role:member)
- Returns `{ secret: '[SOME_SECRET_CODE]' }`

**PUT /v1/account/2fa**

- `sdk.account.update2fa('[6_DIGIT_CODE]', '[CURRENT_PASSWORD]')` (role:member)
- Returns `{ backupCodes: [ '3ag7ZNgDD2', 'y8efSVgWrO', 'lrwl712xyj' ] }` (10 codes)

**DELETE /v1/account/2fa**

- `sdk.account.delete2fa('[6_DIGIT_CODE]', '[CURRENT_PASSWORD]')` (role:member)
- Returns `{ backupCodes: [ '3ag7ZNgDD2', 'y8efSVgWrO', 'lrwl712xyj' ] }` (10 codes)

**GET /v1/users/{userId}/2fa**

- `sdk.users.verify2fa('[USER_ID]', '[6_DIGIT_CODE]')` (role:api_key)
- Returns 2XX if code is valid, but throws 4XX error if code could not be verified. Should also throw error if 2FA is not enabled on the account (you can use `users.get()` to check if 2FA is enabled)

### Data Structure

`users` collection will get new attributes:

- `2faSecret` string, a secret used to verify 6-digit codes. If null, 2fa is not enabled
- `backupCodes` string array, 10 backup codes that can be used infinitely

Both attributes will have `encrypt` filter, so we dont store them as a plain text.

### Supporting Libraries

A 3rd party library [bacon/bacon-qr-code](https://packagist.org/packages/bacon/bacon-qr-code) will be used for generating 2FA secrets, and verifying codes. We could create and maintain such a library under utopia-php, but I don't have enough knowledge about the topic to confidently create a library.

QR codes will be generated using existing API endpoints - no new library is required there.

### Breaking Changes

No breaking changes. We will be adding new endpoints and optional parameters.

Migration not required. Default null values will be fine for newly added attributes.

### Reliability (Tests & Benchmarks)

#### Scaling

Same as any other Appwrite API endpoint. We are not querying any external API. If the endpoint ends up taking a lot of CPU, we can enable abuse-control on it. If verification takes too much time, we can disable `encrypt` filter on `2faSecret` attribute.

#### Benchmarks

We will benchmark how many secrets can Appwrite generage in 30 seconds (`create2fa`), and how many verification can it do (`createSession`). With bad results, we can enable rate limits.

#### Tests (UI, Unit, E2E)

There will be E2E test testing out the whole flow of enabling 2FA, testing all endpoints that should be protected and disabling 2FA. Current tests should pass - by doing that, they test if `code` is properly optional if 2FA is disabled.

Unit tests are not required and should be part of 3rd party libary.

UI tests are not required, as we are not doing any huge changes to the UI.

### Documentation & Content

Documentation changes are not required, as all important information will be generated from Swagger docs.

A demo application alongisde article show-casing how to use this feature should be published, since there is a trick of using AvatarsAPI together with AccountAPI (generate QR code from secret).

### Prior art

[prior-art]: #prior-art

I didn't do too deep research into existing solution, the RFC was created from my past experience with 2FA. All websites differ in the level of security a little (when 2QA should be asked), and how backup codes works. I think this implementation is great for first iteration and we could improve it with community feedback, if required.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

1. Should some new endpoints be rate-limited? There is no outside communication, but I also don't think we should let user generate new secrets every seconds without actually enabling the 2FA.
2. Should this feature interact with server-SDK in any way?
3. Does OAuth account require 2FA verification? Not sure if I ever saw that.. You relay on Google's or Discord's 2FA, right?
4. I noticed critical account updates are not protected by password. Why? Which endpoints shoud be protected by nothing, which by passowrd-only, and which by both password and 2FA?

### Future possibilities

[future-possibilities]: #future-possibilities

1. We could expose 2FA to console-sdk too, and allow it in Appwrite Console
2. Parameter option to destroy all sessions after enabling 2FA. Same would be cool for password change
