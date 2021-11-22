# New Functions Runtime for Refactored Execution Model <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 10-08-2021
- Target Date: N/A
- Appwrite Issue:
  [Is this RFC inspired by an issue in appwrite](https://github.com/appwrite/appwrite/issues/)

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
We are completely redesigning our Cloud Functions Runtime environments this time using web server.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

This change is in the process to make Appwrite Cloud functions more efficient as well as allow synchronous execution of Functions. This should also reduce the function's execution time in subsequent execution by a lot.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

While working with Appwrite Cloud Functions at the moment, the execution time are notable. Everytime the code has to be executed, the runtime loads and executes the code file using Cloud Function Runtime's docker container. There is an overhead of container starting and executing.

Also at the moment, there is no way to make the Function's execution synchronous.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

Well, the process of executing Cloud Functions will completely change from Docker based execution to web requests. So the Functions worker will have to be modified accordingly to work with new function's runtimes.

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

The new Functions runtime will be developed as a web server. A web server that will receive a POST request to execute a code, then executes user code and returns the result. Being a web server, the request and response can happen synchronously, so that we can achieve Synchronous function execution.

### How will the Runtime look

Runtime will be have a web server, that will execute the functions. The functions during execution will receive a request and a response object as defined below.

#### Request Object

Request object will have,
1. Headers - A key value pairs of headers
2. Payload - A key value pairs of Payload data
3. Env - A key value pairs of Environment vairables that are available to the functions

#### Response Object

Response object will have,
   1. send method - will accept a string data and integer status code
   2. json method - will accept json data and integer status code

The response object will be used the user's functions to send back the response data.

### How will execution occur

The executor will build runtimes servers along with user code using some pre defined build scripts for each tag that gets activated and save the information regarding the container that is built for the tag. The executor will using another launch script to launch the build user code using the runtime container.

For languages that are compiled like Rust and Dart, the plain alpine or Ubuntu runtime will be used to launch the runtime server once it's built. 

For languages that don't build like Python and Javascript, same container is used to build and launch the user's code. The build and launch script has to be pre-defined for each language-runtime we support.

User function should either contain the pre-defined function signature for the runtime as defined by the runtime or it should export a callback function with proper signature that runtime can import and execute.

The user function will always receive request and response object. Request object will contain any values that is available to the function's execution. And users can use response object to send response back.

### How will the Runtime Web Server Look

The runtime web server will be a simple HTTP server running on port 3000, that accepts POST request with some pre-defined header in order to validate the request is from authorized source. Upon receiving the request, it executes the users code passing the received request object as well as newly formed response object with the structure defined above. Once executed it should receive response from users code otherwise it will throw an error.
The Runtime server will not be exposed publicly, the executor will use the docker internal DNS to send request to runtime servers.

Some languages like Dart, will require a package that defines the types for request and response that we will be using on our runtimes so that user's writing the Dart functions will be able to import it for better IDE support and better error handling.

### Building Runtimes for Various Languages

1. NodeJS
   - use micro, express or fastify to implement the server
2. Deno
   - Use the native HTTP server
3. Dart
   - Use the shelf package
4. PHP
   - use swoole
5. Rust
6. Python
   - use flask
7. Ruby
   - https://dev.to/leandronsp/web-basics-a-simple-http-server-in-ruby-2jj4
8. Kotlin
9.  .Net
10. Swift
11. Go

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

Fission functions are already using similar technology in order to build their runtimes for their Functions As a Service product. We can check out their runtimes at their [GitHub repository](- https://github.com/fission/environments)

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
