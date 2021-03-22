# Database Adapters, and Custom Indices

- Implementation Owner: @eldadfux
- Start Date: 07-03-2021
- Target Date: Unknown
- Appwrite Issue:
  [New database rule types #395](https://github.com/appwrite/appwrite/issues/395)
  [Optimize default DB indices #506](https://github.com/appwrite/appwrite/issues/506)
  [Add Fields param in ListDocument and GetDocument APIs #499](https://github.com/appwrite/appwrite/issues/499)
  [MongoDB in Appwrite #909](https://github.com/appwrite/appwrite/issues/909)

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Appwrite's database doesn't have a built-in way to customize DB indexes or under the hood adapters. The idea in this RFC is to make both options available directly from the database dashboard.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

The current implementation of the Appwrite database is very good and quick to get started easily, but doesn't allow enough flexibility for when your data set grows or when you need to implement more advanced use-cases.

**What is the context or background in which this problem exists?**

Larger data set and more complex projects, require more flexibility and transparency regarding the database inner structure and performance capabilities.

**Once the proposal is implemented, how will the system change?**

We will implement simplified data structures that could be manipulated from both the Appwrite dashboard and directly from the relevant connected DB. We will also add new endpoints to easily add, delete, and view collection rules and indexes.

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

### Utopia Database

We'll implement an extra database abstraction layer as part of the Utopia project. This layer will include support for multiple database, and will expose a consistent API which will act as the lowest common denominator for all the supported adapters.

The list of supported adapters will include:
* Postgres
* MySQL
* MariaDB
* MongoDB¹

#### Data Types

This library will support storing and fetching of all common JSON simple and complex [data types](https://restfulapi.net/json-data-types/).

##### Simple Types

* String
* Integer
* Float
* Boolean
* Null

##### Complex Types
* Array
* Object
* Relationships
  * Reference (collection / document)
  * References (Array of - collection / document)

> Databases that don't support the storage of complex data types should store them as strings and parse them correctly when fetched.

#### Persistency

Each database adapter should support the following actions for fast storing and retrieval of collections of documents.

**Databases** (Schemas for SQL)
* create
* delete

**Collections** (Tables for SQL)
* createCollection($name)
* deleteCollection($name)

**Attributes** (Table columns for SQL)
* createAttribute(string $collection, string $name, string $type)
* deleteAttribute(string $collection, string $name)

**Indexes** (Table indexes for SQL)
* createIndex(string $collection, string $name, string $type)
* deleteIndex(string $collection, string $name, string $type)

**Documents** (Table rows columns for SQL)
* getDocument(string $collection, $id)
* createDocument(string $collection, array $data)
* updateDocument(string $collection, $id, array $data)
* deleteDocument(string $collection, $id)

#### Queries

Each database adapter should allow for simple and advanced queries in consideration of underlying limitations.

Method for querying data:
* find(string $collection, $filters)
* findFirst(string $collection, $filters) (extending `find`)
* findLast(string $collection, $filters) (extending `find`)
* count(string $collection, $filters) (extending `find`)

##### Supported Operators
* Equal (==)
* Not Equal (!=)
* Less Than (<)
* Less or equal (<=)
* Bigger Than (>)
* Bigger or equal (>=)
* Contains / In
* Not Contains / Not In
* Is Null
* Is Not Null
* Is Empty
* Is Not Empty

##### Joins / Relationships

#### Paging

Each database adapter should support two methods for paging. The first method is the classic `Limit and Offset`. The second method is `Limit and After` paging, which allows for better performance at a larger scale.

#### Orders

Support multi-column orders, multiple order type (ASC, DESC). We should leverage native casting for all types and avoid any manual casting of the data.

#### More Features

> Each database collection should hold parallel tables (if needed) with row-level metadata, used to abstract features that are not enabled in all the adapters.

##### Row-level Security

Each database collection will hold metadata information containing all the read and write permissions data for each document in the collection. In SQL based adapters this data can be left joined using a dedicated table postfixed with the name `_permissions`. Row level security could be disabled when accessing the data using admin credentials for improved performance. Security credentials should be indexed by default on every new collection.

##### GEO Queries

##### Free Search

##### Filters

Allow to apply custom filters on specific pre-chosen fields. Available filters:

* Encryption
* JSON (might be redundant with object support)
* Hashing (md5,bcrypt)

##### Caching

The library should support memory caching using internal or external memory devices for all read operations. Write operations should actively clean or update the cache.

##### Encoding

All database adapters should support UTF-8 encoding and accept emoji characters.

#### Tests

* Check for SQL Injections

#### Examples (MariaDB)

**Collections Metadata**

```sql
CREATE TABLE IF NOT EXISTS `collections` (
  `_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `_metadata` text() DEFAULT NULL,
  PRIMARY KEY (`id`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Documents**

```sql
CREATE TABLE IF NOT EXISTS `documents_[NAME]` (
  `_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `_uid` varchar(128) NOT NULL AUTO_INCREMENT,
  `custom1` text() DEFAULT NULL,
  `custom2` text() DEFAULT NULL,
  `custom3` text() DEFAULT NULL,
  PRIMARY KEY (`_id`),
  UNIQUE KEY `_index1` (`$id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Documents Authorization**

```sql
CREATE TABLE IF NOT EXISTS `documents_[NAME]_authorization` (
  `_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `_document` varchar(128) DEFAULT NULL,
  `_role` varchar(128) DEFAULT NULL,
  `_action` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`_id`),
  KEY `_index1` (`_document`)
  KEY `_index1` (`_document`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
``` 

### Database API

**Collections**

* POST /v1/database/collections (create collection)
* GET /v1/database/collections (list collections)
* GET /v1/database/collections/:id (get collection)
* PUT /v1/database/collections/:id (update collection)
* DELETE /v1/database/collections/:id (delete collection)

**Collection Attributes** (no update)

* POST /v1/database/collections/:id/attributes (create attribute)
* GET /v1/database/collections/:id/attributes/:attribute (list attributes)
* GET /v1/database/collections/:id/attributes/:attribute (get attribute)
* DELETE /v1/database/collections/:id/attributes/:attribute (delete attribute)

**Collection Indexes** (no update)

* POST /v1/database/collections/:id/indexes (create index)
* GET /v1/database/collections/:id/indexes/:index (list indexes)
* GET /v1/database/collections/:id/indexes/:index (get index)
* DELETE /v1/database/collections/:id/indexes/:index (delete index)

**Collection Documents**

* POST /v1/database/collections/:id/documents (create document)
* GET /v1/database/collections/:id/documents/:document (list documents)
* GET /v1/database/collections/:id/documents/:document (get document)
* PUT /v1/database/collections/:id/documents/:document (update document)
* DELETE /v1/database/collections/:id/documents/:document (delete document)

> We'll use index and indexes for naming conventions, as its is more consistent than indexs vs indices.

### Database Worker

### Documentation

Below is a list of some of the main topics we should cover as part of the database documentation. This is in addition to the auto-generated API spec.

* Overview (concept, architecture)
* Data Types
* Permissions
* Operations (queries + examples)
* Attributes (rules) and indexes
* Security
* Benchmarks
* Performance Tips
* Known Limitations

### Resources

* SQL Optimization - https://www.eversql.com/
* Function syntax parsing https://gist.github.com/lamberta/3768814

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

#### Query Syntax for Filter

Our current query syntax is limited and doesn't allow us to detect given value types, or to use operators like `OR` and `AND`.

**Current**:

`movie.director.name=Michael Bay`

**New Alternative**:

`movie.director.name.equal('Michael Bay')`

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->


### Comments

¹ Optional - might be a bit tricky to have fully supported adapter.