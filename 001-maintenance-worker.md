# Maintenance Worker

- Contribution Name: Maintenance Worker
- Implementation Owner: Unknown
- Start Date: 05-12-2020
- Target Date: 15-12-2020
- Appwrite Issue: None

## Summary

[summary]: #summary

Add a new Appwrite worker to handle periodic maintenance tasks like deleting old logs, abuse records, and inactive sessions or tokens.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

### What problem are you trying to solve?

Large Appwrite systems create lots of logs and data that might not be relevant after X time.

### What is the context or background in which this problem exists?

Lots of data is being stored on Appwrite which might not be relevant. Deleting the data can save a lot of system resources.

### Once the proposal is implemented, how will the system change?

A new worker and maintenance tasks will be added to Appwrite and will run on preset time as part of Appwrite background workers.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal
Create a simple PHP daemon that will be act as a new possible entrypoint for the Appwrite container.

A nice example that could use as a starting point is available here: https://stackoverflow.com/a/44420339/2299554

The sole purpose of the new worker is to execute repetitive server maintenance task at preset intervals (per-hour & per-day for start).

The worker don't need to execute the tasks itself but rather pass them through to the relevant worker message queue (for now mainly the deletes worker), but this might change as more tasks are added and we get a clearer pattern of the actual tasks.

**Code Structure**

The worker script should be located at: `./app/tasks` and use the cli framework to be consistent with other CLI tasks Appwrite executes. The Console class should also be used for different logs. We might want to add a `loop` function to the Console class, but this should be disccused separately.

Add a simple bash script names `maintenance` on `./bin` to allow the entry point to have a nice simple command, the same file should also have relevant references in the main Dockerfile to make it work.

**Tasks:**

The maintenance tasks that should be included in day 1 are:

* Delete all system logs older then `X days` (add a new method to [Audit lib](https://github.com/utopia-php/audit))
* Delete all abuse logs older then `X days` (add a new method to [Abuse lib](https://github.com/utopia-php/abuse))
* Delete all [functions executions (SYSTEM_COLLECTION_EXECUTIONS)](https://github.com/appwrite/appwrite/blob/master/app/config/collections.php#L1619) older then `X days` (delete using exisiting document deletion method available at the [delete worker](https://github.com/appwrite/appwrite/blob/d0f7558ddf1c58ecaeffa9503ac84d0ccc11daee/app%2Fworkers%2Fdeletes.php))

**New Env Vars**

Add a new env variable to determine the amount of `X days` for data deletion. Default value should be 30 days. As with most env vars and settings in Appwrite we should define this time interval in seconds for consistency and predictability. The key of the new variable should be named: `_APP_SYSTEM_RETENTION_TIME` (please suggest better name if any).

**Docs**

Add the new env varibale option to https://appwrite.io/docs/environment-variables under `System Settings`.

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is **reasonably clear how the contribution would be implemented**
- Dependencies on libraries, tools, projects or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables

Ensure that you include examples, code-snippets etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**
-->

### Prior art

[prior-art]: #prior-art

Haven't found any specific data that might be relevant for our exact use case.

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
-->

### Unresolved questions

[unresolved-questions]: #unresolved-questions

Should we also run database optimization scripts as might be recommended after deleting a lot of data?

<!--
- What parts of the design do you expect to resolve through the RFC process
  before this gets merged?
-->

### Future possibilities

[future-possibilities]: #future-possibilities

In the long run we could use this feature to run lots of system health related tasks.

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->
