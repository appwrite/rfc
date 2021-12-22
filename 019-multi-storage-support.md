# Multi Storage Support (BackBlaze and Linode Object Storage)<!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: [@everly-gif](https://github.com/everly-gif)
- Start Date: 22/12/2021
- Target Date: 07/04/2022
- Appwrite Issue: NA
- LinkedIn : [Everly Precia Suresh](https://www.linkedin.com/in/everly-precia-suresh-196bba1b7/)
- Resume/CV : [View Here]()

## Summary
As part of the ongoing progress of implementing multiple storage support to make appwrite agnostic, I propose to implement a BackBlaze and Linode Object storage adapter.
BackBlaze B2 and Linode object storage has been a go to alternative for AWS S3 among users. By implementing these adapters, the user will now be able to choose to store their files in Backblaze B2 or Linode Object Storage.

[summary]: #summary 

<!-- Brief explanation of the proposed contribution. Write your answer below. -->

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

By default, Appwrite uses a docker volume to store the user's files which restricts the user to only one choice of storage. It also makes the user's application incompatible to deploy to mordern day platforms like [Heroku](https://www.heroku.com/).
Therefore, a need for storage adpaters to support multiple storage services is required. 
By implementing storage adapters for backblaze and Linode , the users will now have a choice to choose from a different storage service with a freedom to deploy to mordern day platforms such as [Heroku](https://www.heroku.com/). Hence, making the platform agnostic.

<!--
What problem are you trying to solve? Explain the context or background in which this problem exists.
Please avoid discussing your proposed solution.
-->

## Design proposal (Step 2)

BackBlaze and Linode provide a S3 compatible API that can be utlilized in this implementation.

### Approach

I would divide the implementation of storage adapters into the following steps, by implementing these functions one can effectively implement storage adapters.

- **Authentication**: Providing account/bucket/file access.
- **Bucket Management**: Creating and managing the buckets that hold files.
- **Upload**: Sending files to the cloud.
- **Download**: Retrieving files from the cloud.
- **List**: Data checking/selection/comparison.

### Implementation and Dependencies

Two new PHP files to be created :  
- src/Storage/Device/LOS.php
- src/Storage/Device/BackBlaze.php

Both of the storage implementation will be extending from the S3 adpater class. Therefore, we can utilize the already implemeted AWS V4 signatures  for authentication and other functions in the Backblaze and Linode implementation.

For LOS(Linode object storage) `..Device/LOS.php`:  

```php
namespace Utopia\Storage\Device;

use Utopia\Storage\Device\S3;

class LOS extends S3
{
/* 
Implementation
*/
}
```
For BackBlaze `..Device/BackBlaze.php`: 

```php
namespace Utopia\Storage\Device;

use Utopia\Storage\Device\S3;

class BackBlaze extends S3
{
/* 
Implementation
*/
}
```
The next step would be to implement region constants. Although both of these implementations are s3 compatible the regions these cloud storage provides vary from AWS s3.

Regions for LOS : Atlanta (USA), Frankfurt (Germany), Newark (USA), and Singapore.  
Regions for BackBlaze B2 : Sacramento California, Phoenix Arizona, and Amsterdam Netherlands.

Example: 
```php
const USWEST0 = 'us-west-000';
```
The next step would be the implement the constructors. We will pass in the values of root, accessKey, secretKey, bucket, region and acl to complete the implementation.

Example: 
```php
public function __construct(string $root, string $accessKey, string $secretKey, string $bucket, string $region = self::USWEST0, string $acl = self::ACL_PRIVATE)
    {
        parent::__construct($root, $accessKey, $secretKey, $bucket, $region, $acl);
        $this->headers['host'] = $bucket . '.s3' . '.' . $region . '.backblazeb2.com'; /*the bucket url for BackBlaze stands similar for LOS */
    }

```
### Editing Other files

The above implements the core functionality of the adapter. The final thing to implement would be to just be to set the name and description. After which Updating this information amongs docs / other files will conclude it.

Below are some of the docs/files I will be editing: 

- I will be editing `src/Storage/Storage.php` to add Linode and BackBlaze under the supported devices.
- I will be editing the `Readme.md` to hold updated information about the implementation.
- I will be writing tests to validate my implementation. Hence, will be adding a new file under `tests/Storage/Device/<storage-adapter-name>.php`.


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

### API Endpoints

<!--
List the new API routes or endpoints that we might need to add for supporting the new feature.
Keep in mind to stay very strict to the API protocol and method, whether your new
changes are for the REST, WebSocket or any other API protocol Appwrite supports.

For example:

**POST /v1/coffee ** - an endpoint for creating coffee.
**DELETE /v1/coffee ** - an endpoint for deleting coffee.
-->
There won't be any need for new API Endpoints.
### Data Structure

<!--
What kind of changes or additions are required for the Appwrite base collections
to support this feature. Explain which entities should be added or updated, what new attributes they
need to have and why. Please think well about the naming conventions and how well they play with other
Appwrite conventions. Try and stay as consistent with existing patterns as much as possible.
-->
There won't be any need for new Data Structures.

### Supporting Libraries

<!--
Which different libraries do we need to support the new features?
Please describe the new library's potential API?
Avoid using 3rd party libraries when possible, if required - explain why.
-->
There won't be any need for new supporting Libraries.

### Breaking Changes

<!--
Do we break any API or SDK backward compatibility?
If possible, explain what actions we can take to avoid that.
-->
Since the implementations extends from the S3 adapter class, if there is some changes in the S3 file , especially new API calls that might not be supported by the S3 compatible API provided by BackBlaze and Linode then there is a chance for my implementations to not function.
### Reliability (Tests & Benchmarks)

#### Scaling


<!-- Explain how we will scale this new feature. -->

#### Benchmarks
This feature can be benchmarked on the basis if the user is able to choose among multiple storage providers and upload their files to the respective buckets.
<!-- Explain how we will benchmark the new feature. -->

#### Tests (UI, Unit, E2E)
I will be using PHP unit to test this new feature. I will be making uploads and deletion to the buckets and validate its functionality.

<!-- 
Explain how we will test the new feature. 
You can use "N/A" if this section is not relevant to your proposal.
-->

### Documentation & Content

#### 1. What **docs** would support this feature?
The Readme for the utopia/storage repo would support this feature.  
For LOS : https://www.linode.com/docs/api/object-storage  
For BackBlaze: https://www.backblaze.com/b2/docs/s3_compatible_api.html
#### 2. Do we need to update the **contribution guide** with a new section or a supporting tutorial?
There won't be any need to update the contribution guide. But updating the Readme is required.
#### 3. What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?
For LOS : https://www.linode.com/docs/api/object-storage  
For BackBlaze: https://www.backblaze.com/b2/docs/s3_compatible_api.html
#### 4. What **demo applications** can help us demonstrate this feature APIs and capabilities?
NA

<!--

Documentation is vital for making this new feature a success for both developers using Appwrite and the Appwrite maintainers.
Please answer the following questions:

1. What **docs** would support this feature?
2. Do we need to update the **contribution guide** with a new section or a supporting tutorial?
3. What **tutorials** (text/video) might help developers understand this feature scope, capabilities, and possible use-cases?
4. What **demo applications** can help us demonstrate this feature APIs and capabilities? 

-->

### Prior art

[prior-art]: #prior-art
NA
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

### Unresolved questions

[unresolved-questions]: #unresolved-questions

- Timeline of implementation
- Understanding the different codebase among different repo


<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

- Since the S3 Adapter has been setup for appwrite, I can continue to implement other S3 compatabile APIs among different Cloud Storage Providers during my tenure, This gives the user plenty of options to choose from.

<!-- This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->
