# Add Custom Exceptions on All SDKs <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 14-02-2021
- Target Date: N/A
- Appwrite Issue:
  https://github.com/appwrite/sdk-for-flutter/issues/13

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
All Appwrite SDKs instead of exposing the internal details via existing exceptions, we want to throw custom AppwriteException. This will make it a lot easier for anyone using the SDK to understand the errors as all SDKs will have similar error response, as they all have same success response.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->
Right now, whenever there's error on Appwrite SDK, each throw platform exceptions. Each are different based on the platform. There should be unified experience in Appwrite SDK in whichever platforms we use. Introducing AppwriteException will resolve this issue.


**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

We have multiple platform SDK each provide error information in different way. So we want to introduce unified way to throw errors accross multiple SDKs

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

All the SDKs will implement new AppwriteException class and throw AppwriteException with error details for all kinds of errors.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

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

## AppwriteException Class
In each SDK implement AppwriteException class that should provide following details
1. Message string
2. Error code (mostly http status code)
3. Appwrite server response data

## Handle Errors
Every server call in Appwrite SDKs should handle error cases and throw AppwriteException with appropriate details.

## Exception Information in Docs
The information regarding exceptions thrown, should be in the Appwrite SDKs docs for each SDK. Most SDKs have doc comments where we could include these or can create separate DOCs that will be generated for the SDK. The Appwrite DOCs should also include information about these errors with details on what information are returned, like the response value in the docs. The DOCs should include
* What exceptions can a method throw
* What are possible error codes and error messages
* What do they mean, and how to debug
* The exception information should be like response information in the DOCs, should include what structure and fields are available

## Error information in Swagger
The error info in Swagger spec will be under responses and can look like this
```json
{
  "responses": {
    "200": {
      "description":"This is success response"      
    },
    "400": {
      "description": "Bad Request. User ID must be provided",
      "schema": {
        "$ref":"#definitions/bad_request"
      }
    }
  }
}
```


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

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
