# Utopia ACME Client <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @irsayvid
- Start Date: 18-12-2021
- Target Date: 18-03-2022
- Appwrite Issue: NA

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Create an Utopia Automatic Certificate Management Environment (ACME) client using let's encrypt as CA (Certificate Authority) for domain validation. Having an own client ensures appwrite is dependency free. Through CLI, users can request, renew and revoke SSL certificates provided by Let's Encrypt.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

One of the services that Appwrite uses is the popular "certbot" tool under the hood to automatically handle SSL certificate generation and renewal, so developers can focus on building their apps. The SSL certificates are typically used to provide HTTPS for web servers but can be used in any situation where domain name validation is required, such as when securing our communication with Appwrite API.

In our effort to make Appwrite as dependency-free as possible, we want to create our own Automatic Certificate Management Environment (ACME) client to create and maintain SSL certificates within the Utopia Framework.

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

POST /v1/accounts.php - to store email of domain owner for verification and communications
POST /v1/generateNew.php (domain name) - generate new private public pair keys and create CSR
POST /v1/validate.php - Validate the domain user and install the certificate
POST /v1/redirect.php - change route traffic from HTTP to HTTPS on validation
POST /v1/renew.php - Renew the SSL certificate
GET /v1/getDetails.php - Get SSL certificate details for that domain

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
