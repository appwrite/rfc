# JWT Support for Authentication <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux
- Start Date: 26-12-2021
- Target Date: Unkwnon
- Appwrite Issue:
  https://github.com/appwrite/appwrite/issues/511

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

JSON Web Tokens are an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties. By adding support for JWT authentication, Appwrite developers will be able to use APIs that are currently restricted to client side only on the server as well and create interesting new use-cases for Appwrite.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

Appwrite Server API is limited and can't access APIs in ac user scope. The only current way to use the server API is in admin mode using an API key with relevant permissions scopes.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

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
