# BackBlaze Storage Support <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: [@everly-gif](https://github.com/everly-gif)
- Start Date: 22/12/2021
- Target Date: 07/04/2022
- Appwrite Issue: NA
- LinkedIn : [Everly Precia Suresh](https://www.linkedin.com/in/everly-precia-suresh-196bba1b7/)
- Resume/CV : [View Here]()

## Summary
As part of the ongoing progress of implementing multiple storage support to make appwrite agnostic, I propose to implement a BackBlaze storage adapter.
BackBlaze B2 has been a go to alternative for AWS S3, among users. By implementing this adapter, the user will now be able to choose to store their files in Backblaze B2.

[summary]: #summary 

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

By default, Appwrite uses a docker volume to store the user's files which restricts the user to only one choice of storage.
Therefore, a need for storage adpaters to support multiple storage services is required. 
By implementing a backblaze storage adapter , the users will now have a choice to choose from a different storage service making the platform agnostic.

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

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
