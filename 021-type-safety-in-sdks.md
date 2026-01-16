# Type Safety in SDKs

- Implementation Owner: @ChiragAgg5k
- Start Date: 15-01-2026
- Target Date: 26-01-2026
- Appwrite Issue:
  No specific issue.

## Summary

[summary]: #summary

Appwrite's SDKs are technically typed but not type safe. There are places where Type Safety does not exist completely, or only partially exists. This RFC proposes various changes not just in the SDKs itself, but in the CLI to help generate a workflow which is completely end-to-end type safe.

We are going to take inspiration from projects like Prisma and Drizzle which are very popular in the community for their type safety and ease of use.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Appwrite's SDKs are only partially typed. For eg. `databaseId` and `tableId` cannot be typed and need to made sure that correct values are passed by the developer.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

### Workflow decisions

1. **Appwrite init**: A complete workflow to initially setup an appwrite project using the CLI. Not just the project (which is what `appwrite init project` does).
2. **Typed Appwrite config**: Currently appwrite config is plain json, we can make it fully typed with zod.
2. **Push/Pull Sync**: Push and Pull commands will sync your entire configuration (currently it does this in a semi-automated manner).
3. **Generate**: `appwrite generate` will create a `generated` folder (can be a customized by param) that exposes a fully typed SDK for the project.

### API Endpoints

No new API endpoints are required.

### Data Structure

We will follow the exisiting appwrite config's data structure:

```json
{
  "projectId": "6839a26e003262977966",
  "projectName": "Testing Project",
  "endpoint": "https://fra.cloud.appwrite.io/v1",
  "tablesDB": [
    {
      "$id": "test-db",
      "name": "Testing Database",
      "$createdAt": "2026-01-15T05:47:28.583+00:00",
      "$updatedAt": "2026-01-15T05:47:28.583+00:00",
      "enabled": true,
      "type": "tablesdb",
      "policies": [],
      "archives": []
    }
  ],
  "tables": [
    {
      "$id": "users",
      "$permissions": [],
      "databaseId": "test-db",
      "name": "users",
      "enabled": true,
      "rowSecurity": false,
      "columns": [
        {
          "key": "username",
          "type": "string",
          "status": "available",
          "error": "",
          "required": true,
          "array": false,
          "$createdAt": "2026-01-15T05:49:30.850+00:00",
          "$updatedAt": "2026-01-15T05:49:31.196+00:00",
          "size": 32,
          "default": null,
          "encrypt": false
        }
      ]
    }
  ]
}
```

Structure of the typed sdk will be as follows:

```text
generated/
├── appwrite.databases.ts
├── appwrite.auth.ts
├── appwrite.storage.ts
├── appwrite.functions.ts
├── appwrite.buckets.ts
├── appwrite.messaging.ts
├── appwrite.teams.ts
├── appwrite.types.ts
├── appwrite.ts
```

Usage -

```typescript
import { databases } from './generated/appwrite';

const db = databases.from('test-db'); // <-- typed out database options the user has

await db.users.create({ username: 'testuser' });
await db.users.get('6968e1d100160eb1a115');
await db.users.update('6968e1d100160eb1a115', { username: 'testuser2' });
await db.users.delete('6968e1d100160eb1a115');
await db.users.list({ limit: 10, offset: 0 });
await db.users.listWithTotal({ limit: 10, offset: 0 });


await db.users.createMany([{ username: 'testuser3' }, { username: 'testuser4' }]);
await db.users.updateMany([{ id: '6968e1d100160eb1a115', username: 'testuser2' }, { id: '6968e1d100160eb1a116', username: 'testuser3' }]);
await db.users.deleteMany(['6968e1d100160eb1a115', '6968e1d100160eb1a116']);

await db.users.upsert('6968e1d100160eb1a115', { username: 'testuser2' });
```

The command will auto detect if server sdk is being used or not (with ability to manually select either), and generate server side methods too -

```typescript
await db.create({ id: 'books', name: 'Books' });
await db.update('books', { name: 'Books Table' });
await db.delete('books');
```

### Supporting Libraries

No new libraries required, uses existing:
- appwrite-cli
- typescript sdks

### Breaking Changes

No breaking changes. Existing type safety works as it is. Users who want the type-safe version can opt in by using the new command.

### Reliability (Tests & Benchmarks)

#### Scaling

Code generation runs locally via CLI - no server-side scaling concerns. Generation time is negligible (< 1 second) for typical projects.

#### Benchmarks

N/A - client-side code generation with no performance-critical paths.

#### Tests (UI, Unit, E2E)

- Unit tests for type generation from various `appwrite.json` configurations
- E2E tests verifying generated SDK compiles and works against live Appwrite instance
- Type tests using `tsd` to verify autocomplete and type errors

### Documentation & Content

- Add "Type-Safe SDK Generation" page under SDK docs
- Update CLI reference with `appwrite generate` command
- Blog post showcasing developer experience
- TypeScript starter template with type generation pre-configured

### Prior art

[prior-art]: #prior-art

- [Prisma](https://www.prisma.io/) - `prisma generate` creates typed client from schema file
- [Supabase](https://supabase.com/docs/guides/api/rest/generating-types) - `supabase gen types typescript` generates types from database
- [Drizzle](https://orm.drizzle.team/) - TypeScript-native ORM with inferred types

### Unresolved questions

[unresolved-questions]: #unresolved-questions

- Output directory naming: `generated/`, `appwrite-generated/`, or `.appwrite/`?
- Multi-language support priority order (Dart, Python, Swift/Kotlin)?
- How to handle relationship types - nested objects or IDs?
- Handling of table/column names that conflict with TypeScript reserved words?

### Future possibilities

[future-possibilities]: #future-possibilities

- Type-safe query builders with typed filters and field selection
- Type-safe real-time subscriptions
- Generate Zod schemas for runtime validation
- Watch mode for auto-regeneration on config changes
- IDE extensions for auto-generation
