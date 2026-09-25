# Folder Support for storage Buckets

- Implementation Owner: @loks0n
- Start Date: 06-10-2025
- Target Date: TBD
- Appwrite Issue: TBD

## Summary

[summary]: #summary

Add folder support to Appwrite Storage for organizing files. Folders are lightweight organizational containers - they don't have independent permissions, don't support nesting initially, and exist purely to help users sort and filter files within buckets.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Large buckets become unmanageable. Users with hundreds or thousands of files have no way to organize them. Every file appears in a flat list. Finding specific files requires searching by name or using complex filtering.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Design decisions

1. **Folders are organizational** - No independent permissions, compression, or encryption settings
2. **Flat structure initially** - No nested folders (folders can't contain folders)
3. **Backward compatible** - Existing files stay at root level
4. **Minimal overhead** - Folders are lightweight documents, not filesystem constructs

### API Endpoints

**POST /v1/storage/buckets/:bucketId/folders**
Create a folder.
```
bucketId: string (required)
folderId: string (required) - Custom ID or ID.unique()
name: string (required) - Folder name (max 255 chars)
permissions: array (optional) - Inherits bucket permissions if null
```

**GET /v1/storage/buckets/:bucketId/folders**
List folders in a bucket.
```
bucketId: string (required)
queries: array (optional) - Standard query support
search: string (optional)
```

**GET /v1/storage/buckets/:bucketId/folders/:folderId**
Get folder by ID.

**PUT /v1/storage/buckets/:bucketId/folders/:folderId**
Update folder name/permissions.

**DELETE /v1/storage/buckets/:bucketId/folders/:folderId**
Delete folder. Fails if folder contains files unless `force=true`.

**Modified: POST /v1/storage/buckets/:bucketId/files**
Add `folderId` parameter (optional, defaults to null = root level).

**Modified: GET /v1/storage/buckets/:bucketId/files**
Add `folderId` query parameter to filter files by folder.

### Data Structure

**New: Folders collection**
Stored in same `bucket_{sequence}` collection as files, differentiated by type.

```php
// Folder document
[
    '$id' => 'unique_folder_id',
    'type' => 'folder', // discriminator field
    'bucketId' => 'bucket_id',
    'bucketInternalId' => 123,
    'name' => 'Invoices',
    '$permissions' => [...], // same as bucket unless overridden
    'search' => 'folder_id Invoices',
    'filesCount' => 0, // cached count
]

// Modified file document - add single field
[
    // ... all existing fields ...
    'folderId' => 'folder_id_or_null', // null = root level
]
```

**Required indexes:**
- `(bucketInternalId, type, name)` - unique constraint for folder names
- `(bucketInternalId, folderId)` - list files in folder
- `(bucketInternalId, type)` - list all folders

**Migration:**
1. Add `folderId` field to all existing file documents (default null)
2. Add `type` field to all existing file documents (default 'file')
3. Create indexes
4. Deploy

### Implementation Details

**Creating a folder:**
1. Validate bucket exists and user has CREATE permission
1. Check for duplicate folder name in bucket (unique constraint)
1. Generate ID if unique() passed
1. Inherit bucket permissions if none specified
1. Create folder document with type discriminator
1. Return folder

**Listing files with folder filter:**
1. If folderId param present:
    1. folderId='root' → query where folderId IS NULL
    1. Otherwise → verify folder exists, query where folderId = X
1. Apply standard queries/pagination
1. Return filtered files

**Deleting folder:**
1. Check folder exists and user has DELETE permission
1. Count files in folder
1. If files exist and force=false → error FOLDER_NOT_EMPTY
1. If force=true → set all child files' folderId to null (move to root)
1. Delete folder document

### Supporting Libraries

No new libraries required. Uses existing:
- Utopia Database for folder documents
- Existing validation/authorization stack
- Current storage device layer (untouched)

### Breaking Changes

**None.** Fully backward compatible:
- Migration applies existing files have `folderId = null` (root level)
- New optional parameters don't affect existing API calls

### Reliability (Tests & Benchmarks)

#### Benchmarks

Measure:
- Folder creation latency
- File listing with/without folder filter
- Moving files between folders
- Deleting folder with 1000+ files

#### Tests (UI, Unit, E2E)

**Unit tests:**
- Create folder with/without custom ID
- Duplicate folder name prevention
- List folders with queries/search
- Delete empty folder
- Delete folder with force flag
- File upload to folder
- List files filtered by folder
- Move file between folders

**E2E tests:**
- Create bucket → create folder → upload file → list by folder
- Delete folder with files (should fail)
- Delete folder with force (files move to root)
- Permission inheritance from bucket
- Search across folders

**Console UI:**
- Folder view in bucket files list
- Create folder button
- Drag-drop files into folders
- Breadcrumb navigation
- Folder delete confirmation

### Documentation & Content

**Docs needed:**
1. API reference for new folder endpoints
1. SDK examples for folder operations

### Prior art

[prior-art]: #prior-art

**AWS S3:** Uses prefixes, not real folders. Files with `/` in name create virtual hierarchy. Works but confusing - users expect folders to be entities they can rename/list.

**Google Drive API:** Folders are files with special MIME type. Can nest infinitely. Complex permission inheritance. Over-engineered for basic use cases.

**Dropbox API:** Clear folder vs file distinction. Simple listing. Nesting supported but not required. Our pattern is similiar.

**Firebase Storage:** Prefix-based like S3. No folder metadata. Awkward for apps that need folder operations.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

1. **Folder permissions:** Initially inherit from bucket. Future: independent folder permissions?
2. **Nested folders:** Defer or include v1? Adds complexity (validation, infinite loops, path resolution).
3. **Moving folders:** If we add nesting later, do we support moving folders between folders?
4. **Folder metadata:** Do folders need custom metadata like files? Size limits?

### Future possibilities

[future-possibilities]: #future-possibilities

**Phase 2 - Nested folders:**
- Add `parentFolderId` to folder documents
- Validate against circular references
- Path reconstruction for breadcrumbs
- Max depth limit (e.g., 10 levels)

**Phase 3 - Advanced features:**
- Folder-level permissions (override bucket)
- Bulk operations (move all files in folder)
- Folder templates (create with predefined structure)
- Shared folders (special permission model)
- Folder hooks/events for automation

**Phase 4 - Performance:**
- Materialized paths for fast hierarchy queries
- Denormalized file counts (already included)
- Folder statistics (total size, last modified)
