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

### Implementation Status

An initial working version is available:

```bash
npm install -g appwrite-cli@13.1.0
```

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
      "enabled": true
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
          "required": true,
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
└── appwrite/
    ├── databases.ts
    ├── types.ts
    └── index.ts
    └── constants.ts
```

The generator auto-detects which Appwrite SDK is being used by checking `package.json` dependencies:
- `node-appwrite` - Server SDK (supports bulk methods)
- `appwrite` - Client SDK
- `react-native-appwrite` - React Native SDK
- `@appwrite.io/console` - Console SDK (supports bulk methods)
- `npm:node-appwrite` - Deno (supports bulk methods)

### Multi-Language Support

The `appwrite generate` command will support multiple languages beyond TypeScript:
- **Phase 1**: TypeScript/JavaScript (Node.js, Browser, React Native, Deno)
- **Phase 2**: Dart/Flutter, Python, Swift, Kotlin

Language detection via config files (`pubspec.yaml`, `requirements.txt`, etc.), explicit `--language` flag, or project structure analysis. Each language will have idiomatic generator templates.

### CLI and SDK Bundling

Exploring bundling strategies to improve DX:
- **Hybrid Approach (Recommended)**: CLI remains standalone; SDK packages include lightweight generator utilities. CLI uses SDK type definitions for validation.
- **Alternatives**: CLI as SDK dependency, standalone with integration hooks, or monorepo workspace support.

### Enhanced Type Safety

The generated code will include:
- Type-safe column names in queries (autocomplete for valid column keys)
- Type-safe relationship traversal (typed foreign key lookups)
- Type-safe enum/select column values (literal types for allowed values)
- Type-safe permissions with callback pattern (typed permission and role builders)
- Type-safe query builder with field-specific filter methods

Usage -

```typescript
import { databases } from "./generated/appwrite/index.js";
import { CategoriesColor } from "./generated/appwrite/types.js";

const USER_ID = "685ba24de34c453eb58c";

const mydb = databases.use("main-db"); // <-- typed database options
const categoriesTable = mydb.use("categories"); // <-- typed table options

// Basic CRUD operations with type-safe permissions callback
const category = await categoriesTable.create({
    name: "Category Name",
    slug: "category-name-slug",
    description: "Category Description",
    color: CategoriesColor.F5FF33, // <-- typed enum values
    icon: "category-icon",
    sortOrder: 1,
    isActive: true,
    posts: ["c4d70ad8-6f0a-4947-a0de-e034098602a5"]
}, {
    permissions: (permission, role) => [
        permission.read(role.user(USER_ID)),
        permission.update(role.user(USER_ID)),
        permission.delete(role.user(USER_ID)),
    ]
});

await categoriesTable.get("6968e1d100160eb1a115");
await categoriesTable.update("6968e1d100160eb1a115", { name: "Updated Name" });
await categoriesTable.delete("6968e1d100160eb1a115");

// Type-safe queries with callback pattern
await categoriesTable.list({
    queries: (query) => [
        query.equal("description", "Category Description"),
        query.greaterThan("sortOrder", 0),
        query.limit(10),
        query.offset(0),
    ]
});

// Bulk operations (available with node-appwrite, npm:node-appwrite, @appwrite.io/console)
// Note: Bulk operations are NOT available for tables with relationship columns

// createMany - create multiple rows at once
await categoriesTable.createMany([
    { name: "Category 1", slug: "cat-1", color: CategoriesColor.F5FF33 },
    { name: "Category 2", slug: "cat-2", color: CategoriesColor.F5FF33, $id: "custom-id" },
], { transactionId: "tx-123" });

// updateMany - update all rows matching query with same data
await categoriesTable.updateMany(
    { isActive: true },
    {
        queries: (query) => [query.equal("isActive", false)],
        transactionId: "tx-123"
    }
);

// deleteMany - delete all rows matching query
await categoriesTable.deleteMany({
    queries: (query) => [query.equal("isActive", false)],
    transactionId: "tx-123"
});

// Optional parameters for single operations
await categoriesTable.create(
    { name: "Test", slug: "test", color: CategoriesColor.F5FF33 },
    {
        rowId: "custom-id",
        permissions: (permission, role) => [
            permission.read(role.any()),
            permission.write(role.users()),
        ],
        transactionId: "tx-123"
    }
);
```

### Supporting Libraries

No new libraries required, uses existing:
- appwrite-cli
- typescript sdks

### Breaking Changes

No breaking changes. Existing type safety works as it is. Users who want the type-safe version can opt in by using the new command.

### Deprecations

The `appwrite types` command will be deprecated in favor of `appwrite generate`. The new command provides a more comprehensive solution with full type-safe SDK generation, not just type definitions. Migration path: `appwrite types` → `appwrite generate --types-only` (if needed) or full `appwrite generate`.

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
