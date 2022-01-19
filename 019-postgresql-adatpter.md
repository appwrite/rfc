# PostgreSQL Adapter <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: [@KevinJ-hub](https://github.com/KevinJ-hub)
- Start Date: 24-01-2022
- Target Date: N/A
- Appwrite Issue: N/A

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

Having support for PostgreSQL will help provide developers the convenience to use their favorite database.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

Currently, Appwrite uses a MariaDB database and provides a No SQL wrapper to interface with this database. NoSQL styled documents and collections are converted to tables and rows under the hood. In order to allow developers to use their favorite database when working with Appwrite, support for PostgreSQl needs to be added.

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

Support for SQL based Databases (MariaDB, MySQL) have already been provided considering the fact that the NoSQL styled documents that are received from the users are converted into tables and rows under the hood for storing in these Databases. This support has been provided by creating an Adapter class which is then inherited in the Database classes.

```bash
database
├── src
│   ├── Database
│   │   ├── Adapter.php
│   │   └── ...
│   └── ...
└── ...
```

The Adapter class lies in the Adapter.php file above which has some predefined function which are used by the users for Database related operations.

To add support for PostgreSQL, an adapter will be created in the following directory.

```bash
database
├── src
│   ├── Database
│   │   ├── Adapter
│   │   │   ├── Postgres.php
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── ...
```

This file has already been created by [@eldadfux](https://github.com/eldadfux). The following file will contain the Postgres class that inherits the Adapter class. All the database operation related functions will be overwritten to support/follow conventions of PostgreSQL. MariaDB.php/MySQL.php file can be used for reference in this case from the directory below.

```bash
database
├── src
│   ├── Database
│   │   ├── Adapter
│   │   │   ├── MariaDB.php
│   │   │   ├── MySQL.php
│   │   │   ├── Postgres.php
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── ...
```

### Reliability

#### Benchmarks

<!-- Explain how we will benchmark the new feature. -->

The support for PostgreSQL can be benchmarked based on the fact that all the functionalities supported in other databases used like MariaDB and MySQL are also properly supported on PostgreSQL.

#### Tests

<!-- 
Explain how we will test the new feature. 
You can use "N/A" if this section is not relevant to your proposal.
-->

Support for PostgreSQL can be tested by using the test files created in the following directly below which use PHPUnit for running the tests.

```bash
database
├── tests
│   ├── Database
│   └── ...
└── ...
```

Some part of the code required for testing has already been written in PostgresTest.php file in the directory below by [@eldadfux](https://github.com/eldadfux) which just needs to be uncommented.

```bash
database
├── tests
│   ├── Database
│   │   ├── Adapter
│   │   │   ├── PostgresTest.php
│   │   │   └── ...
│   └── ...
└── ...
```
