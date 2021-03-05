# One Click Deploy to Heroku <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 12-01-2021
- Target Date: Unknown
- Appwrite Issue:
  https://github.com/appwrite/appwrite/issues/461, https://github.com/appwrite/appwrite/issues/547

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
Supporting one click deploy to Heroku right from GitHub, with Appwrite Lite. Which will increase our adoption rate as anyone can click to deploy to Heroku for free and try Appwrite, or even use in in production with less resources.

___ 

Heroku and similar platforms are modern day standards for deploying applications. But deploying on such platforms requires certain fundamental architectural decisions made. Supporting these platforms will help make deploying and testing Appwrite easier for beginners. We can even support one click deployment right from Github.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->
Deploying to Heroku, with a deploy to Heroku button, right from GitHub readme of Appwrite Lite repo.

___
Heroku, deploys applications on an **ephemeral** storage called dynos, where data can be written to disk however those data will not persist after the application is restarted. This provides fundamental challenge to deploy Appwrite, as appwrite uses filesystem storaging and managing things like uploads, functions, config and cache.

Next up, for services like database and redis, there are Heroku Add-ons that can be used, however there is no influxdb/telegraf Add-ons on Heroku.

These problems are limiting the ability of Appwrite to be deployed in Heroku


**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

The current implementation of storage and influxdb/telegraf prevents from deployment to Heroku and make it usable properly.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

- There will be an option to disable influxdb/telegraf stats using environment variable.
- There will be an option to switch default storages (uploads, configs, cache and functions) to external services like Amazon S3 or Digitalocean Spaces (may be).

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Heroku docker configuration
Need to create a Heroku docker configuration file in the Appwrite Lite repo, which will allow us to build and deploy Appwrite Lite docker image easily to Heroku. [Reference to heroku docker configuration](https://devcenter.heroku.com/articles/build-docker-images-heroku-yml).

### Heroku app configuration
Create a Heroku app configuration file (app.json) in the Appwrite Lite repo which will allow us to support one click deployment to Heroku right from GitHub. [Reference to app configuration ](https://devcenter.heroku.com/articles/app-json-schema).

Redis and Mysql add-ons will have to be added in the config. The environment variables for Redis and Mysql connections will have to be updated accordingly in the Heroku docker configuration.

### POC
First create a POC of one click deploy to Heroku as it's current state. Even if no data in file system persists, we can use the same idea further once we add support to external storage services like S3.

---

### Introduce new environment variable to disable stats
Should introduce a new environment variable _APP_USAGE_STATS, default value is **enabled** and can be set to **disabled** to disable the stats, enabling user to completely drop influxdb/telegraph without any errors. More on this is described in [Disable Usage Stats RFC](https://github.com/appwrite/rfc/blob/main/007-disable-usage-stats.md)

### Introduce external storage support
Should introduce new environment variables for storage service config.

### Update documentation
Update documentation to highlight these new options, to integrate external storage service and to disable stats

### Tutorial
Create a short totorial on how to disable stats completely and how to integrate external storage service like Amazon S3.

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

This feature will make it easier for beginners to quickly spin up appwrite server and try it or even use it in production with low resource cost.