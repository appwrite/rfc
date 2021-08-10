# Phone Authentication <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @torstendittmann
- Start Date: 10-08-2021
- Target Date: (expected date of completion, dd-mm-yyyy)
- Appwrite Issue:
  - https://github.com/appwrite/appwrite/issues/1068

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

As of now we are not offering a way to authenticated via a phone number using SMS.

**What is the context or background in which this problem exists?**

Depending on an applciation target audience, authenticating via a mobile number might be required.

**Once the proposal is implemented, how will the system change?**

This implementation will pave the way for any future 2 step authentication.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

First of all, this implementation will require a third party provider like [Twilio](https://twilio.com/) or [MessageBird](https://messagebird.com/) for sending the actual SMS to a phone. Also this implementation will be the first authentication method requiring 2 steps on the client side, **Initialization** and **Completion**, to authenticate a user.

Right now the only unique value to identify a user is the E-Mail address. Since phone authentication should work without an e-mail address - this implementation will need to add another collection, that will contain all the connected providers to an account. The data structure on a user would look like this:

```json
{
  "$id": "some_id",
  "name": "Lorem Ipsum",
  "providers": {
    "email:lorem@gmail.com": false,
    "google:lorem@ipsum.com": true,
    "phone:0049123456789": true
  }
}
```

Notice the missing `email` attribute, which will now reside in a `providers` attribute. This allows multiple providers while maintaining unique identifiers per provider. The key will be in a `PROVIDER:UID` syntax with a boolean value hinting if the provider is verified or not.

This syntax is required to query the data more efficient.

> This is a big change, but required in my opinion. I have suggested this before when working on the sessions refactor - but we decided to drop it.
### Initialization

This step will initialize the authentication using a phone number as the unique identifier for a user. For phone, this will be a phone number.

#### `POST: /v1/account/sessions/phone`

The payload must contain the phone number, where the SMS is gonna be send to. This will create a random 6 digit secret number, save it as a `Token` in the database and send this number to the related phone number using the configured Third Party provider. This endpoint can be requested again to re-send the SMS.

This endpoint will return the token `$id`, which will be used in the completion step. The user would need to manage this `$id` and have it available in the next step.

### Completion

This step will compare the token's `$id` and `secret` created in the initialization step. 
#### `POST: /v1/account/sessions/phone/complete`

The payload must contain the `$id` of the Token and the 6 digit number send to the phone representing the Token's `secret`. If both values are valid, this endpoint will either login an existing user or create a new one if the phone number isn't present on any of the user provider attribute.

### Flow

┌────────────────┐
│                │
│     Start      │
│                │
└───────┬────────┘
        │
 ┌──────▼───────┐         Send SMS
 │Initialization├───────────┐
 └──────────────┘           │
                    ┌───────▼─────────┐
                    │    Provider     │
                    └───────┬─────────┘
 ┌──────────────┐           │
 │  Completion  ◄───────────┘
 └──────┬───────┘    User enters secret
        │
 ┌──────▼───────┐ No   ┌─────────────┐
 │  Is valid?   ├──────► Throw error │
 └──────┬───────┘      └─────────────┘
        │ Yes
 ┌──────▼───────┐ No   ┌─────────────┐
 │ User exists? ├──────► Create user │
 └──────┬───────┘      └──────┬──────┘
        │ Yes                 │
        │◄────────────────────┘
 ┌──────▼───────┐
 │Create session│
 └──────┬───────┘
        │
┌───────▼────────┐
│                │
│      End       │
│                │
└────────────────┘

### Providers Endpoints

This addition also needs to have more endpoints, which are needed to see which providers are linked to which account and also delete a provider from a user.

#### `GET: /v1/account/providers`
#### `GET: /v1/users/{ID}/providers`

This endpoint will return all providers linked to a user.

#### `GET: /v1/account/providers/{PROVIDER}/{UID}`
#### `GET: /v1/users/{ID}/providers/{PROVIDER}/{UID}`

This endpoint returns a single provider from a user.

#### `DELETE: /v1/account/providers/{PROVIDER}/{UID}`
#### `DELTE: /v1/users/{ID}/providers/{PROVIDER}/{UID}`

This endpoint removes single provider from a user. This will only work if there will always be a provider left after deletion.

### Adapters

This section will cover implmenting multiple adapters. For this we usually need following attributes and methods:

#### Attributes
- `provider` - Provider
- `endpoint` - API Endpoint
- `secret` - API Secret

#### Methods
- `send(number, text)` - Send SMS

Possible providers for Adapters are [Twilio](https://twilio.com/) or [MessageBird](https://messagebird.com/).

The implementation itself will be quite simple, since the API we are using usually only needs a single API call using an API key.

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

The change regarding the providers on the user object, will change the way how sessions will be created. Since now we need to parse the whole `providers` attribute for an E-Mail address instead of a single `email` attribute.

But this will allow user to be completely de-attached from an e-mail address.

This would also require a refactor on how anonymous accounts are created.

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

#### Are the required credentials consitent across multiple Third Party Providers?

#### How de we add new providers to a user?

This could potentially be solved with the already existing authentication methods, but when a user is already logged in - will append the used provider to the current user.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

This implementation will allow more implementation in the future and also decouple the user account from an E-Mail address.
