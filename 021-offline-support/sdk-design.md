# SDK Design

## Local Collections

### Data

Data is cached locally into into local collections. Each Appwrite Model is stored in it's own local collection. For example, the List Contintents API returns a list of `Continent` objects so there is a local `/locale/continents` collection.

| attribute          | description                                     |
| ------------------ | ----------------------------------------------- |
| key                | unique identifier for the cached record         |
| [model attributes] | each attribute on the collection is also stored |

Each Appwrite Collection will also have it's own local collection: `/databases/{databaseId}/collections/{collectionId}/documents`.

### Metadata

In addition to the data collections, there are some metadata collections too.

#### Access Timestamps

The `accessTimestamps` collection is used to store the timestamp in which a cached record was accessed. It has the following attributes:

| attribute  | description                                          |
| ---------- | ---------------------------------------------------- |
| model      | the local collection where this record is in         |
| key        | the unique identifier for the cached record          |
| accessedAt | the timestamp in which this record was last accessed |

This collection is used to determine which records are least used and can be evicted if the local cache is too full.

#### Cache Size

The `cacheSize` collection is used to store the total size of the cached data. When any cached data changes, the value is updated to reflect the new value.

#### Queued Writes

The `queuedWrites` collection stores the create, update, or delete requests made while offline. It has the following attributes:

| attribute                 | description                                                                                                       |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| queuedAt                  | timestamp in which this request was initially created                                                             |
| method                    | MTTP method for the request                                                                                       |
| path                      | HTTP path for the request                                                                                         |
| headers                   | HTTP headers for the request                                                                                      |
| params                    | all parameters for the HTTP request                                                                               |
| cacheModel                | local collection for the record                                                                                   |
| cacheKey                  | unique identifier in the local collection                                                                         |
| cacheResponseIdKey        | the property in the JSON response that has the ID of the record (usually `$id`)                                   |
| cacheResponseContainerKey | the property in the JSON response that has the list of records (e.g. `documents` for the List Documents API call) |
| previous                  | the version of the record before this write was queued. used to revert the local cached record on failure         |

## SDK Behavior

### Initialization

To enable Offline Support, you must call the `setOfflinePersistency` function on `Client` and wait for it to resolve before proceeding:

JavaScript:

```javascript
await client.setOfflinePersistency(true);
```

Dart:

```dart
await client.setOfflinePersistency(status: true)
```

This will:

1. Set an offline persistency flag to true.
1. Initialize the offline database.
1. Register listeners for connectivity to help with detecting online status.
1. Register a listener on the `cacheSize` collection such that if the value is greater than the limit, the least accessed records will be deleted.
1. Process the queued writes.

### Processing Queued Writes

When the queued writes are processed during initialization, the SDK:

1. Returns if offline
1. Iterates over each queued write
1. Attempts to send the HTTP request
1. If successful, update cached data
1. Else, restore previous record
1. Delete queued write

### Sending the HTTP Request

The `call()` function on `Client` is used to send the request or use the offline database. At a high level, this function:

1. Checks the online status.
1. If the device is offline and the offline persistency flag is true:
   1. Add the current timestamp to the `X-Appwrite-Timestamp` header.
   1. If the request method is GET:
      1. Fetch the record(s) from the local collection.
      1. For each record, update the accessed at timestamp.
      1. Return the data.
   1. If the request method is POST, PATCH, PUT, or DELETE:
      1. If the request method is POST:
         1. Insert the record into the appropriate local collection.
         1. Queue a write into the `queuedWrites`.
      1. If the request method is DELETE:
         1. Fetch the record from the local collection.
         1. Delete the local record.
         1. Queue a write into the `queuedWrites`.
      1. If the request method is PUT or PATCH:
         1. Fetch the record from the local collection.
         1. Update the local record.
         1. Queue a write into the `queuedWrites`.
      1. Register a listener for when device is online again.
1. If the device is online:
   1. Make the API call.
   1. If offline persistency flag is set to true:
      1. Update local collection

### Checking Online Status

In order to check for online status, a network request must be made:

1. Make a socket connection to appwrite.io on port 443.
1. If successful, device is online.
1. If unsuccessful, device is offline.

### Going Online

While offline, each POST, PUT, PATCH, or DELETE API call registers a listener that triggers when the offline status updates. This listener executes the following in a loop:

1. Get the next queued write.
1. Check if it matches the current request.
1. If not, pause and try again.
1. Try the API call.
1. If successful:
   1. If request method is POST, update record in local collection.
   1. Delete queued write.
   1. Resolve the asyncronous operation.
1. If unsuccessful:
   1. If error code is 404:
      1. Delete the record from the local collection.
      1. Delete the queued write.
   1. If error code is >= 400:
      1. Restore the previous record
      1. Delete the queued write.
   1. Resolve the asyncronous operation.
1. Remove the listener.

### Adding or Updating a Record in a Local Collection

1. Fetch the record from the local collection.
1. Calculate the size difference between the old record and new record.
1. Update the cache size with the difference.
1. Add or Update the record in the local collection.
1. Update the accessed at timestamp
