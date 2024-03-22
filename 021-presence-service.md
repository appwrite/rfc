# Presence service

* Creator: Eldad Fux
* Relevant Issues:  

- [RFC Template](#rfc-template)
  - [Summary](#summary)
  - [Resources](#resources)
  - [Implementation](#implementation)
    - [API Changes](#api-changes)
    - [Workers / Commands](#workers--commands)
    - [Supporting Libraries](#supporting-libraries)
    - [Data Structures](#data-structures)
    - [SDKs](#sdks)
    - [Breaking Changes](#breaking-changes)
    - [Documentation & Content](#documentation--content)
  - [Reliability](#reliability)
    - [Security](#security)
    - [Scaling](#scaling)
    - [Benchmarks](#benchmarks)
    - [Tests (UI, Unit, E2E)](#tests-ui-unit-e2e)
  - [Open Questions](#open-questions)
  - [Future Possibilities](#future-possibilities)

## Summary
<!-- Describe the problem we want to solve and suggest a solution in a few paragraphs -->

In many use cases where we build an application that requires realtime collaboration, we need to know which of our users are online and what their status is. Today, this is possible in Appwrite only by building a complex custom integration. This can be relevant to collaborative apps like Slack, Google Docs, or multiplayer gaming apps. 

To reduce friction, make this a breeze, and enable new use cases for using Appwrite, we want to introduce a new presence service that will also be integrated with the Appwrite realtime server REST and GraphQL APIs and will allow each user to set strict privacy permissions for who can see their current presence and what is their current status.

## Resources
<!-- List all the resources used for writing this RFC -->

## Implementation

<!-- Write an overview to explain the suggested implementation -->

### API Changes
<!-- Do we need new API endpoints? List and describe them and their API signatures -->

```POST /v1/presences```

Create a presence. Set the user presence status, expiry (minutes/date - TBD), and permissions for who has access to see the current presence. This will imitate the equivalent action in the GraphQL and Realtime APIs.

```GET /v1/presences```

List all presences, on client side this will show only presneces I have permission to view. On server it will show all presences. This endpoint will support queries and pagination like any other list endpoint on Appwrite.

```GET /v1/presences/:id```

Get presence by ID.

```PUT /v1/presences/:id```

Update presnece by ID.

```DELETE /v1/presences/:id```

Delete presence by ID.

### Events

The new endpoints will trigger standard events that will be used for listening to Realtime API changes or to execute functions and webhooks.

### Console updates

- We should implement presence in the console itself and make current org members online or not or last online
- We will add a new section under auth called `Presences`, this section will show a dashboard with usage showing how many people were online in a specific time range. Below that, we will show avatars of all currently online users or users who just recently went offline. 

###  Workers / Commands
<!-- Do we need new workers or commands for this feature? List and describe them and their API signatures -->

- The maintenance worker will need to clean all present documents that have expired. We should have a specific grace period that will allow us also to show people who just recently went offline. So if the presence is valid for 5 minutes, we will delete it after 5+ X minutes. X will be decided later. 
- Usage service should count how many people were online at a specific time. 

###  Supporting Libraries
<!-- Do we need new libraries for this feature? Mention which, define the file structure and different interfaces -->

Utopia/databases will need to expose Group by functionality so we can fetch only a unique list of presences per user as a user is allowed to report it multiple times to multiple groups. The most recent presence will take priority in case multiple presences allow a different user to view them. 

### Data Structures
<!-- Do we need new data structures for this feature? Describe and explain the new collections and attributes -->

We need to add a new collection to store presences.

- _id
- _uid
- _createdAt
- _updatedAt
- userInternalId
- userId
- expiry (DateTime)
- status (string)
- source (string - REST / GraphQL / Realtime)
- hostname (used only for realtime to clean old presences in case of server failover)

### SDKs
<!-- Do we need to update our SDKs for this feature? Describe how -->

SDKs will generate standard REST methods to support the new APIs. For realtime, we will need to add new parameters when establishing a new connection to enable the creation of a presence if a user chooses to create one. 

#### Realtime

The presence will be created if a user set its status and permissions. The realtime server will delete the presence when the connection disconnect. For cases where the server might fail, every startup will delete all the previous presence reported by the current hostname.

```js
import { Client } from "appwrite";

const client = new Client()
    .setEndpoint('https://cloud.appwrite.io/v1')
    .setProject('<PROJECT_ID>');

// Subscribe to files channel
client.subscribe([CHANNELS], status, [PERMISSIONS], function() {
    console.log(response.payload);
});

// TODO consider sending the presence as a message to keep the connection creation simpler/focused/cleaner. 

```

### Breaking Changes
<!-- Will this feature introduce any breaking changes? How can we achieve backward compatibility -->

No breaking changes are planned on Appwrite side. Realtime connection signature might change and force major SDK releases.

### Documentation & Content
<!-- What documentation do we need to update or add for this feature? -->

## Reliability

### Security
<!-- How will we secure this feature? -->

The same security practices we take for other APIs and services will take place.

### Scaling
<!-- How will we scale this feature? -->

The same scalability practices we take for other APIs and services will take place.

### Benchmarks
<!-- How will we benchmark this feature? -->

We can run a few benchmarks and test how the service performs on large amounts of presences. Memory tests should also be performed against the realtime server.

### Tests (UI, Unit, E2E)
<!-- How will we test this feature? -->

Standard unit and E2E tests.

## Open Questions
<!-- List of things we need to figure out or further discuss -->

Should we be allowed to attach metadata with each presence update, and how should we handle the nested data structure? Should it work like user/team prefs?

## Future Possibilities
<!-- List of things we could do in the future to extend or take advantage due to this new feature -->
