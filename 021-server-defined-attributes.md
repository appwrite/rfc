# Optional server-defined attributes

* Creator: Khushboo Verma
* Relevant Issues: N/A

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
<!-- Describe the problem we want to solve and suggested solution in a few paragraphs -->
This RFC proposes the addition of optional server-defined attributes in all the project collections that can only be added by server and cannot be modified by the client. The user can add them to each collection through the API or console. Some useful sttributes can be `createdBy`, `updatedBy`, `ip` etc.

## Resources
<!-- List all the resources used for writing this RFC -->
NA

## Implementation

<!-- Write an overview to explain the suggested implementation -->
#### Console Changes
We need to add an option in the `create attribute` settings dropdown to add a certain server-defined attribute from one of the existing options like `createdBy`.

### API Changes
<!-- Do we need new API endpoints? List and describe them and their API signatures -->
We need 2 new API endpoints in `databases.php` for creating and deleting server-defined attributes.
- POST `/v1/databases/:databaseId/collections/:collectionId/attributes/server`
- DELETE `/v1/databases/:databaseId/collections/:collectionId/attributes/server/:key`

We can allow users to create indexes for new attributes from the `createIndex` endpoint.

We need to add a check in `createDocument` and `updateDocument` endpoints to populate the values of these attributes if they exist for client-side requests.

###  Workers / Commands
<!-- Do we need new workers or commands for this feature? List and describe them and their API signatures -->
No new workers are anticipated.

###  Supporting Libraries
<!-- Do we need new libraries for this feature? Mention which, define the file structure, and different interfaces -->
No new changes required in libraries.

### Data Structures
<!-- Do we need new data structures for this feature? Describe and explain the new collections and attributes -->
#### In appwrite/appwrite
The new attributes would add attributes to `attributes` collection in `$projectCollections` for each optional attribute.

```
$projectCollections = array_merge([
  'attributes' => [
        '$collection' => ID::custom(Database::METADATA),
        '$id' => ID::custom('attributes'),
        'name' => 'Attributes',
        'attributes' => [
            [
                '$id' => ID::custom('createdBy'),
                'type' => Database::VAR_STRING,
                'format' => '',
                'size' => 256,
                'signed' => true,
                'required' => false,
                'default' => null,
                'array' => false,
                'filters' => [],
            ],
...
]
```

### SDKs
<!-- Do we need to update our SDKs for this feature? Describe how -->
SDKs will be updated to accommodate the changes in the API endpoints.

### Breaking Changes
<!-- Will this feature introduce any breaking changes? How can we achieve backward compatability -->
No breaking changes are anticipated as this is an additive feature.

### Documentation & Content
<!-- What documentation do we need to update or add for this feature? -->
The documentation will be updated to inform developers about the new endpoints and how to add these optional attributes in their collections.

## Reliability

### Security
<!-- How will we secure this feature? -->

### Scaling
<!-- How will we scale this feature? -->
This feature will rely on existing infrastructure, so additional scaling measures are not anticipated.

### Benchmarks
<!-- How will we benchmark this feature? -->
No new benchmarks are required as existing infrastructure will support the feature.

### Tests (UI, Unit, E2E)
<!-- How will we test this feature? -->
- End-to-end testing for each new endpoint and functionality.
- User Interface testing if a GUI component is added.

## Open Questions
<!-- List of things we need to figure out or farther discuss -->
N/A

## Future Possibilities
<!-- List of things we could do in the future to extend or take advatage due to this new feature -->
We could add more attributes like `device`, `geoLocation` etc.
