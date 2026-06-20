---
name: drizzle
description: "Drizzle ORM conventions for Cloudflare D1 (SQLite) in Nuxt v4 Nitro projects. Use when defining schemas, writing migrations, querying D1, or setting up drizzle-kit. Companion to the cloudflare-pages skill."
---

# Drizzle ORM — D1 (SQLite) Conventions

Drizzle ORM is the preferred ORM for Cloudflare D1 in bigin fullstack projects. It provides type-safe SQL with a thin abstraction layer and first-class D1 support.

---

## Installation

```bash
pnpm add drizzle-orm
pnpm add -D drizzle-kit
```

---

## File Structure

```
server/
  database/
    schema.ts          ← table definitions (single file unless large)
    index.ts           ← db helper (creates Drizzle client from D1 binding)
    migrations/        ← generated SQL migration files (committed)
drizzle.config.ts      ← drizzle-kit config (project root)
```

---

## Schema Definition

```typescript
// server/database/schema.ts
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core'

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),                          // use ULID or UUID
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  createdAt: integer('created_at', { mode: 'timestamp' })
    .$defaultFn(() => new Date()),
})

export const posts = sqliteTable('posts', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  body: text('body').notNull(),
  authorId: text('author_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  createdAt: integer('created_at', { mode: 'timestamp' })
    .$defaultFn(() => new Date()),
})
```

### Type conventions

| SQLite type | Drizzle helper | Use for |
|-------------|---------------|---------|
| `TEXT` | `text()` | strings, IDs, enums |
| `INTEGER` | `integer()` | numbers, booleans (0/1), timestamps |
| `REAL` | `real()` | floats |
| `BLOB` | `blob()` | binary data |

Use `integer('col', { mode: 'timestamp' })` for dates — stored as Unix epoch, returned as `Date`.

---

## DB Client Helper

```typescript
// server/database/index.ts
import { drizzle } from 'drizzle-orm/d1'
import * as schema from './schema'

export function useDB(env: Env) {
  return drizzle(env.DB, { schema })
}
```

Usage in any Nitro handler:

```typescript
// server/api/users/index.get.ts
export default defineEventHandler(async (event) => {
  const { DB } = event.context.cloudflare.env
  const db = useDB({ DB })

  const users = await db.select().from(schema.users).all()
  return users
})
```

---

## drizzle.config.ts

```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  schema: './server/database/schema.ts',
  out: './server/database/migrations',
  dialect: 'sqlite',
  driver: 'd1-http',
  dbCredentials: {
    accountId: process.env.CLOUDFLARE_ACCOUNT_ID!,
    databaseId: process.env.CLOUDFLARE_DATABASE_ID!,
    token: process.env.CLOUDFLARE_D1_TOKEN!,
  },
})
```

> **Note:** `d1-http` driver is only used by `drizzle-kit` for migrations. The runtime always uses the D1 binding directly.

---

## Migrations Workflow

```bash
# 1. Generate SQL migration from schema changes
pnpm drizzle-kit generate

# 2. Apply to local D1 (via Wrangler)
wrangler d1 execute your-db-name --local \
  --file=server/database/migrations/0001_init.sql

# 3. Apply to production D1
wrangler d1 execute your-db-name \
  --file=server/database/migrations/0001_init.sql
```

Always commit the generated `.sql` files in `server/database/migrations/`.

---

## Common Queries

```typescript
import { eq, and, desc, like } from 'drizzle-orm'
import * as schema from '~/server/database/schema'

// Select all
const all = await db.select().from(schema.users).all()

// Select one by ID
const user = await db.select()
  .from(schema.users)
  .where(eq(schema.users.id, id))
  .get()

// Insert
await db.insert(schema.users).values({
  id: crypto.randomUUID(),
  email: 'user@example.com',
  name: 'Alice',
})

// Update
await db.update(schema.users)
  .set({ name: 'Bob' })
  .where(eq(schema.users.id, id))

// Delete
await db.delete(schema.users)
  .where(eq(schema.users.id, id))

// Filter + order + limit
const recent = await db.select()
  .from(schema.posts)
  .where(eq(schema.posts.authorId, userId))
  .orderBy(desc(schema.posts.createdAt))
  .limit(10)
  .all()

// Join
const postsWithAuthor = await db.select({
  postId: schema.posts.id,
  title: schema.posts.title,
  author: schema.users.name,
})
  .from(schema.posts)
  .innerJoin(schema.users, eq(schema.posts.authorId, schema.users.id))
  .all()
```

---

## ID Convention

Use `crypto.randomUUID()` (available in Cloudflare Workers without import):

```typescript
id: crypto.randomUUID(),
```

For ordered IDs, use `ulid` package:

```bash
pnpm add ulid
```

```typescript
import { ulid } from 'ulid'
id: ulid(),
```

---

## wrangler.toml — D1 Binding

```toml
[[d1_databases]]
binding = "DB"
database_name = "your-db-name"
database_id = "YOUR_DATABASE_ID"
```

The binding name `DB` must match what's used in `useDB(env.DB)`.

---

## Env Type Augmentation

Declare the `Env` type so TypeScript knows about D1:

```typescript
// server/types/cloudflare.d.ts
interface Env {
  DB: D1Database
  STORAGE?: R2Bucket
  KV?: KVNamespace
}
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `DB is not defined` | Check `binding = "DB"` in `wrangler.toml` matches exactly |
| Migration not applied locally | Run with `--local` flag; check `.wrangler/state/v3/d1/` |
| `drizzle-kit generate` produces no output | Schema file path in `drizzle.config.ts` must match actual file |
| Type errors on query result | Use `.get()` for single row (returns `T \| undefined`), `.all()` for arrays |
| `crypto.randomUUID is not a function` | Cloudflare Workers have this built-in; in tests use `import { randomUUID } from 'crypto'` |
