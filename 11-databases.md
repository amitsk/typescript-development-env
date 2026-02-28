# Chapter 11: Database Libraries

[← Previous: Full-Stack with TanStack Start](./10-tanstack-start.md) | [Back to README](./README.md) | [Next: CI/CD →](./12-ci-cd.md)

---

Your API or full-stack app can only do so much with in-memory arrays. Real applications need to persist data — and that means a database. This chapter covers how to connect a TypeScript/Node.js application to a database, and introduces three popular libraries that make database access type-safe and ergonomic: **Drizzle ORM**, **Prisma**, and **Kysely**.

## Database Access in Node.js

There are three levels of abstraction you can work at when talking to a database from Node.js:

### Level 1: Raw SQL Drivers

Packages like `pg` (PostgreSQL), `mysql2` (MySQL), and `better-sqlite3` (SQLite) let you send raw SQL strings directly to the database.

```typescript
import { Pool } from 'pg';
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// You write raw SQL — no type safety
const result = await pool.query('SELECT * FROM users WHERE id = $1', [1]);
const user = result.rows[0]; // typed as `any`
```

This gives you complete control and zero overhead, but there is no type safety. `result.rows[0]` is `any`, so a typo in a column name will only fail at runtime. You also have to write every SQL string yourself, including migrations.

### Level 2: Query Builders

A query builder like **Kysely** provides a TypeScript-friendly API for constructing SQL queries. You still think in SQL terms (select, from, where, join), but the library adds TypeScript types throughout. If you misspell a column name, TypeScript catches it before you run your code.

### Level 3: ORMs

An Object-Relational Mapper (ORM) like **Drizzle** or **Prisma** goes further: you define your data model in TypeScript (or a special schema language), and the ORM generates queries, handles migrations, and provides a high-level API. The tradeoff is more abstraction — sometimes the ORM generates inefficient SQL for complex queries, and you need to understand the abstraction to debug it.

### Which Level Should You Choose?

| You want... | Consider... |
|-------------|-------------|
| Maximum control, writing all SQL yourself | Raw driver (`pg`, `mysql2`) |
| Type-safe queries, still writing SQL | Kysely |
| TypeScript-first schema + auto migrations + clean API | Drizzle ORM |
| Fastest possible start, great DX, rich ecosystem | Prisma |

In practice, most TypeScript projects today use either Drizzle or Prisma. Kysely is popular in performance-sensitive applications and among developers who want to stay close to SQL.

---

## Drizzle ORM

[Drizzle ORM](https://orm.drizzle.team) ([GitHub](https://github.com/drizzle-team/drizzle-orm)) describes itself as "SQL-first". Its query API looks and feels like SQL, but with TypeScript types on everything. The schema is defined in plain TypeScript files — there is no separate schema language to learn — and that schema is the single source of truth for both your TypeScript types and your database migrations.

Drizzle is lightweight, has no hidden runtime magic, and supports PostgreSQL, MySQL, SQLite, and several other databases.

### Installation

For a PostgreSQL project:

```bash
pnpm add drizzle-orm pg
pnpm add -D drizzle-kit @types/pg
```

`drizzle-orm` is the runtime library. `drizzle-kit` is the CLI tool for generating and running migrations.

### Defining Your Schema

Create `src/db/schema.ts`. This file defines your tables as TypeScript objects:

```typescript
import { pgTable, serial, varchar, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  bio: text('bio'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

Each call like `varchar('name', { length: 255 })` defines both the database column (Drizzle knows it is a `VARCHAR(255)`) and the TypeScript type (`string`). Drizzle infers the full TypeScript type of a row from this definition — no separate interface needed.

### Connecting to the Database

Create `src/db/index.ts`:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

export const db = drizzle(pool, { schema });
```

The `DATABASE_URL` environment variable should look like: `postgresql://user:password@localhost:5432/mydb`.

Passing `{ schema }` to `drizzle()` enables relational query helpers (covered in the Drizzle docs) — it is a good habit to include it from the start.

### Type-Safe CRUD Queries

```typescript
import { db } from './db';
import { users } from './db/schema';
import { eq } from 'drizzle-orm';

// Select all users
// Type: { id: number; name: string; email: string; bio: string | null; createdAt: Date }[]
const allUsers = await db.select().from(users);

// Select with a where clause
const alice = await db
  .select()
  .from(users)
  .where(eq(users.email, 'alice@example.com'));

// Insert a new user — TypeScript enforces required fields
const newUser = await db
  .insert(users)
  .values({
    name: 'Charlie',
    email: 'charlie@example.com',
  })
  .returning();  // returns the inserted row with generated id and createdAt

// Update
await db
  .update(users)
  .set({ name: 'Charles' })
  .where(eq(users.id, 1));

// Delete
await db.delete(users).where(eq(users.id, 1));
```

The `eq` import is a helper for `=` comparisons. Drizzle exports similar helpers for other operators: `ne` (not equal), `gt` (greater than), `like`, `inArray`, `and`, `or`, and more.

If you try to insert a row without a required field, or use a column name that does not exist in the schema, TypeScript will give you a compile-time error — not a runtime crash.

### Migrations with Drizzle Kit

Create `drizzle.config.ts` in the project root:

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

Then use the Drizzle Kit CLI:

```bash
pnpm drizzle-kit generate   # compare schema.ts to the database and generate SQL migration files
pnpm drizzle-kit migrate    # apply pending migration files to the database
pnpm drizzle-kit studio     # open a visual database browser at http://localhost:4983
```

The workflow is: modify `schema.ts` → run `generate` → review the generated SQL → run `migrate`. The generated SQL files are committed to version control, giving you a full history of every schema change.

---

## Prisma

[Prisma](https://www.prisma.io) ([GitHub](https://github.com/prisma/prisma)) is the most widely used Node.js ORM, known for its excellent developer experience. Unlike Drizzle (which defines the schema in TypeScript), Prisma uses its own **Prisma Schema Language** — a purpose-built DSL for describing your data model.

From the `.prisma` schema file, Prisma generates a fully type-safe JavaScript/TypeScript client. The generated client includes types for every table, every query result, and every input — so your editor gives you perfect autocomplete and catch errors before you run anything.

### Installation

```bash
pnpm add prisma @prisma/client
pnpm prisma init
```

`pnpm prisma init` creates a `prisma/` directory with a starter `schema.prisma` file and adds a `DATABASE_URL` placeholder to a new `.env` file.

### Defining Your Schema

Edit `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  bio       String?
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        Int    @id @default(autoincrement())
  title     String
  content   String
  author    User   @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

The Prisma Schema Language is declarative and easy to read. `String?` means optional (nullable). `Post[]` on `User` and the `@relation` on `Post` define the one-to-many relationship between users and posts.

### Generating the Client and Running Migrations

```bash
pnpm prisma generate       # reads schema.prisma and generates the TypeScript client into node_modules/@prisma/client
pnpm prisma migrate dev    # creates a new migration file and applies it to the database
pnpm prisma studio         # open a visual database browser at http://localhost:5555
```

You run `prisma generate` any time you change the schema. Run `prisma migrate dev` during development to keep the database in sync.

### Connecting and Using Prisma Client

Create `src/db.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

export const prisma = new PrismaClient();
```

In a production application, you should instantiate `PrismaClient` as a singleton to avoid creating too many database connections. The [Prisma documentation](https://www.prisma.io/docs/guides/performance-and-optimization/connection-management) has a recommended pattern for this.

### CRUD Queries

```typescript
import { prisma } from './db';

// Find all users
const users = await prisma.user.findMany();

// Find one user by email, including their posts (SQL JOIN)
const user = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
  include: { posts: true },
});

// Create a new user
const newUser = await prisma.user.create({
  data: { name: 'Alice', email: 'alice@example.com' },
});

// Create a user and a post in one nested write
const userWithPost = await prisma.user.create({
  data: {
    name: 'Bob',
    email: 'bob@example.com',
    posts: {
      create: { title: 'Hello World', content: 'My first post.' },
    },
  },
  include: { posts: true },
});

// Update a user
await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' },
});

// Delete a user
await prisma.user.delete({ where: { id: 1 } });
```

Prisma's API is notably readable. The nested `posts: { create: {...} }` syntax for creating related records in a single query is a particularly nice touch. The return types are all inferred automatically — `prisma.user.findUnique(...)` returns `User | null`, and `prisma.user.findUnique({ include: { posts: true } })` returns `User & { posts: Post[] } | null`.

---

## Kysely

[Kysely](https://kysely.dev) ([GitHub](https://github.com/kysely-org/kysely)) takes a fundamentally different philosophy from Drizzle and Prisma. It is not an ORM — it is a **type-safe SQL query builder**. Kysely does not generate queries for you or manage your schema. Instead, it gives you a fluent TypeScript API for constructing SQL queries, with full type inference throughout.

If you enjoy writing SQL and just want TypeScript types on top of it, Kysely is an excellent choice. There is no abstraction layer between your code and the SQL being executed — what you write is exactly what runs.

### Installation

For PostgreSQL:

```bash
pnpm add kysely pg
pnpm add -D @types/pg
```

### Defining Your Database Types

Kysely does not read your database schema automatically. You define the TypeScript types for your tables manually in a file:

Create `src/db/types.ts`:

```typescript
import { Generated, ColumnType } from 'kysely';

// Represents the 'users' table
export interface UserTable {
  id: Generated<number>;          // auto-generated by the database
  name: string;
  email: string;
  bio: string | null;
  created_at: ColumnType<Date, never, never>;  // read-only: can select but not insert/update
}

// The Database interface maps table names to their row types
export interface Database {
  users: UserTable;
}
```

`Generated<T>` tells Kysely the column is auto-generated (like a serial primary key) — it is present when you read a row but optional when you insert one. `ColumnType<SelectType, InsertType, UpdateType>` gives you fine-grained control over what is allowed in each operation.

### Creating the Kysely Instance

Create `src/db/index.ts`:

```typescript
import { Kysely, PostgresDialect } from 'kysely';
import { Pool } from 'pg';
import type { Database } from './types';

const dialect = new PostgresDialect({
  pool: new Pool({
    connectionString: process.env.DATABASE_URL,
  }),
});

export const db = new Kysely<Database>({ dialect });
```

The `Database` generic parameter is what gives every query its type safety. Kysely reads the table types from `Database` to infer what columns exist, what types they have, and what operations are valid.

### Type-Safe Queries

```typescript
import { db } from './db';

// Select all users
// Return type is inferred as: { id: number; name: string; email: string; bio: string | null; created_at: Date }[]
const users = await db.selectFrom('users').selectAll().execute();

// Select specific columns
const names = await db
  .selectFrom('users')
  .select(['id', 'name'])
  .execute();
// Return type: { id: number; name: string }[]

// Select with a where clause
const user = await db
  .selectFrom('users')
  .selectAll()
  .where('email', '=', 'alice@example.com')
  .executeTakeFirst();
// Return type: { id: number; ... } | undefined

// Insert a new row
const newUser = await db
  .insertInto('users')
  .values({ name: 'Charlie', email: 'charlie@example.com', bio: null })
  .returningAll()
  .executeTakeFirstOrThrow();

// Update
await db
  .updateTable('users')
  .set({ name: 'Charles' })
  .where('id', '=', 1)
  .execute();

// Delete
await db.deleteFrom('users').where('id', '=', 1).execute();
```

If you try to use a column name that does not exist in `UserTable`, or pass the wrong type for a column value, TypeScript will report an error immediately. There are no runtime surprises from typos.

`executeTakeFirst()` returns `T | undefined` (the first row or undefined if no results). `executeTakeFirstOrThrow()` throws if no row is found. `execute()` always returns an array.

### Migrations

Kysely does not include a migration system. For migrations you have a few options:

- Write plain `.sql` files and apply them manually or with a simple script
- Use a dedicated migration library like `db-migrate` or `umzug`
- Use Kysely's built-in [migration API](https://kysely.dev/docs/migrations) — it manages migration state in a `kysely_migration` table but you write the SQL (or Kysely queries) for each migration yourself

This is a deliberate design choice: Kysely keeps its scope narrow so you can combine it with whatever migration strategy fits your project.

---

## Comparison Table

Here is a detailed side-by-side comparison of all three libraries:

| Dimension | Drizzle ORM | Prisma | Kysely |
|-----------|-------------|--------|--------|
| **Abstraction level** | ORM (SQL-first) | ORM (schema-first) | Query builder |
| **Schema definition** | TypeScript files | Prisma Schema Language (`.prisma`) | TypeScript interfaces (manual) |
| **Type safety** | Excellent — inferred from schema | Excellent — generated from schema | Excellent — inferred from your types |
| **Migration support** | Built-in (`drizzle-kit`) | Built-in (`prisma migrate`) | Not included — bring your own |
| **Bundle size** | Very small (~35kb) | Larger (includes query engine binary) | Small (~30kb) |
| **Learning curve** | Low–Medium | Low | Medium (requires SQL knowledge) |
| **Visual DB browser** | Drizzle Studio | Prisma Studio | Not included |
| **Raw SQL escape hatch** | `db.execute(sql\`...\`)` | `prisma.$queryRaw` | You are already writing SQL |
| **Relations/joins** | Supported (relational query API) | Excellent (nested includes) | Manual joins (full SQL control) |
| **Best for** | SQL-savvy devs wanting TypeScript-first ORM | Rapid prototyping, teams, rich ecosystem | Complex queries, max SQL control |

---

## Which Should You Choose?

All three libraries are production-ready and widely used. The choice comes down to your priorities and working style.

**Choose Drizzle if** you are comfortable with SQL and want a TypeScript-first ORM that stays close to the database. Drizzle gives you the benefits of an ORM (schema definition, migrations, type-safe queries) without hiding what SQL is being executed. It is particularly well-suited for projects that need to stay lean and performant. The schema-as-TypeScript approach means no context switching between languages.

**Choose Prisma if** you want the fastest path from zero to a working database layer, or if you are working on a team where not everyone has deep SQL knowledge. Prisma's schema language is easy to read and write, the generated client API is beautiful, and the ecosystem around Prisma is rich — there are adapters for many databases and a large community. The slightly heavier runtime footprint is rarely a problem in practice.

**Choose Kysely if** you want maximum control over the SQL being executed, have complex query requirements (deep joins, window functions, CTEs), or are working on a performance-critical application where you need to reason precisely about every query. Kysely is also a great choice when you want to gradually add TypeScript types to an existing application that already has a raw `pg` or `mysql2` setup.

## What's Next?

You now have everything you need to build a working full-stack TypeScript application: a bundler (Vite), an API framework (Fastify) or full-stack framework (TanStack Start), and a database library. The final piece of a professional workflow is automation: making sure your code is tested and deployed reliably every time you push a change. That is what CI/CD, covered in the next chapter, is all about.

---

[← Previous: Full-Stack with TanStack Start](./10-tanstack-start.md) | [Back to README](./README.md) | [Next: CI/CD →](./12-ci-cd.md)
