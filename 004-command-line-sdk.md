# Command Line SDK <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux
- Start Date: 30-12-2020
- Target Date: Unknown
- Appwrite Issue:
  https://github.com/appwrite/appwrite/issues/540

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Build an Appwrite CLI that will include all standard backend SDKs functionality and allow Appwriters to interact with the Appwrite HTTP API directly from their terminals.

The CLI should rely on common Appwrite CLI FW for consistency and easy maintenance. We should package the CLI as a Docker image for ease of use across different platforms or operating systems.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

There is no easy way to interact with Appwrite directly from the operating system terminal.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

Using Appwrite directly from the terminal can be extremely useful for integrating with CI and automation in general. With the introduction of Cloud Functions, a CLI integration would allow reducing crucial steps required to package cloud functions and upload them to the Appwrite server directly from the developer workspace.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

No changes are expected to the Appwrite server or API. The new CLI will perform exactly the same as every other Appwrite SDK.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Create a Prototype

Build a working prototype of the SDK using the team common tools-stack. Once ready, integrate the mock into the Appwrite [SDK generator](https://github.com/appwrite/sdk-generator) for easy syncing with any future API changes and an automated release cycle. The new CLI tool should comply with all the Appwrite SDK generator [requirements](https://github.com/appwrite/sdk-generator/blob/master/CONTRIBUTING.md#sdk-checklist) for consistency with other Appwrite SDKs.

#### Suggested file structure:

Dockerfile
composer.json
app/
  client.php
  services/
    users.php
    storage.php
    ...

To start, we can skip any input validation in the CLI layer and rely on the HTTP validations and status codes. All user input for the HTTP client settings (API Keys, Project ID, Endpoint URL) should be saved locally on the container for persistency and ease of use.

**Usage Examples:**
```bash
docker run appwrite/cli:1.0.0 client setProject --value=5e63e0a61d9c2
```

```bash
docker run appwrite/cli:1.0.0 users createUser --email=test@appwrite.io --password=secret --name=Demo User
```

**References**:
- PHP-SDK: https://github.com/appwrite/sdk-for-php/blob/master/src/Appwrite/Client.php
- Node-SDK: https://github.com/appwrite/sdk-for-node
- Utopia CLI: https://github.com/utopia-php/cli

### Update Documentation

List the new Appwrite CLI in our `/docs/sdks` page. Add code examples for using the new CLI SDK in the Appwrite server API docs alongside existing server languages. 

### Tutorial

Create a short tutorial that explains how to use the new SDK in a specific common use case.

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

Ensure that you include examples, code-snippets, etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal. A
few examples of what this can include are:

- Does this functionality exist in other software, and what experience has their
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

It would be nice to find a way to use the CLI without the Docker prefix. For example: 

**Usage Examples:**
```bash
appwrite client setProject --value=5e63e0a61d9c2
```

```bash
appwrite users createUser --email=test@appwrite.io --password=secret --name=Demo User
```

**Not crucial for the first release**

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

We could use the new Appwrite CLI as a legit SDK, just like any other official Appwrite SDK. The CLI would also make integration with other upcoming features much easier.
