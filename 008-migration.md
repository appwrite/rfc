# Migration

<!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @torstendittmann
- Start Date: (today's date, dd-mm-yyyy)
- Target Date: (expected date of completion, dd-mm-yyyy)
- Appwrite Issue:
  [Is this RFC inspired by an issue in appwrite](https://github.com/appwrite/appwrite/issues/)

## Summary

[summary]: #summary

Add a migration script to handle internal changes when upgrading to a new version with breaking changes.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

During development, internal breaking changes are often introduced, which must be incorporated when upgrading to a new version. Also, the current migration script is not ready for easy extension of upcoming versions.

**What is the context or background in which this problem exists?**

Since Appwrite is still in early development, breaking change will most likely be introduced with each release. We will face a similar problem with major releases in the future.

**Once the proposal is implemented, how will the system change?**

Not much for the user, only for the developer. The concept of object-oriented programming is introduced to the migration script. Object Inheritance and Polymorphism via Interfaces will ensure easy and safe introduction of upcoming migrations of versions.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

Initial situation is as follows, the current migration script serves its purpose - but is very inflexible and offers little basis for upcoming additions. To improve this, a main parent class and an interface is to be created, which is extended and implemented by the individual migrations. Significant refactoring is performed for the previous migrations.

### New class and interface

A new class will be introduced to `src/Appwrite/Migration`. The class will inherit instances (`$register`, `$projectDB` & co) and methods to the individual migrations, which are needed regardless of the version. Also, abstract functions are introduced which enforces certain methods that have to be custom to the migration. The main class will not be instantiable.

It will look something like this:

```php
abstract class Migration {
  private function __construct() { }

  public function setProject($project, $projectDB) { /* ... */ }
  public function forEachDocument($callback) { /* ... */ }
  
  abstract protected function execute();
}

class V07 extends Migration {
    public function execute() { /* ... */ }
}
```

Following functions are gonna be added the the main `Appwrite\Migration\Migration` class:

#### `setProject(Document $project, Database $projectDB)`
Sets the current project for migration. Will be used by the migration script run from the CLI.

#### `forEachDocument(callable $callback)`
Iterates through every document. A method is passed that takes and returns a `Document`. The returned `Document` will be overwritten in the set project. This will be called by the individual migration class (childs of Appwrite\Migration\Migration) to adjust document structure.

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

There is no reference to the version right now, before we run the migration script and after we finished setting up the new version. If we have a reference in the future, we can move to a dynamic migration that can migrate over different versions.

### Future possibilities

[future-possibilities]: #future-possibilities


In addition, only one version is migrated at the moment and a migration of 2 versions could be possible. 

If a version was skipped, 2 migrations are carried out. For example, if you want to migrate from 0.5 to 0.7, you will first migrate to 0.6 and then to 0.7. This is done automatically in a single operation. For this we would need a reference to the used version number after setting up the new one.
