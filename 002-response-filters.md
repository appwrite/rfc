# Response Filters <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux
- Start Date: 16-12-2020
- Target Date: 20-12-2020
- Appwrite Issue: None

## Summary

[summary]: #summary

Add a the option to enable different response filters to the Appwrite response object. This filters will act as a middleware and allow us to manipulate the response code before sending it to the HTTP client.

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

While in beta we are changing some of our APIs response formats.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->
Appwrite is still in beta, and as we progress with the project development, we need to break some existing APIs response format from time to time. Initially, this wasn't an issue because the product is still in beta, but it has become a requirement with growing usage by the developers' community.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

A new API option will allow devs to keep backward compatibility.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

We should introduce a new env variable and HTTP header that will allow to change the response format. This variable & header will allow devs to enable a response object filter which will parse the Appwrite response to continue support for old Appwrite response formats without breaking or changing our developers apps when upgrading an appwrite version.

### New ENV Var (_APP_SYSTEM_RESPONSE_FORMAT)

Should be checked on server startup. If the value is not empty we should init the relevant filter. If not filter is avaliable we should throw an error using the Console::error method and stop the proccess with exit(1);. Make sure to add the new var to Appwrite docs page: https://appwrite.io/docs/environment-variables#system-settings

### New Optional Header (X-Appwrite-Response-Format)

Should be checked on each request (general.php / ->init() can be a good location). If the value is not empty we should init the relevant filter. If not filter is avaliable we should throw an 404 HTTP error with relevant error message.

### Response Filters and Adapters

Added 2 new method to Appwrite response class (src/Appwrite/Utopia/Response.php) called `static function setFilter(Filter $filter): self` and `static function getFilter(): Filter`. The new method will set and get the new filter to a static property called `self::$filter`. Add a method called `static function isFilter(): bool` to check if a filter has been set or is null. 

### Create a Filter Interface

Create a new Appwrite\Utopia\Response\Filter class (src/Appwrite/Utopia/Response/Filter.php). The interface should decalre an implemntation for the `parse(array $content): array` method. Each new filter will implement the interface and use the parse method to manipulate incoming data. Create our first filter (Appwrite\Utopia\Response\Filters\V06) to accept current version (v0.7) data and convert it to v0.6 data. For that we'll need to run both instances of Appwrite to manually convert data structures.

### Run the Filter

Add a piece of code in the response output stage, that will check if a filter is set and if it is will execute his `parse` method and alter the response output before it is returned to the client. A good location for this method to run might be: https://github.com/appwrite/appwrite/blob/0.7.x/src/Appwrite/Utopia/Response.php#L296

### Tests

Add unit-tests for the new methods in the Response class, and for our first filter using mock data as input.

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

In the future we might want to do something similar to the Request input in case we'll introduce any API signature breaking changes.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

None.
