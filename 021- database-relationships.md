# Database Relationships

* Creator: Jake Barnby
* Relevant Issues: https://github.com/appwrite/appwrite/issues/2735

- [Database Relationships](#database-relationships)
    - [Summary](#summary)
    - [Resources](#resources)
    - [Implementation](#implementation)
        - [Console Mockup](#console-mockup)
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

Modelling data without being able to describe relationships between entities can be difficult and force developers to structure their data in a way that doesn't feel natural to them.

Allowing database relationships will help Appwrite meet developers where they are and reduce friction.

## Resources
<!-- List all the resources used for writing this RFC -->

## Implementation
<!-- Write an overview to explain the suggested implementation -->

We can add a new attribute type for relations: `relationship`. The attribute can have the following options:

- Related collection (select from dropdown)
- Relationship type:
    - One to one (e.g. one user has one profile)
    - One to many (e.g. one user has many posts)
    - Many to one (e.g. many posts have one author)
    - Many to many (e.g. many users have many roles)
- Relationship name (optional, defaults to related collection name, will be used to replace the ID attribute name when fetching data)
- Create twoWay (optional, defaults to true, child of relation also has parent entity as an attribute)
- Two-way attribute name (optional, defaults to collection name, relationship name on child, shown when twoWay is enabled)
- On update action (restrict, cascade, set null)
- On delete action (restrict, cascade, set null)

To create this attribute type you will be able to use an API route or the Appwrite console.

Creating this attribute will add a new entry in the project's `attributes` collection, and add a new attribute to the collection's entry in the project's `metadata` collection.

Additionally, for SQL databases:

- For one-to-one and one-to-many relationships, a new column will be added to the table referencing the `_id` of the related collection.
- For many-to-many relationships, a private junction table will be created with two columns, one for the `_id` of the collection and one for the `_id` of the related collection.

And for NoSQL databases:

- For one-to-one and one-to-many relationships, a new attribute will be added to the collection that references the `_id` of the related collection.
- For many-to-many relationships, a new private collection will be created with two attributes, one for the `_id` of the collection and one for the `_id` of the related collection.

### Relationship Types Details

- One to one, one way:
    - Add relationship to metadata attributes for parent collection
    - Add ID attribute to parent collection
- One to one, two-way:
    - Add relationship to metadata attributes for both collections
    - Add ID attribute to parent and child collections
- One to many, one way:
    - Add relationship to metadata attributes for parent collection
    - Add ID attribute to child collection
- One to many, two-way:
    - Add relationship to metadata attributes for both collections
    - Add ID attribute to child collection
- Many to one, one way:
    - Add relationship to metadata attributes for parent collection
    - Add ID attribute to parent collection
- Many to one, two-way:
    - Add relationship to metadata attributes for both collections
    - Add ID attribute to parent collection
- Many to many, one way:
    - Add relationship to metadata attributes for parent collection
    - Create a new collection with ID attributes for both collections
- Many to many, two-way:
    - Add relationship to metadata attributes for both collections
    - Create a new collection with ID attributes for both collections

### Create

Create operations will be updated to check if the collection has any relationship attributes, if it does, the related data will:

- Be created if it was included as nested data and a document with the same ID doesn't already exist
- Be updated if it was included as nested data and a document with the same ID already exists
- Be referenced if it was included as an ID only

### Read

Get operations will be updated to check if the collection has any relationship attributes, if it does, the related data will be fetched with sub-queries and included in the response.

List operations will allow querying on relationship attributes using dot-notation like: `Query::equal('user.name', ['John'])`, with a maximum level of nesting.

### Update / Delete

Update and delete operations will be updated to check for relationships and perform the appropriate action on related data (restrict, cascade, set null).

If a relationship is deleted, the attributes will be removed from the collection when the type is one-to-one or one-to-many, and the junction table will be deleted when the type is many-to-many.

### POC

`Database.php`:

```php
function createRelationship(
    string $collectionId,
    string $relatedCollectionId,
    string $type,
    bool $twoWay = false,
    string $id = '',
    string $twoWayId = '',
    string $onUpdate = self::RESTRICT,
    string $onDelete = self::RESTRICT
) {
    $collection = $this->getCollection($collection);
    
    $collection->setAttribute('attributes', new Document([
        '$id' => ID::custom($key),
        'type' => $type,
        'key' => $key,
        'options' => [
            'relatedCollectionId' => $relatedCollectionId,
            'twoWay' => $twoWay,
            'twoWayId' => $twoWayId,
            'onUpdate' => $onUpdate,
            'onDelete' => $onDelete,
        ],
    ]), Document::SET_TYPE_APPEND);
    
    $this->updateDocument(self::METADATA, $collection->getId(), $collection);
    
    $relationship = $this->adapter->createRelationship(
        $collection->getId(),
        $relatedCollectionId,
        $type,
        $key,
        $twoWay,
        $twoWayId,
        $onUpdate,
        $onDelete
    );
}
```

`MariaDB.php`

```sql
-- Create the new relationship entry
INSERT INTO `appwrite`.`_2_attributes`
    VALUES (1, 1, 2, 'manyToOne', 'artist', 0, 'cascade', 'cascade');

-- Update the metadata table with the new relationship
UPDATE `_2__metadata`
    SET `attributes` = '[{"$id": "artist", "type": "manyToOne", "key": "artist", "options": { "relatedCollection": "database_1_collection_2", "twoWay": false, "onUpdate": "cascade", "onDelete": "cascade" }}]'
    WHERE `_uid` = 'database_1_collection_1';

-- Add the new column to the collection
ALTER TABLE `appwrite`.`_2_database_1_collection_1`
    ADD COLUMN `artistId` INT(11) UNSIGNED;
```

Creating a relationship with an SDK:

```php
$client = new Client();
$databases = new Databases(client);

$relationship = $databases->createRelationship(
    databaseId: 'music',
    collectionId: 'albums',
    relatedCollectionId: 'artists',
    type: Relationship.ManyToOne,
    key: 'artist',
    twoWay: true,
    twoWayId: 'albums',
    onUpdate: self::CASCADE,
    onDelete: self::RESTRICT
);
```

Creating a document with a relationship, passing related data, with an SDK.

In this case, the artist document will inherit the permissions of the album document.

```php
$client = new Client();
$databases = new Databases(client);

$relationship = $databases->createDocument(
    databaseId: 'music',
    collectionId: 'albums',
    documentId: ID::unique(),
    permissions: [
        Permission::read(Role::users()),
        Permission::delete(Role::user('me')),
    ],
    data: [
        'name' => 'Abbey Road',
        'artist' => [
            '$id' => ID::unique(),
            'name' => 'The Beatles',
        ],
    ]
);
```

Creating a document with a relationship, passing related document ID, with an SDK.

In this case, the artist document permissions will be checked, and an exception will be thrown if the user does not have read permission.

```php
$client = new Client();
$databases = new Databases(client);

$relationship = $databases->createDocument(
    databaseId: 'music',
    collectionId: 'albums',
    documentId: ID::unique(),
    permissions: [
        Permission::read(Role::users()),
        Permission::delete(Role::user('me')),
    ],
    data: [
        'name' => 'Abbey Road',
        'artist' => '6fef123456'
    ]
);
```

When getting/finding a document, the adapter will check for any relationships and results will be joined with the related collections:

```sql
SELECT * FROM `appwrite`.`_2_database_1_collection_1`
    JOIN `appwrite`.`_2_database_1_collection_2`
    ON `appwrite`.`_2_database_1_collection_1`.`artistId` = `appwrite`.`_2_database_1_collection_2`.`_id`
```

If two-way is set to true, the adapter will also join the related collection to the current collection:

```sql
SELECT * FROM `appwrite`.`_2_database_1_collection_2`
    JOIN `appwrite`.`_2_database_1_collection_1`
    ON `appwrite`.`_2_database_1_collection_2`.`_id` = `appwrite`.`_2_database_1_collection_1`.`artistId`
```

Once the data is fetched, it will be shaped using the relationship's metadata.

Following the collections set up above: artists and albums with a many-to-one relationship from album to artist with key `artist`, where each collection has a name attribute, the resulting data would be shaped like:

```json
{
    "name": "Abbey Road",
    "artist": {
        "name": "The Beatles"
    }
}
```

And with two-way:

```json
{
    "name": "The Beatles",
    "albums": [
        {
            "name": "Abbey Road"
        }
    ]
}
```

### Console Mockup

https://www.figma.com/proto/WxczFR7RG5OLKMnC0EW9YD/Databases?node-id=2283%3A140215&scaling=scale-down&page-id=2283%3A136756&starting-point-node-id=2283%3A140215

### API Changes
<!-- Do we need new API endpoints? List and describe them and their API signatures -->

- POST /v1/databases/:databaseId/collections/:collectionId/attributes/relationship
    - databaseId: string
    - collectionId: string
    - relatedCollectionId: string
    - key: string
    - type: whitelist(oneToOne, oneToMany, manyToOne, manyToMany)
    - required: bool
    - twoWay: bool
    - twoWayId: string
    - onUpdate: whitelist(restrict, cascade, set null)
    - onDelete: whitelist(restrict, cascade, set null)

### Console Changes
<!-- Do we need new console features? List and describe them -->

- Add a new attribute type to the list shown when selecting "create attribute"
- Add a selector for selecting the related document(s) when creating/updating a document in a collection with relationships
- Add a link to related data in the document list view
- Add a link to related data in the document view

###  Workers / Commands
<!-- Do we need new workers or commands for this feature? List and describe them and their API signatures -->

Workers will need to create the new column in the collection table and add the foreign key constraint.

###  Supporting Libraries
<!-- Do we need new libraries for this feature? Mention which, define the file structure, and different interfaces -->

N/A

### Data Structures
<!-- Do we need new data structures for this feature? Describe and explain the new collections and attributes -->

- _{projectId}__metadata new fields in the `attributes` attribute:
    - relatedCollectionId: string
    - relationType: whitelist(oneToOne, oneToMany, manyToOne, manyToMany)
    - twoWay: bool
    - twoWayId: string
    - onDelete: whitelist(restrict, cascade, set null)
    - onUpdate: whitelist(restrict, cascade, set null)

- _{projectId}_attributes new attributes:
    - relatedCollectionId: string
    - relationType: whitelist(oneToOne, oneToMany, manyToOne, manyToMany)
    - twoWay: bool
    - twoWayId: string
    - onDelete: whitelist(restrict, cascade, set null)
    - onUpdate: whitelist(restrict, cascade, set null)

### SDKs
<!-- Do we need to update our SDKs for this feature? Describe how -->

Need to be able to supported nested objects in document data. Most SDKs should already support this.

### Breaking Changes
<!-- Will this feature introduce any breaking changes? How can we achieve backward compatability -->

N/A

### Documentation & Content
<!-- What documentation do we need to update or add for this feature? -->

- Update [Database guide](https://appwrite.io/docs/databases), add a new section for relationships
- Add new route reference for the new endpoint

## Reliability

Ensured by tests

### Security
<!-- How will we secure this feature? -->

Existing security features. Permissions will cascade to related documents.

When creating a document with a relationship, the related collection permissions will be checked and an error will be thrown if the user does not have the create permission.

When creating a document and passing related data at the same time, the permissions will default to the same as the parent document, but can be overridden with the `$permissions` parameter.

When creating a document and passing related document ID, the permissions will of the related document will be checked and an error will be thrown if the user does not have read permission.

If a user is allowed to read a document, but not allowed to read a related document, the related document will be returned as null.

### Scaling
<!-- How will we scale this feature? -->

There will be a defined maximum for how many levels deep (of nesting) relationships can be on the Appwrite side.

N/A

### Benchmarks
<!-- How will we benchmark this feature? -->

- Load tests will be performed to ensure the feature is not causing any performance issues.

### Tests (UI, Unit, E2E)
<!-- How will we test this feature? -->

- Add new E2E tests for the new API endpoints.
- Add new unit tests for the new adapter methods.
- Add new unit tests for the new adapter methods, specifically checking permissions features.

## Open Questions
<!-- List of things we need to figure out or further discuss -->

## Future Possibilities
<!-- List of things we could do in the future to extend or take advantage due to this new feature -->

Use of foreign keys to automate the creation of indexes and cascading updates/deletes.