# Function Execution Model Rewrite
- Implementation Owner: @PineappleIOnic
- Start Date: 25-07-2021
- Target Date: TBD

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
This RFC contains a proposal for rewriting Cloud Functions in a way that will enable us to perform synchronous executions. This will introduce a new server into Appwrite specifically for dealing with executions called the "executor". The executor will deal with spinning up docker containers for runtimes that are required. When API requests for functions are received the executor will forward them to the relevant runtime which will then handle it.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Cloud Functions currently do not support the ability to execute synchronous functions and will require a complete rethink to how we approach how we execute our functions in order for us to realize this goal.

**What is the context or background in which this problem exists?**

Cloud Functions are currently a bit limited, this new design idea will allow for more customizability and control of how functions will execute and run for users.

**Once the proposal is implemented, how will the system change?**

- A new process will be introduced to execute functions
- All current runtimes will be updated to use a web server as it's core
- The Functions worker will be stripped of all direct docker interactions and will instead call the executor
- Functions will now have a build stage which will allow for dependencies for user functions to be fetched and for compiled languages to be supported.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

With this redesign of how cloud functions work the functions worker will no longer have to deal with spinning up docker containers but will instead use the executor server. Which will handle spinning up containers on behalf of the functions worker.

The executor will have multiple tasks but the main one is to spin up runtimes when they are needed. Runtimes will now be web servers based of their relevant languages. A new runtime will be launched for each tag that is executed.

The executor will be built using a Swoole HTTP Server and will be stored in the file: `app/executor.php`. The executor will need to be on the same network as appwrite so the two can communicate. When a API call for the executor is recieved by appwrite it will simply forward it to the executor.

The functions worker will also be able to directly make API Calls to the executor server itself due to being on the same network.

#### API Endpoints
Most API endpoints externally will stay the same apart from adding a `async` parameter to the `Create Execution` endpoint.

The `async` parameter will be by default set to `1`. If it is set to `0` the request will not be forwarded to the functions worker but will instead be directly communicated from appwrite to the executor. The result of the function will be returned directly as a result of the request in the following schema:
```json
{
    "status": "completed",
    "response": "Hello World!",
    "time": 0.0001890380324243
}
```

The Create Function Tag endpoint will also be updated to change the `command` parameter to be called `entrypoint` to better show that the user will no longer be putting the entire launch command for a function but instead only the file name of their custom function.

Internally however the executor has 5 different endpoints.
These endpoints are as follows:

`/v1/execute` - This is the main endpoint used to execute functions. It replaces the execute() function in the functions worker instead making it a HTTP Call.

`/v1/cleanup/function` - This endpoint is used internally by appwrite mainly when a function is deleted. It will cleanup all runtimes and tags associated with the function.

`/v1/cleanup/tag` - This endpoint does the same thing as `/v1/cleanup/function` but only cleans up that one tag.

`/v1/tag` - This gets called when a tag is selected. It will build a tag and launch it's runtime ready for execution.

`/v1/healthz` - A basic sanity check for the executor. Not currently used within appwrite but useful for debugging and testing.

#### Security
As an added security precaustion to prevent runtimes from communicating with other runtimes each runtime is given a secret that is stored within a swoole table on the executor. This secret is also known to the runtimes themselves using the `APPWRITE_INTERNAL_RUNTIME_KEY` environment variable. This secret is passed to all requests from the executor to the runtimes using a header called `x-internal-challenge` this is verified by the runtime. If the secret is not correct a 401 will be returned and the function will not execute.

Communication between appwrite and the executor is also protected using a environment variable key similarly to the runtimes however this environment variable key is set by the user. This is to prevent runtimes from accessing the executor directly and requesting executions that way.

#### Signal Handler
The executor also listens for the Unix `SIGINT`, `SIGQUIT`, `SIGKILL` and `SIGTERM` signals. If recieved the executor will remove all runtime containers, fail all executions with a message saying that the executor was shutdown during execution and cleanly exit.


#### Flowchart Visualisation:

![Flowchart Visualisation](flowchart.svg)

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

#### Documentation
Documentation will need to be updated to explain how syncronous execution works and how to use it.
Documentation will also need to be updated to explain the unique quirks of each language since each of them have different ways of handling dependencies for example or how the code should be exported.

### Prior art

[prior-art]: #prior-art

https://docs.fission.io/docs/architecture/

### Unresolved questions

[unresolved-questions]: #unresolved-questions
 
 - Discuss monitoring health of containers
 - Discuss horizontal scaling of containers
 - Using things such as [Amazon Firecracker](https://firecracker-microvm.github.io/)

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

