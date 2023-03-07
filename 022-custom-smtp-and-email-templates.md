# Custom SMTP and Email/SMS Templates Support

- Implementation Owner: @lohanidamodar
- Start Date: 2023-03-06
- Target Date: 2023-03-18
- Appwrite Issue: [https://github.com/appwrite/appwrite/issues/4664](https://github.com/appwrite/appwrite/issues/4664)

## Summary

[summary]: #summary

Support custom SMTP configuration per project and support customizing the email and sms templates. This allows users to set project specific templates as required by the project. Custom email templates will only be supported if the custom SMTP is enabled.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Right now the whole Appwrite system uses a single global SMTP configuration set via environment variables, this means all the projects will use the same email configuration. As well there is no way for user to customize the default email templates. As Appwrite supports multiple projects, which may have different needs for emailing. It is even more crucial in the cloud to support custom email templating as each individual users will have their own need for the project and may want to use their own emailing system to send emails.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### API Endpoints

For this we will introduce various endpoints

**PATCH /v1/project/:projectId/smtp**       - an endpoint for updating smtp configuration for the project

It will accept following parameters

- **enabled** - boolean - **required** - whether or not the SMTP configuration is enabled
- **sender** - string - SMTP sender email
- **host** - string - SMTP server host name address
- **port** - string - SMTP server tcp port
- **secure** - string - SMTP secure protocol (empty or `tls`)
- **username** - string - SMTP server user name
- **password** - string - SMTP server password

**PATCH /v1/project/:projectId/template** - an endpoint to update custom templates (email/sms)

- **locale** - string - required (eg: en_us) - locale of the template
- **key** - string - template key (email.emailVerification, sms.pincode) - the key that defines what the template is for
- **template** - json encoded string - template content, that may have `subject`, `body`, `title`, `senderName`, `senderEmail`.

**GET /v1/project/:projectId/template/:type/:locale** - endpoint to get existing templates. accepts following parameters
- **key** - string - key to identify the template
- **locale** - string - locale of the template

If custom template doesn't exist for the given details, the default server template will be returned.

**DELETE /v1/project/:projectId/template/:type/:locale** - endpoint to remove custom template
- Remove matching custom template or throw 404 if the custom template doesn't exist

### Data Structure

Project document will be updated with new attributes to support SMTP and templates configuration

- **smtp**: json - save SMTP configuration for the project will have following fields
    - **enabled** - boolean - whether or not the SMTP configuration is enabled
    - **sender** - email of the sender
    - **host** - SMTP server host name address
    - **port** - SMTP server tcp port
    - **secure** - SMTP secure protocol (empty or `tls`)
    - **username** - SMTP server user name
    - **password** - SMTP server password

- **templates**: json - save custom templates
    - **key** - string - key that defines what the template is for
        - **message** - string - template body (SMS templates will only have message)
        - **locale** - string - template locale
        - **subject** - string - subject, if email template
        - **senderName** - string - sender name
        - **senderEmail** - string - email of the sender
        - **replyTo** - string - email reply to


### UI

We will need new section under Authentication called **Email & SMS** where user will be able to add these template and SMTP configuration

### Supporting Libraries

We don't need any additional libraries to support this, we already have everything in place.

### Breaking Changes

This is a new feature and shouldn't break any existing features

### Reliability (Tests & Benchmarks)

#### Scaling

This being the part of project API, scaling should already be taken care of.

#### Benchmarks

We don't need separate benchmark for this.

#### Tests (UI, Unit, E2E)

- New e2e tests will be added for these new configurations.
- UI tests will be written for newly added UI section

### Documentation & Content

- SMTP setup documentation should be updated to reflect project level configuration support
- New documentation regarding custom email and sms tempates should be added

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Does this functionality exist in other software, and what experience has their community had?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us, whether they are brand new or an adaptation from other software.

Write your answer below.
-->

N/A

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

- Data structure for Template variables

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->