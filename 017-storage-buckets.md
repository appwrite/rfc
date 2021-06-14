# Storage Buckets

- Implementation Owner: @torstendittmann
- Start Date: 26-05-2021
- Target Date: (expected date of completion, dd-mm-yyyy)
- Appwrite Issue: https://github.com/appwrite/appwrite/issues/363

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
We are adding Buckets to our Storage service. These will allow users to create containers around multiple files. Buckets will allow users to enforce logic around Files. The mentioned logic involves custom permissions and settings.
## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Developers want to organize the files and their storage and have dedicated destinations with different Policies for specific use-cases.

**What is the context or background in which this problem exists?**

Right now there is no way of configuring Policies or Permissions for multiple Files. This results into developers not being able to configure access to the Storage Service.

**Once the proposal is implemented, how will the system change?**

The Storage API is going to have additional routes - that will allow configuring Buckets for the Server Side and uploading files to specific Buckets on the Client Side.

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
You can use buckets to organize your storage and control access to your files, but unlike directories and folders (which are not part of the Storage API anyway), you cannot nest buckets.

In the Storage Service, Buckets are going to act like a folder that will add logic to how the API responds. A Bucket has following configurations available:
- **Enabled**
  - This setting decides if a Bucket is enabled and is accessible.
- **Adapter**
  - This decides to what Storage Adapter the files will be uploaded to. Possible options are Local, S3, DigitaOcean Spaces, etc. This is not allowed to be changed after a file has been added to the bucket.
- **Maximum File Size Limit**
  - This setting decides the maximum single file size that can be uploaded.
- **Maximum Number of Files**
  - This setting decides how many files a Bucket can contain.
- **Accepted File Extensions**
  - This setting decides what file extensions can be uploaded.
- **Encryption**
  - This setting decides if files are encrypted. This will come into play, when encrypting large files causes too much load. This is not allowed to be changed after a file has been added to the bucket.
- **Virus Scan**
  - This setting decides if files are scanned by ClamAV after upload. Same reasoning as **Encryption**.
- **TTL**
  - This setting decides how long a file is supposed to alive. Files are going to be deleted automatically. This is not allowed to be changed after a file has been added to the bucket.
- **Permission**
  - This setting decides what permission role is allowed to upload files to the bucket. Read and Write permissions will still exist for each file and be the source of permission check.

A Bucket will have following Model in our internal Database:
```typescript
{
  $id: string;
  $write: string[];
  name: string;
  enabled: boolean;
  adapter: 'local'|'s3'|'etc'
  encrypted: boolean;
  antivirus: boolean;
  ttl?: number;
  maximumFiles?: number;
  maximumFileSize?: number;
  allowedFileExtensions?: string;
}
```

In terms of a storage adapter except `Local`, additional values will be present like configurations for S3.

### New Endpoints

In this section I will explain all necessary endpoints for integratting Storge Buckets.

#### `GET: /v1/storage/buckets/`

This endpoint lists all Buckets. *Server*

Payload:
- **search = ''** Search term to filter your list results.
- **limit = 25** Maximum number of Buckets to return.
- **offset = 0** Offset value.

#### `GET: /v1/storage/buckets/{bucketId}`

This endpoint returns a single Bucket by its unique ID. *Server*

#### `GET: /v1/storage/buckets/{bucketId}/files`

This endpoint returns a list of all Files in a specific Bucket. *Server & Client*

Payload:
- **filters = []** Array of filter strings.
- **limit = 25** Maximum number of Buckets to return.
- **offset = 0** Offset value.
- **search = ''** Search query.

#### `POST: /v1/storage/buckets/{adapter}`

This endpoint creates a new Bucket. *Server*

Payload:
- **name** Bucket name.
- **write** An array of strings with write permissions.
- **encrypted = true** Enable Encryption.
- **antivirus = true** Enable Anti Virus.
- **ttl = false** Enables TTL for files.
- **maxFiles = null** Maximum amount of files.
- **maxFileSize = null** Maximum file size.
- **fileExtensions = null** Allowed File extensions. Leave empty to allow any extension.

Additional Payload for **Local** (`/v1/storage/buckets/local`):
- *none*

Additional Payload for **S3** (`/v1/storage/buckets/s3`):
- **accessKeyId** S3 Access Key ID.
- **secretKey** S3 Secret Key.
- **bucket** S3 Bucket Name.
- **region** S3 Region.
- **acl** S3 Access Control List.

#### `PUT: /v1/storage/buckets/{bucketId}`

This endpoint updates a Bucket by its unique ID. *Server*

Payload:
- **name = null** Bucket name.
- **write = null** An array of strings with write permissions.
- **encrypted = null** Enable Encryption.
- **antivirus = null** Enable Anti Virus.
- **adapter = null** Adapter to use.
- **adapterConfig = null** Adapter Config.
- **ttl = null** TTL in seconds for files created.
- **maximumFiles = null** Maximum amount of files.
- **maximumFileSize = null** Maximum file size.
- **allowedFileExtensions = null** Allowed File extensions.

#### `DELETE: /v1/storage/buckets/{bucketId}`

This endpoint deletes a Bucket by its unique ID. *Server*

> This endpoint should trigger a background delete of all files in the bucket from both db and storage.

#### `POST: /v1/storage/buckets/{bucketId}/files`

This endpoint creates a new file. This endpoints will be equal to the already existing one - but will act in a Buckets scope.

Additional Payload:
- **ttl = null** This parameter will be the TTL of the file in seconds.

#### `GET: /v1/storage/buckets/{bucketId}/files/{fileId}`
#### `GET: /v1/storage/buckets/{bucketId}/files/{fileId}/preview`
#### `GET: /v1/storage/buckets/{bucketId}/files/{fileId}/download`
#### `GET: /v1/storage/buckets/{bucketId}/files/{fileId}/view`

These endpoints gets a file by its unique ID. These endpoints will be equal to the already existing ones - but will act in a Buckets scope.

### Initial implementation

The first implementation will not contain following features:
- TTL
- Maximum Number of files

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
- [About Storage Buckets](https://docs.uipath.com/orchestrator/docs/about-storage-buckets)
- [Google Cloud Storage](https://cloud.google.com/storage/docs/json_api/v1/buckets)
- [Google Cloud Storage - Key Terms](https://cloud.google.com/storage/docs/key-terms)

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

#### Prevent Breaking Changes

To prevent a huge breaking change in the Storage Services API's - we are allowing users to use the current Storage implementation as well. Now there is the problem of offering a consistent API pattern across the services.

Let's take a look at the `GET: /v1/storage/files/{fileId}` endpoint. This will result into retrieving a file from the current Storage service. Now if a file is inside a Bucket - the endpoint would have to look like `GET: /v1/storage/files/{bucketId}/{fileId}`. This becomes problematic, since right now it is not possible to allow optional Parameters in the Path with the same Controller/SDK Method and would require refactoring in all SDKs, Utopia and Appwrite.

Possible solutions can be the following:

- Add the targeted Bucket as a query param to `GET: /v1/storage/files/default/{fileId}?bucket={bucketId}`.
  - This breaks our API Standards - but could be a good tradeoff in terms of the migration for develoeprs, since there will be no breaking change.
- Adding a keyword passing to the Bucket parameter that falls back to the default Storage `GET: /v1/storage/files/default/{fileId}`.
  - This can be done without introducing any breaking changes to the SDK usage
- Allowing to do calls to `GET: /v1/storage/files/YYYY/XXXX` and `GET: /v1/storage/files//XXXX`
  - Not a 100% sure this is compliant with any REST Standards
- Having 2 separate endpoints for each, the default Storage and Storage Buckets, with its own SDK methods. **This is the currently implemented solution for this RFC**

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

- The ability to switch to a different Adapter with existing files - meaning all files will be transfered.
- The ability to move files between Buckets
- The ability to change the encryption setting with existing files - meaning all files can be converted.
