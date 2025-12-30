# Skill: Database Patterns
<!-- Triggers: schema, migration, Drizzle, database, db schema, queries -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- Schemas in `lib/db/schemas/[app].ts`
- Register in `lib/db/schema.ts`
- Use `pnpm db:push` after schema changes
- Use `pnpm db:studio` to browse data
- Always use typed queries with Drizzle

## Common Patterns

### Creating a Schema

```typescript
// lib/db/schemas/[app].ts
import {
  pgTable,
  uuid,
  text,
  varchar,
  timestamp,
  boolean,
  integer
} from 'drizzle-orm/pg-core'

export const myTable = pgTable('app_[app]_items', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  description: text('description'),
  isActive: boolean('is_active').default(true),
  count: integer('count').default(0),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export type MyItem = typeof myTable.$inferSelect
export type NewMyItem = typeof myTable.$inferInsert
```

### Registering Schema

```typescript
// lib/db/schema.ts
export * from './schemas/agents'
export * from './schemas/foot-measure'
export * from './schemas/[app]'  // Add this line
```

### Common Queries

```typescript
import { db, eq, desc, and, or, like } from '@/lib/db'
import { myTable } from '@/lib/db/schemas/[app]'

// Get all
const items = await db.select().from(myTable)

// Get by ID
const [item] = await db
  .select()
  .from(myTable)
  .where(eq(myTable.id, id))

// Insert
const [newItem] = await db
  .insert(myTable)
  .values({ name: 'Test' })
  .returning()

// Update
const [updated] = await db
  .update(myTable)
  .set({ name: 'New name', updatedAt: new Date() })
  .where(eq(myTable.id, id))
  .returning()

// Delete
await db
  .delete(myTable)
  .where(eq(myTable.id, id))

// Filter with multiple conditions
const filtered = await db
  .select()
  .from(myTable)
  .where(
    and(
      eq(myTable.isActive, true),
      like(myTable.name, '%search%')
    )
  )
  .orderBy(desc(myTable.createdAt))
```

## Templates

### Complete Service with Database

```typescript
// app/apps/[slug]/_services/my.service.ts
import { db, eq, desc } from '@/lib/db'
import { myTable, type NewMyItem } from '@/lib/db/schemas/[slug]'

export const myService = {
  async getAll() {
    if (!db) throw new Error('Database not configured')
    return db.select().from(myTable).orderBy(desc(myTable.createdAt))
  },

  async getById(id: string) {
    if (!db) throw new Error('Database not configured')
    const [item] = await db.select().from(myTable).where(eq(myTable.id, id))
    return item
  },

  async create(data: NewMyItem) {
    if (!db) throw new Error('Database not configured')
    const [item] = await db.insert(myTable).values(data).returning()
    return item
  },

  async update(id: string, data: Partial<NewMyItem>) {
    if (!db) throw new Error('Database not configured')
    const [item] = await db
      .update(myTable)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(myTable.id, id))
      .returning()
    return item
  },

  async delete(id: string) {
    if (!db) throw new Error('Database not configured')
    await db.delete(myTable).where(eq(myTable.id, id))
  },
}
```

## After Schema Changes

```bash
# Push schema to database (development)
pnpm db:push

# Browse database visually
pnpm db:studio
```

## Common Field Types

- `uuid('id')` - UUID primary key
- `varchar('name', { length: 255 })` - Variable character
- `text('description')` - Long text
- `boolean('is_active')` - True/false
- `integer('count')` - Whole number
- `timestamp('created_at')` - Date/time
- `jsonb('metadata')` - JSON data

## References

- See `.claude/rules/database-patterns.md` for comprehensive patterns
- See existing schemas in `lib/db/schemas/` for examples
- Drizzle docs: https://orm.drizzle.team/docs/overview
