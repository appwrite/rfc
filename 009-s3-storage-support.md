# S3 Storage Support <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: (@lohanidamodar)
- Start Date: (22-01-2021)
- Target Date: (N/A)
- Appwrite Issue:
  [Heroku Support](https://github.com/appwrite/appwrite/issues/461),
  [Appwrite Lite](https://github.com/appwrite/appwrite/issues/547)

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
This is next step in supporting multiple storage adapters to make Appwrite Storage platform agnostic. This will add S3 adapter, that will help use AWS s3 as an storage with Appwrite.


## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->

This feature is in process to make Appwrite Storage platform agnostic. This will allow adding AWS S3 as an storage option wih Appwrite, this will in turn allow appwrite to be delopyed in the modern platforms like Heroku, without loosing data.


**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->

While trying to deploy Appwrite to modern platforms like Heroku, using local device as an storage is not an option. As Heroku uses temporary storage devices to deploy applications. So we need to outsource storage to outside platform. This and other adapters will allow us to make this happen and make Appwrite ready to deploy on platforms like Heroku.

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

<!-- Please avoid discussing your proposed solution. -->
This will add S3 storage adapter as an alternative to Local storage adapter. This will allow users to choose between S3 or Local as the default storage adapter.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

We need to implement new storage adapters.

### Add S3 adapter
Add, S3 adapter that implements `Device` and its required functions in [utopia-php/storage](https://github.com/utopia-php/storage) library. Also add more functions that is required for S3. We already have a barebones S3 adapter, where we need to implement the required functions

### Generating Signature
The authorization signature will have to be generated, for that we will need, Access Key, Secret, Bucket, Region, and Access Control List. The following sample code shows how we can generate AWS auth signature v4.

We will use `private` ACL so that the files are only accessible via Appwrite.

```php
$accessKeyId = 'YOUR_ACCESS_KEY_ID';
$secretKey = 'YOUR_SECRET_KEY';
$bucket = 'YOUR_BUCKET_NAME';
$region = 'BUCKET_AMAZON_REGION'; // us-west-2, us-east-1, etc
$acl = 'ACCESS_CONTROL_LIST'; // private, public-read, etc

// VARIABLES
// These are used throughout the request.
$longDate = gmdate('Ymd\THis\Z');
$shortDate = gmdate('Ymd');
$credential = $accessKeyId . '/' . $shortDate . '/' . $region . '/s3/aws4_request';

// POST POLICY
// Amazon requires a base64-encoded POST policy written in JSON.
// This tells Amazon what is acceptable for this request. For
// simplicity, we set the expiration date to always be 24H in 
// the future. The two "starts-with" fields are used to restrict
// the content of "key" and "Content-Type", which are specified
// later in the POST fields. Again for simplicity, we use blank
// values ('') to not put any restrictions on those two fields.
$policy = base64_encode(json_encode([
    'expiration' => gmdate('Y-m-d\TH:i:s\Z', time() + 86400),
    'conditions' => [
        ['acl' => $acl],
        ['bucket' => $bucket],
        ['starts-with', '$Content-Type', ''],
        ['starts-with', '$key', ''],
        ['x-amz-algorithm' => 'AWS4-HMAC-SHA256'],
        ['x-amz-credential' => $credential],
        ['x-amz-date' => $longDate]
    ]
]));

// SIGNATURE
// A base64-encoded HMAC hashed signature with your secret key.
// This is used so Amazon can verify your request, and will be
// passed along in a POST field later.
$signingKey = hash_hmac('sha256', $shortDate, 'AWS4' . $secretKey, true);
$signingKey = hash_hmac('sha256', $region, $signingKey, true);
$signingKey = hash_hmac('sha256', 's3', $signingKey, true);
$signingKey = hash_hmac('sha256', 'aws4_request', $signingKey, true);
$signature = hash_hmac('sha256', $policy, $signingKey);
```
**Ref**
- https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html

### Upload file
1. Need to generate authorization signature using S3 key and secret. We will be using Signature V4 which is the latest at the moment
2. create url using bucket and region (<bucket>.s3.<region>.amazonaws.com)
3. Create and execute the request in the following format

```http
PUT /my-image.jpg HTTP/1.1
Host: myBucket.s3.<Region>.amazonaws.com
Date: Wed, 12 Oct 2009 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-Length: 11434
x-amz-meta-author: Janet
Expect: 100-continue
[11434 bytes of object data]
```
**Ref**         
- https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html
- https://www.h3xed.com/programming/php-amazon-s3-file-upload-code-aws-signature-version-4


### Delete file
1. Need to generate authorization signature using S3 key and secret. We will be using Signature V4 which is the latest at the moment
2. create url using bucket and region (<bucket>.s3.<region>.amazonaws.com)
3. Create and execute the request in the following format

```http
DELETE /my-second-image.jpg HTTP/1.1
Host: <bucket>.s3.<Region>.amazonaws.com
Date: Wed, 12 Oct 2009 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain        
```
https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteObject.html#API_DeleteObject_RequestSyntax

### Environment variables
- _APP_STORAGE_DEVICE=local/s3, select default storage device
- _APP_STORAGE_DEVICE_S3_ACCESS_KEY=AWS access key ID
- _APP_STORAGE_DEVICE_S3_SECRET: AWS secret key
- _APP_STORAGE_DEVICE_S3_REGION: S3 Region
- _APP_STORAGE_DEVICE_S3_BUCKET: S3 bucket

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

1. With S3 adapter we can support other S3 compatible services like Digitalocean spaces
2. Once we have multiple adapters, we can allow support multiple storage adapters at once, later introduce features like storage bucket. So that Appwrite itself can be used as an interface to multiple storage platforms
3. We can also support multiple buckets features with S3 and other compatible services
4. We can also later support file versioning features of S3 and other compatible services
