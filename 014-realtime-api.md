# Cross System Realtime API <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux @TorstenDittmann
- Start Date: 21-02-2021
- Target Date: Unknown
- Appwrite Issue:
  [Is this RFC inspired by an issue in appwrite](https://github.com/appwrite/appwrite/issues/)
  - https://github.com/appwrite/appwrite/pull/692

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
We are adding a real-time API for the Appwrite server, which allows streaming and listening of all the Appwrite [system events](https://appwrite.io/docs/webhooks#events). The new real-time API will allow new and interesting use cases of building an app with Appwrite from simple chat apps, multiplayer games, live collaboration to real-time dashboards and enhanced UIs.

Unlike other BaaS products, Appwrite real-time should be completely cross-platform and work well from both client or backend. The new API should also be completely decoupled from the Appwrite database and transmit any supported system events that Appwrite produces, including storage, users, account events, and more.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

The Appwrite current REST API, and upcoming GraphQL are both HTTP based which is not suitable for realtime applications. HTTP classis request-response communication is not very well suited for listening and reporting of events.

Currently the only way to build realtime, events based application with Appwrite is by using a 3rd party server which is hard and complicated to implement espcially when data segmentatation and ACL is a requirement. An current alternative to using a 3rd party service, is to use HTTP-Polling which is costly and not optimized for performance.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

Allowing new use-cases for Appwrite, and giving more flexibily for end-developers relaying on Appwrite for user authentication and data proccessing.

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
### Architecture


### Protocols Support



- Websocket support: https://www.swoole.co.uk/docs/modules/swoole-websocket-server
- MQTT support: https://www.swoole.co.uk/docs/modules/swoole-mqtt-server
- SSE support: https://github.com/hhxsv5/php-sse
- Socket.io support: https://github.com/shuixn/socket.io-swoole-server

### Security

#### Abuse Control

x mesages per connection

#### CORS Validation

#### Limit payload size

#### JWT Authentication (in path / or in message)

Cookies support in webscoket and other protocols is limited.

### Logs

We should add the following logs for easy debugging and monitoring of our realtime server. Below is a list of some of the logs we could initialy add:

- Server start (stdout - using Console::success)
- Worker start  (stdout - using Console::success)
- New connection (total connections per worker)
- Errors (stderr - using Console::error)

Debug mode logs:

- Event received (stdout - using Console::log)
- Message sent (stdout - using Console::log)

### Scalability

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

#### 2-way Communication

#### Messaging Channel

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
