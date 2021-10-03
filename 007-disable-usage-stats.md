# Ability to Disable Usage Stats <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 12-01-2021
- Target Date: Unknown
- Appwrite Issue: https://github.com/appwrite/appwrite/issues/547

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
Due to difficulty in deployment of influxdb and telegraf in services like Heroku, in the Appwrite lite version, usage stats has to be disabled. So, want to introduce a environment variable that will allow disabling usage stats completely there by influxdb, telegraf and appwrite usage worker dependencies will no longer be required.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

Making Appwrite lite version easily deployable to 1 click deployment platforms like Heroku, where there is no easy way to support influxdb and telegraf that are used to record and display usage stats. So, allowing to disable this and disabling this in the Appwrite lite version will enable easy deployment to platforms like Heroku.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

Supporting one click deployments to free tiers of platforms like Heroku, will help a lot in Appwrite adoption. This will allow any beginners to easily instantiate an Appwrite platform and see it in action, even use it in production in an environment with less resources.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

Users will be able to disable the usage stats feature completely. This will disable the display of usage statistics graph in dashboard. But this will not change any other parts of Appwrite or it's API.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Introduce new environment variable
Introduce a new environment variable to allow disabling usage stats.
    - _APP_USAGE_STATS = enabled | disabled

### Use the environment variable
Use the environment variable to disable collection and display of usage stats. Once disabled, the usage stats end points (project and functions) will only return other stats if any apart from those from influxdb/telegraf. Also, console dashboard will hide the usage stats graph.

### Changes to be made
Following changes will be required in order to support disabling usage stats

1. functions usage endpoint : https://github.com/appwrite/appwrite/blob/d6df6b9fdc9e05b72fe0fb391e787a7dce9fc5b9/app/controllers/api/functions.php#L134
  
  Here, after making sure the function exists. Check if stats is disabled. If disabled skip calculations from line declaring $period - sending the response. Instead just send a dummy response in the same format with dummy data.

2. projects usage endpoint : https://github.com/appwrite/appwrite/blob/055a7ef8eb56067484ec27c03e6210afc84c7244/app/controllers/api/projects.php#L155

  Here, after making sure the project exists. Check if stats is disabled. If disabled skip calculations from line declaring $period - before getting users collection data. In the response if the stats is disabled send dummy data for `requests`, `network`, `functions` keys.

3. Console home controller: https://github.com/appwrite/appwrite/blob/7fc7e0b93d8b0ae9bcbf5f775c73c59db5910af5/app/controllers/web/console.php#L116

  Here, just pass the value of environment variable to the view.

4. Home view : https://github.com/appwrite/appwrite/blob/881b1e71a3244b11d3e1eb064727791676636bc3/app/views/console/home/index.phtml#L64

  Here, based on the value of environment variable, show / hide the charts inside project stats div.

5. Functions controller : https://github.com/appwrite/appwrite/blob/7fc7e0b93d8b0ae9bcbf5f775c73c59db5910af5/app/controllers/web/console.php#L379

  Here, pass the value of environment variable to the view.

6. Functions view : https://github.com/appwrite/appwrite/blob/881b1e71a3244b11d3e1eb064727791676636bc3/app/views/console/functions/function.phtml#L243

  Here based on the value of environment variable, show/hide the usage, monitor graph below line 243.

7. App shutdown function : https://github.com/appwrite/appwrite/blob/9c421e2dfc64cc7c929f26e5a9193735e1b1c9c6/app/controllers/general.php#L320

  Here, if stats disabled, stop sending the data to redis.

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

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
This will allow easily deployment of Appwrite to various platforms like Heroku, which will help a lot in increasing adoption. This will also favor easy deployment to servers with less resources.
