# Offline Support

* Creator: Eldad Fux
* Relevant Issues:  https://github.com/appwrite/appwrite/issues/1168

- [Offline Support](#offline-support)
  - [Summary](#summary)
  - [Implementation](#implementation)
    - [Cache](#cache)
    - [Network Status](#network-status)
    - [Write Queue](#write-queue)
    - [Write Conflicts](#write-conflicts)
    - [Promise Resolution](#promise-resolution)
    - [API Changes](#api-changes)
    - [Workers / Commands](#workers--commands)
    - [Supporting Libraries](#supporting-libraries)
    - [Data Structures](#data-structures)
    - [SDKs](#sdks)
    - [Breaking Changes](#breaking-changes)
    - [Documentation \& Content](#documentation--content)
  - [Reliability](#reliability)
    - [Security](#security)
    - [Scaling](#scaling)
    - [Benchmarks](#benchmarks)
    - [Tests (UI, Unit, E2E)](#tests-ui-unit-e2e)
  - [Open Questions](#open-questions)
  - [Future Possibilities](#future-possibilities)

## Summary
<!-- Describe the problem we want to solve and suggested solution in a few paragraphs -->

Offline support in Appwrite can help users interact with applications even when there is no network connectivity. This RFC explains how we could potentially have offline support implemented in the different Appwrite SDKs and APIs.

## Implementation
<!-- Write an overview to explain the suggested implementation -->

The following steps describe how offline support could be implemented in the different Appwrite SDKs in collaboration with the Appwrite API for conflict resolution.

For details on the SDK implementation, see the [SDK Design](./sdk-design.md) document.

### Cache

Cache every read response (GET method and content-type is text). Avoid caching any images or files. The cache should have an [LRU](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU) implementation ([see example](https://blog.devgenius.io/implementing-lru-cache-in-php-1632cf6a7443)) + max cache size setting. We will need to discuss the storage engine for each platform, but our preference would be to use native capabilities and avoid added dependencies as much as reasnoably possible.

### Network Status

Automatically update online / offline status. When online execute all the write calls that have been stacked in the queue.

### Write Queue

Create a queue for all write requests (POST / PUT / PATCH / DELETE) and store their timestamp. Once we go back online start submitting all the requests with thier original timestamp and the added header `X-Appwrite-Timestamp`. 

### Write Conflicts

Avoid race conditions, if the request was meant to be sent at X time make sure the last updated time on the server is older. We will use the new `X-Appwrite-Timestamp` header to let the database library know what was the original time the document update was requested. If the document on the server has a more recent update than the one we sent with the header, an exception will be thrown. Every exception will be translated to 409 HTTP conflict error.

### Promise Resolution

Promises / Futures should be resolved only after the request was accepted by the server. If we're offline, the promise should not be resolved. This is so that if there's an error on the server, the client can handle that accordingly.

### API Changes
<!-- Do we need new API endpoints? List and describe them and their API signatures -->

The Appwrite API will acceprt the new `X-Appwrite-Timestamp` header. The header will be used to initalize the relevant database instance. We need to figure whether a similar approach should also be taken for the different workers.

### Workers / Commands
<!-- Do we need new workers or commands for this feature? List and describe them and their API signatures -->

No extra workers or server commands are required.

### Supporting Libraries
<!-- Do we need new libraries for this feature? Mention which, define the file structure, and different interfaces -->

Utopia/Database will expose a new timestamp attribute that will help us to determine any conflicts when updating documents. If there's a conflict, a new `Conflict` exception will be thrown. The Appwrite API will catch the new exception in relevant use-cases and translate it to the proper 409 conflict HTTP error code. 

```php
$database->withRequestTimestamp($requestTimestamp, $callback);
```

### Data Structures
<!-- Do we need new data structures for this feature? Describe and explain the new collections and attributes -->

No data structures changes will be required.

### SDKs
<!-- Do we need to update our SDKs for this feature? Describe how -->

The SDK will publicly expose three new methods. `setOfflinePersistency` will enable or disable offline support. `setOfflineCacheSize` will define the total size of offline cache to store. `isOnline.value` will return a boolean with the connection status.

```dart
final client  = new Client();

client
    .setEndpoint('http://localhost/v1')
    .setProject('455x34dfkjsa542f')
    .setOfflineCacheSize(16000);  // 16MB
    
client.setOfflinePersistency(status: true).then((result) {
  print(result);
});

client.isOnline.value; // returns a boolean connection status
```

### Breaking Changes
<!-- Will this feature introduce any breaking changes? How can we achieve backward compatability -->

This feature should not introduced any breaking changes.

### Documentation & Content
<!-- What documentation do we need to update or add for this feature? -->

We should create a guide expalining how to use offline support and how it works behind the scenes. Each getting started guide should also have a small section explaining this feature briefly in favor of discoverability. We need to update all the relevant contribution guidelines in the SDK geneartor and provide good guidelines for future implementations in other SDKs.

## Reliability

### Security
<!-- How will we secure this feature? -->

N/A

### Scaling
<!-- How will we scale this feature? -->

Not relevant. All of the workload is on the client side.

### Benchmarks
<!-- How will we benchmark this feature? -->

We can do manual tests on a local device or browser. Not sure automating the proccess is worth our time and effort at this stage compared to value as all of the workload is done on the client side.

### Tests (UI, Unit, E2E)
<!-- How will we test this feature? -->

We need to add a test for offline support as part of the SDK generator. We can achieve offline simulation using built in platofrm features or if not possible to create a method in the SDK to mock the network conectivty and overwrite the device/browser original status.

## Open Questions
<!-- List of things we need to figure out or farther discuss -->

N/A

## Future Possibilities
<!-- List of things we could do in the future to extend or take advatage due to this new feature -->

1. Ensure all client side queries work
2. Cache related data
