# Cross System GraphQL API <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux, @christyjacob4, @abnegate
- Start Date: 10-01-2021
- Target Date: Unknown
- Appwrite Issue:
  * https://github.com/appwrite/appwrite/issues/272
  * https://github.com/appwrite/appwrite/issues/511

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL consumes a complete and understandable description of the models in your API, then gives clients the power to ask for **exactly** what they need from each model and nothing more. This means app developers can control the volume of data coming into their applications, and server developers have an easier time expanding their products.

### GraphQL vs REST API

A REST API is an architectural concept for network-based software. GraphQL, on the other hand, is a query language, a specification, and a set of tools that operates over a single endpoint using HTTP. Over the last few years, REST has continued to be used to make new APIs, while the focus of GraphQL has been to optimize for performance and flexibility.

While the Appwrite REST API is an easy way to get started with Appwrite quickly, GraphQL will allow better performance for specific use-cases and allow new ways to integrate with Appwrite.

The new Appwrite GraphQL API is not designed or aimed to replace the REST API, but rather to complement it.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

While the Appwrite REST API is easy and simple to use, some developers may prefer to integrate their API and client applications using a GraphQL API due to its benefits in particular scenarios.

Additionally, developers using Appwrite should not be forced to integrate with the server using a specific protocol. Appwrite should be protocol agnostic and provide multiple API interfaces for different use-cases and ease of use for developers who come with particular preferences or existing knowledge.

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

### Containers

A new container will be added to the Appwrite network as `appwrite-graphql`.

### Entrypoints

Per convention there will be a new php entrypoint and a shell wrapper.

- php: `./app/graphql.php`
- shell: `./bin/graphql`

### Routing

Traefik will be used to route all requests with a path prefix `/v1/graphql` to the new GraphQL container.

### API Endpoints

<!--
List the new API routes or endpoints that we might need to add for supporting the new feature.
Keep in mind to stay very strict to the API protocol and method, whether your new
changes are for the REST, WebSocket or any other API protocol Appwrite supports.

For example:

**POST /v1/coffee ** - an endpoint for creating coffee.
**DELETE /v1/coffee ** - an endpoint for deleting coffee.
-->

**POST /v1/graphql** - an endpoint for querying and mutating Appwrite data with GraphQL. We will use Utopia PHP routes' object callbacks and metadata to init GraphQL queries and mutations. Per request, we will load the related projects collections and extend the schema built from internal models.

### Data Structure

<!--
What kind of changes or additions are required for the Appwrite base collections
to support this feature. Explain which entities should be added or updated, what new attributes they
need to have and why. Please think well about the naming conventions and how well they play with other
Appwrite conventions. Try and stay as consistent with existing patterns as much as possible.
-->

Each Appwrite route has a label with its method name (`->label('sdk.method', 'getPrefs')`). This label is being used to generate the REST API SDKs using the [SDK generator](https://github.com/appwrite/sdk-generator). We will preload all the routes' types, queries and mutations and sort them in a hash where the method name is the key for fast routing between the different GraphQL API actions.

**Query Example:**

```graphql
query accountGet { # [serviceName][ActionName]
  user { # Returned Model
    name
    email
  }
}
```

**Mutation Example:**

```graphql
mutation accountSessionCreate($email: string, $password: string) { # [ServiceName][ActionName]
  user { # Returned Model
    name
    email
  }
}
```

**Subscription Example:**

```graphql
subscription usersCreate { # [ServiceName][ActionName]
  user { # Returned Model
    name
    email
  }
}
```

### Async Queries

We can support async query resolvers using a [Promises/A+](https://github.com/promises-aplus/promises-spec) implementation. We provide an implementation of the required interface that leverages Swoole coroutines.

References
  - https://github.com/webonyx/graphql-php/blob/master/docs/data-fetching.md#async-php
  - https://github.com/streamcommon/promise
  
### Query Depth

Query depth is unlimited by default but this could introduce performance issues for complex requests. We will limit queries to a maximum depth of 10. By analyzing the query document’s abstract syntax tree (AST), the GraphQL server can reject or accept a request based on its depth.

Examples:
- Sangria: 7 levels https://sangria-graphql.github.io/learn/#limiting-query-depth
- Webonyx: 10 levels https://github.com/webonyx/graphql-php/blob/master/docs/security.md#limiting-query-depth
- Spring-boot: 10 levels https://github.com/graphql-java-kickstart/graphql-spring-boot/issues/569

### File Uploads

File uploads can be achieved by creating a middleware that allows requests with header `multipart/form-data` following a predefined spec to attach files to the request payload.

References
- https://github.com/Ecodev/graphql-upload
- https://github.com/jaydenseric/graphql-multipart-request-spec

### Error Handling

There are 3 types of errors in GraphQL:

- Syntax: query has invalid syntax and could not be parsed;
- Validation: query is incompatible with type system (e.g. unknown field is requested);
- Execution: occurs when some field resolver throws (or returns unexpected value).

Error response schema:

```php
<?php
[
    'message' => 'Error message',
    'extensions' => [
        'key' => 'value',
    ],
    'locations' => [
        ['line' => 1, 'column' => 2],
    ],
    'path' => [
        'listField',
        0,
        'fieldWithException',
    ],
];
```

### Supporting Libraries

<!--
Which different libraries do we need to support the new features?
Please describe the new library's potential API?
Avoid using 3rd party libraries when possible, if required - explain why.
-->

- [utopia-php/graphql](https://github.com/utopia-php/graphql) - To be completed
  - [webonyx/graphql-php](https://github.com/webonyx/graphql-php)

Utopia GraphQL library will provide an abstraction layer around GraphQL parser/executors. Currently [webonyx/graphql-php](https://github.com/webonyx/graphql-php) would be the only supported adapter.

### Security

#### Abuse Control

We should implement an abuse control check to prevent people from abusing the new endpoint, which technically also acts as a validator for JWT tokens. This will completely prevent any large-scale attack from guessing tokens or abuse of the server.

#### CORS Validation

On each request, we should validate that the client origin is valid similarly to what we do in the [REST API](https://github.com/appwrite/appwrite/blob/master/app/controllers/general.php#L131-L139). This is a good security practice to avoid project data being presented on un-authorized clients. This will force devs to list their platforms before using the GraphQL API.

#### Cookie Authentication

On HTTP Handshakes, cookies are usually sent along, which we can use for authentication. If a technology does not provide a native handshake - we can imitate this functionality to handle passing authentication tokens to the endpoint.

#### JWT Authentication

Using JWT authentication can be easier to implement and pass to the server.


#### Scaling

<!-- Explain how we will scale this new feature. -->

As an individual container it can be duplicated and load-balanced with Traefik.

#### Benchmarks

<!-- Explain how we will benchmark the new feature. -->

Comparing test execution times for the same requests using the REST API vs the GraphQL API.

#### Tests (UI, Unit, E2E)

<!-- 
Explain how we will test the new feature. 
You can use "N/A" if this section is not relevant to your proposal.
-->

To fully test the integration we would need to create a test suite for both Client and Server authentication methods similar to what we're doing in our REST API. We can use [mghoneimy/php-graphql-client](https://github.com/mghoneimy/php-graphql-client) as a **dev dependency** for sending GraphQL API calls. Create some tests that are testing for system failure and authentication integrity.

### Documentation & Content

<!--

Documentation is vital for making this new feature a success for both developers using Appwrite and the Appwrite maintainers.
Please answer the following questions:

1. What **docs** would support this feature?
2. Do we need to update the **contribution guide** with a new section or a supporting tutorial?
3. What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?
4. What **demo applications** can help us demonstrate this feature APIs and capabilities? 

-->

- Create a new dedicated documentation page for using the GraphQL service. List all the different authentication methods, actions available, and response models.
- Create a short tutorial that explains how to use the new service in common use-cases.
- Create an in-depth tutorial that explains how to use the new service.
- Create an article that discusses GraphQL and REST
- Create demo client applications

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
  you have some relevant papers to refer to, and this can serve as a more detailed
  theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other software.

Write your answer below.
-->

**References**:
- https://shopify.dev/tools/graphiql-admin-api
- https://developer.github.com/v4/explorer/
- http://localhost:9505 (Appwrite instance of GraphiQL available [here](https://github.com/appwrite/appwrite/blob/1349a1a8b594ff132ae6e9e9d246d856a949733d/docker-compose.yml#L493))
- https://tech.okcupid.com/the-shadow-request/
- https://tech.okcupid.com/moving-okcupid-from-rest-to-graphql/

Public GraphQL APIs:
- [Shopify](https://shopify.dev/docs/admin-api/graphql/reference)
- [GitHub](https://docs.github.com/en/free-pro-team@latest/graphql)

**An Appwrite POC available on the `graphql` branch:**
https://github.com/appwrite/appwrite/compare/0.7.x...graphql?expand=1

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### SDK Use

How should the SDK's allow GraohQL use? 
  
Options
- Global `UseGraphQL` flag
- Per service request flag

### Subscriptions:
  - Read-only
  - Can be provided via WebSockets
  - Suggested lifecycle:
    - https://github.com/apollographql/subscriptions-transport-ws/blob/master/src/message-types.ts
  - Build a subscription type for each possible realtime event
  - Create a subscription resolver function that accepts a realtime callback and returns a graphql response
  - Subscribe to realtime passing resolver function as callback
  - https://siler.leocavalcante.dev/graphql#graphql-subscriptions
  - https://github.com/leocavalcante/siler

- Notes:
  - Lazy loading types: https://github.com/webonyx/graphql-php/blob/master/docs/schema-definition.md#lazy-loading-of-types
  - Resolver per type instead of field: https://github.com/webonyx/graphql-php/blob/master/docs/data-fetching.md#default-field-resolver-per-type

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

- Add a function to the SDK generator to enable using the GraphQL API globally instead of using the REST API.
- Add a parameter to the SDK generator service functions to enable using the GraphQL API instead of using the REST API per request.