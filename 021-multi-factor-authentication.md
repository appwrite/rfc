# Multi Factor Authentication

- Implementation Owner: @TorstenDittmann
- Start Date: 12-06-2023
- Target Date: 01-07-2023
- Appwrite Issue: [#5240](https://github.com/appwrite/appwrite/issues/5240)

## Summary

[summary]: #summary

Single factor authentication poses a security risk to user accounts. If a user's password is compromised, their account can be easily accessed by an attacker. To mitigate this risk, I propose adding Multi Factor Authentication (MFA) to Appwrite's Accounts Service.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Appwrite currently only supports single factor authentication. This presents a security risk as it leaves user accounts vulnerable to attacks such as password guessing and phishing. To address this, I propose implementing Multi Factor Authentication (MFA) in Appwrite's Accounts Service.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Flow

Scenario: User **Walter** has 2FA on his account enabled.

- Walter attempts to create a session using his e-mail and password
  - Appwrite identifies that he has MFA enabled
  - Appwrite creates a challenge
  - Appwrite returns an exception with the type `mfa_required`, challenge token and user id, indicating to the user that MFA is needed to continue
- Walter's application retrieves the list of available providers with the user id
- Walter's application initiates the MFA challenge with the token and a provider of choice
- Walter chooses TOTP, enters the current authentication key to verifiy the challenge
- The challenge was successful and a session is created

It is important to point out, that the attempt to create a session when MFA is enabled will throw an exception - making the developer aware of additional steps.

The exception will contain the informations needed to create and verify a challenge.

### API Endpoints

I propose the following endpoints for MFA implementation:

#### Account Service
`PATCH:  /v1/account/mfa` enables/disables MFA for the account
`POST:   /v1/account/mfa/totp` adds TOTP authenticator to account
`PUT:    /v1/account/mfa/totp` verifies added TOTP authenticator
`DELETE: /v1/account/mfa/totp` removes TOTP authenticator from account
`GET:    /v1/account/mfa/providers`: returns a list of available MFA providers
`POST:   /v1/account/mfa/challenge`: initiates an MFA challenge with the provider of choice and returns a challenge token
`PUT:    /v1/account/mfa/challenge`: verifies an MFA challenge and generates a session

#### User Service
`PATCH:  /v1/user/mfa`: enables/disables disables MFA for the user
`GET:    /v1/user/mfa/providers`: returns a list of available MFA providers
`DELETE: /v1/user/mfa/totp`: removes authenticator from user

Obviously all crucial/sensitive endpoints will be covered by rate limits.

### Data Structure

For my proposal I need to add a new internal collection called `challenges` with following data structure (excluding default `$` attributes):
- `token` secret to be used for identification, similiar to the `secret` of the tokens collections
- `provider` the provider used for the challenge (TOTP, SMS, E-Mail, etc)
- `session` the type of session that has triggered the creation of the challenge

Additionally we need to add new properties to the `user` collection:
- `mfa` boolean to whether MFA is enabled or not
- `totp` boolean to whether a TOTP device is active
- `totp_secret` the secret key to be used with a TOTP authentication app
- `totp_backup` array of backup codes that can each used once

### Supporting Libraries

Following libraries can be used as dependency or inspiration:

- https://github.com/Spomky-Labs/otphp
- https://github.com/lfkeitel/php-totp

### Breaking Changes

No breaking changes will be introduced on the API. But a few data structure changes will have to be migrated.

### Reliability (Tests & Benchmarks)

#### Scaling

This should already be scalable out-of-the-box with existing Appwrite scalability.

#### Benchmarks

Due to the nature of this feature, I don't expect any complications with performance. Of course this will be manually checked during migration.

#### Tests (UI, Unit, E2E)

- Unit Tests with highest Coverage possible
- E2E Tests covering:
  - all new endpoints
  - all challenges tested for endpoints creating a session

### Documentation & Content

We probably need a new Section in the Documentation explainig the steps to implement MFA.

### Prior art

[prior-art]: #prior-art

I investigate existing MFA implementations and best practices to inform our design proposal. Specifically, I will review MFA implementation in other authentication services, such as Auth0 and Okta.

Also existing RFC's for TOTP are a helpful resource ([RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238)).

Additionally, this feature is planned in a way - that we can extend the MFA to be used with more than 2 factors in the future.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

- A project wide setting that MFA needs to be used
- A team wide setting that MFA is required
- We might wanna think about adding additional roles for users with MFA

### Future possibilities

[future-possibilities]: #future-possibilities

Right now this RFC only covers a challenge adding a second factor, in the future we can extend with even more factors need in total and also different factors than TOTP, E-Mail and SMS. Something like Captcha, HOTP, wabauthn and co.
