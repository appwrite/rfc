# GraphQL API <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @eldadfux
- Start Date: 10-01-2021
- Target Date: Unknown
- Appwrite Issue:
  * https://github.com/appwrite/appwrite/issues/272
  * https://github.com/appwrite/appwrite/issues/511

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

### GraphQL vs REST API

A REST API is an architectural concept for network-based software. GraphQL, on the other hand, is a query language, a specification, and a set of tools that operates over a single endpoint using HTTP. In addition, over the last few years, REST has been used to make new APIs, while the focus of GraphQL has been to optimize for performance and flexibility.

While Appwrite REST API is an easy way to get started with Appwrite quickly, GraphQL will allow better performance for specific use-cases and new ways to integrate Appwrite with platforms that we don't yet support that relies on that protocol. Ex: iOS, Android, or Gatsby.js.

The new Appwrite GraphQL API is not designed or aimed to replace the REST API, but rather to complete it and work well alongside it.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

While Appwrite REST API is easy and simple to use, a lot of developers would prefer to integrate their API and client applications using a GraphQL API. GraphQL can be a bit more complex but also more flexible and introduce better performance in some use-cases.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

Developers using Appwrite should not be forced to integrate with the server using a specific protocol. Appwrite should be protocol agnostic and provide multiple integrations API for different use-cases and ease of use for developers who come with particular perforations or existing knowledge.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

We'll introduce a new API route called POST /v1/graphql. The new endpoint will **parse**, and **route** GraphQL requests to existing Appwrite controllers and routes and return the same response models as the Appwrite REST API endpoints.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Create a New Route

Create a new route called POST /v1/graphql. The new endpoint will initialize the GraphQL parser [webonyx/graphql-php](https://github.com/webonyx/graphql-php). We should use Utopia PHP routes object callbacks and metadata to init GraphQL queries and mutations.

### Get Route by Method Name

Each Appwrite route has a label with its method name (`->label('sdk.method', 'getPrefs')`). This label was used only to generate the REST API SDKs using the [SDK generator](https://github.com/appwrite/sdk-generator). We should preload all the routes and sort them in a hash where the method name is the key for fast routing between the different GraphQL API actions.

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
mutation usersCreate { # [ServiceName][ActionName]
  user { # Returned Model
    name
    email
  }
}
```

### Load Routes, Models, and Collections

Load all Appwrite relevant routes, models, and Database collections into the GraphQL parser library. Any `GET` routes should be loaded as GraphQL queries, any `POST`, `PUT`, `PATCH`, or `DELETE` methods should be loaded as mutations. Both Appwrite Models objects and Database collections should be loaded as GraphQL objects. All the data is available using existing Appwrite libraries. Make sure to also include models for error objects ([dev](https://github.com/appwrite/appwrite/blob/764672e15e8c3a4c0c3891d620d293e1ead9045c/src/Appwrite/Utopia/Response/Model/Error.php) & [non-dev](https://github.com/appwrite/appwrite/blob/764672e15e8c3a4c0c3891d620d293e1ead9045c/src/Appwrite/Utopia/Response/Model/ErrorDev.php)).
### Authentication

We need to make sure authentication is enforced in the context of each specific GraphQL action. An endpoint that can only be used with an API key or can only be accessed by an authenticated user should act exactly the same for the GraphQL API. Because the GraphQL endpoint is basically just another public HTTP endpoint as part of the Appwrite HTTP server, it can grant access to all the different server actions.

To avoid this situation, we need to enforce scopes based security with an additional layer that will be implemented inside the GraphQL controller.

![Authentication Flow](https://raw.githubusercontent.com/appwrite/appwrite/1da4fa8168e9295282d8e8f0265f923c153b2a23/docs/specs/authentication.drawio.svg "Authentication Flow")

This section handles the current socpes + roles based authentication:
https://github.com/appwrite/appwrite/blob/f9afa2c95152b15eb079c1c65f249be4fe201c75/app/controllers/general.php#L164-L218

### Error Handling

TODO. Available resources:

- https://xuorig.medium.com/a-guide-to-graphql-errors-bb9ba9f15f85


**References**:
- https://shopify.dev/tools/graphiql-admin-api
- https://developer.github.com/v4/explorer/
- http://localhost:9505 (Appwrite instance of GraphiQL available [here](https://github.com/appwrite/appwrite/blob/1349a1a8b594ff132ae6e9e9d246d856a949733d/docker-compose.yml#L493))
- https://tech.okcupid.com/the-shadow-request/
- https://tech.okcupid.com/moving-okcupid-from-rest-to-graphql/

### Documentation

Create a new dedicated documentation page for using the GraphQL API. List all the different authentication methods, actions available, and response models.

### Tests

To fully test the integration we would need to create a test suite for both Client and Server authentication methods similar to what we're doing in our REST API. We can use [mghoneimy/php-graphql-client](https://github.com/mghoneimy/php-graphql-client) as a **dev dependency** for sending GraphQL API calls. Create some tests that are testing for system failure and authentication integritiy.

### Tutorial

Create a short tutorial that explains how to use the new API in common use-cases.

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

Ensure that you include examples, code-snippets, etc., to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->

### Prior art

[prior-art]: #prior-art

Public GraphQL APIs:
- [Shopify](https://shopify.dev/docs/admin-api/graphql/reference)
- [GitHub](https://docs.github.com/en/free-pro-team@latest/graphql)

**An Appwrite POC available on the `graphql` branch:**
https://github.com/appwrite/appwrite/compare/0.7.x...graphql?expand=1

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

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

We should find an alternative way to handle the custom JSON object that we currently use in the REST API. One example is the user prefs key-value object, which doesn't have a predefined structure. Another is the Function objects `vars` attribute that can hold a custom key-value object.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

The new API will allow integration with more GraphQL supported tools.