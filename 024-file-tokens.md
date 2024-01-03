# File tokens

- Implementation Owner: @lohanidamodar
- Start Date: 20-06-2023
- Target Date: N/A
- Appwrite Issue:
  [Is this RFC inspired by an issue in appwrite](https://github.com/appwrite/appwrite/issues/)

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Accessing files currently requires either files to be public or need user session with user that have access permission. It's mainly painful for image previews that we want to show publicly. For this reason, we want to introduce file tokens. The tokens will be linked to specific file and will be passed in URL for previews and downloads to provide access.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

Currently only way to access private files in Appwrite storage is by using a user session. However it's painful for image previews that we want to display on our applications. Also there are no safe ways to share file with external users or applications without making is completely public.

That is why we want to introduce file tokens. A file toke will be linked to files and manage it's own permissions over the resource as what kind of access the token will have to the file. Token can be passed as a header or request get parameter.

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

Tokens will be a part of storage service

**POST /v1/storage/buckets/:bucketId/files/:fileId/tokens** -           create token
**PUT /v1/storage/buckets/:bucketId/files/:fileId/tokens** -            update token
**GET /v1/storage/buckets/:bucketId/files/:fileId/tokens** -            List token
**GET /v1/storage/buckets/:bucketId/files/:fileId/tokens/:tokenId** -   Get token
**DELETE /v1/storage/buckets/:bucketId/files/:fileId/tokens/:tokenId** -         delete token


### Data Structure

<!--
What kind of changes or additions are required for the Appwrite base collections
to support this feature. Explain which entities should be added or updated, what new attributes they
need to have and why. Please think well about the naming conventions and how well they play with other
Appwrite conventions. Try and stay as consistent with existing patterns as much as possible.
-->

We need to introduce new collection to save tokens.

**resourceTokens**
  - resourceId - String
  - resourceInternalId - String
  - resourceType = String
  - secret - String
  - secretHash - String (md5 hash of secret for query)
  - expiryDate - date time

> Note: Resource ID / InternalId will need a structure to incorporate the parent resource as well. For example when resource is file we need both bucketId and fileId. We can use `:` as a separator to keep the resource Id in single field. So for the tokens for file the resource Id will be `bucketId:fileId` and similarly for a document it will be `databaseId:collectionId:documentId`

We will also need a token validator, that will validate that the token is not expired and has access to the resources.

### Supporting Libraries

<!--
Which different libraries do we need to support the new features?
Please describe the new library's potential API?
Avoid using 3rd party libraries when possible, if required - explain why.
-->

This particular feature doesn't require any supporting libraries.

### Breaking Changes

<!--
Do we break any API or SDK backward compatibility?
If possible, explain what actions we can take to avoid that.
-->

This shouldn't break any existing features

### Reliability (Tests & Benchmarks)

#### Scaling

<!-- Explain how we will scale this new feature. -->
This will be part of Appwrite core, which should scale along with the existing Appwrite infrastructure.

#### Benchmarks

<!-- Explain how we will benchmark the new feature. -->

We don't need separate banchmark as this is part of Appwrite core API

#### Tests (UI, Unit, E2E)

<!-- 
Explain how we will test the new feature. 
You can use "N/A" if this section is not relevant to your proposal.
-->

We will write relevant Unit and E2E test for this feature.

### Documentation & Content

<!--

Documentation is vital for making this new feature a success for both developers using Appwrite and the Appwrite maintainers.
Please answer the following questions:

1. What **docs** would support this feature?
2. Do we need to update the **contribution guide** with a new section or a supporting tutorial?
3. What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?
4. What **demo applications** can help us demonstrate this feature APIs and capabilities? 

-->

We need to update storage documentation. Specially on the file preview, view and downloads.

* It might be a good idea to create a separate guide, file tokens that provides guide on previewing, viewing and downloading files using the tokens

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

- File tokens are used by many similar services to provide file access.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

1. Token permission (read, write, update, delete), right now we only provide read permission

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

* Use token for other resources
* More granular permission control to allow read/create/update/delete permissions to make something like google docs possible
