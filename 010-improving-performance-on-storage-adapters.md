# Improving performance on storage adapters <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 02-02-2021
- Target Date: N/A
- Appwrite Issue: N/A

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
In order to improve the performance while using storage adapters like S3 and DoSpaces, below propsed changes to the implementation to the storage functions on the adapter and implementation of storage api in Appwrite is required. Below proposed changes will make sure that each storage api functions will only make minimum requests to the storage services.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->
Right now, the storage api is making multiple requests to the storage services during file creation, preview, view or download. 
1. create file is making 4 calls (1 upload, 2. get mime type (for this we can use local device with tmp file), 3. read and 4. encrypt and write) => Not sure why we first upload, then read, then encrypt/compress and then write
2. Preview, View and Download makes 2 calls -> 1 for exists and 1 for read

So, we want to reduce these number of calls to improve performance.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

Previously the Appwrite only supported Local storage device and making multiple request was not a problem as everything was local. However with support for external storage services, making multiple requests will significantly increase our api response time  causing a performance nightmare. 


**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

The system will not have any changes that are visible to the users, however, internally the implementation of Storage API in Appwrite as well as storage adapters in utopia-php/storage will change significantly.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### For create file api
We are first uploading, then getting the mime type, then reading and compressing and then again writing. We can reduce these steps, 
1. get mime type from temporary file locally
2. encrypt/compress from local storage
3. Upload

This will make only one external request to upload the file.

### For Preview/View/Download
We are making first request to make sure file exists, then second request to read the file. We can reduce these by
1. The read method throws exception with code 404 if file doesn't exist
2. In storage API we simply call read and handle the exception instead of first checking it exists then calling read.

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
