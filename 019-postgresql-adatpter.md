# PostgreSQL Adapter <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner's Github: [@KevinJ-hub](https://github.com/KevinJ-hub)
- Implementation Owner's Resume: [KevinJoshi_Resume.pdf](https://drive.google.com/file/d/13U1FjjL2Jg5pfoUbtPgt-hBeXdS5XgRV/view?usp=sharing)
- Implementation Owner's Linkedin: [@kevinjoshi46b](https://www.linkedin.com/in/kevinjoshi46b/)
- Start Date: 17-01-2022
- Target Date: 10-04-2022
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

## Extra

[extra]: #extra

### Why am I the best person to execute this proposal?

I have had a bit of practical experience of using PostgreSQL with php in the past. (I had created a currency converter mini project while learning basics of php that you can checkout [here](https://github.com/KevinJ-hub/CurCon)) I have learned Object Oriented Php as well but never got to implement it in a project so this will be a good learning experience for me where I will be able to implement the core knowledge that I already have in a practical scenario. I am also extremely excited to contribute and be a part of appwrite which aims to provide users with all the core tools and services required for building any modern day software easily. (A one-stop solution)

### Project Timeline

#### **Week 1 - 2 (17-01-2022 to 30-01-2022)**

- Getting some key insights from mentors and community bonding.
- Setting up the project on my system.
- Exploring the file structure and the flow of the system in depth.

#### **Week 3 - 8 (31-01-2022 to 13-03-2022)**

- Working on properly completing the PostgreSQL adapter to support all the database operations.

#### **Week 9 - 10 (14-03-2022 to 27-03-2022)**

- Running tests and debugging to make sure that everything works properly as it should.

#### **Week 11 - 12 (28-03-2022 to 10-04-2022)**

- Finalizing the project by completing any left over work
- Completing the documentation work (if any)

> **NOTE:** I am currently a third year undergraduate student so I will be having my mid-term exams in week 8 (07-03-2022 to 13-03-2022) hence, the project timeline is tentative and could be subject to change.

### Project Deliverables

At the 3 month program the following result shall be delivered:-

- Properly tested and working PostgreSQL Adapter.
