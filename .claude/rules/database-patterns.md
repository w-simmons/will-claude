---
paths:
  - "lib/db/schemas/**"
  - "**/_services/**/*.ts"
priority: high
---

# Database Patterns

**Auto-applies to**: Database schemas, services with DB operations

## Purpose

This rule covers database development patterns using Drizzle ORM, including schema creation, queries, migrations, and type safety.

## Key Principles

1. **Per-app schemas** in `lib/db/schemas/`
2. **Use Drizzle query patterns** consistently
3. **Type safety** with Drizzle inference
4. **Relations** for complex queries

## Schema Creation

### File Location

Schemas live in `lib/db/schemas/[app-slug].ts`

### Basic Schema Pattern

```typescript
import {
  pgTable,
  uuid,
  text,
  varchar,
  integer,
  boolean,
  timestamp,
} from 'drizzle-orm/pg-core'
import { relations } from 'drizzle-orm'

export const myTable = pgTable('app_my_app_items', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  email: varchar('email', { length: 255 }),
  isActive: boolean('is_active').default(true),
  count: integer('count').default(0),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

// Type exports
export type MyItem = typeof myTable.$inferSelect
export type NewMyItem = typeof myTable.$inferInsert
```

### Schema with Relations

```typescript
export const chatConversations = pgTable('chat_conversations', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull(),
  title: varchar('title', { length: 255 }),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export const chatConversationsRelations = relations(chatConversations, ({ many }) => ({
  messages: many(chatMessages),
}))

export const chatMessages = pgTable('chat_messages', {
  id: uuid('id').primaryKey().defaultRandom(),
  conversationId: uuid('conversation_id').notNull().references(() => chatConversations.id, { onDelete: 'cascade' }),
  role: varchar('role', { length: 20 }).notNull(), // 'user' | 'assistant'
  content: text('content').notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
})

export const chatMessagesRelations = relations(chatMessages, ({ one }) => ({
  conversation: one(chatConversations, {
    fields: [chatMessages.conversationId],
    references: [chatConversations.id],
  }),
}))
```

### Register Schema

Export from `lib/db/schema.ts`:

```typescript
// lib/db/schema.ts
export * from './schemas/chat'
export * from './schemas/my-app' // Add your schema here
```

## Query Patterns

### Basic Queries

```typescript
import { db, eq, desc, and, like } from '@/lib/db'
import { myTable } from '@/lib/db/schemas/my-app'

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
  .values({ name: 'Test', email: 'test@example.com' })
  .returning()

// Update
const [updated] = await db
  .update(myTable)
  .set({
    name: 'New name',
    updatedAt: new Date()
  })
  .where(eq(myTable.id, id))
  .returning()

// Delete
await db
  .delete(myTable)
  .where(eq(myTable.id, id))
```

### Filtered Queries

```typescript
// Single condition
const active = await db
  .select()
  .from(myTable)
  .where(eq(myTable.isActive, true))

// Multiple conditions
const filtered = await db
  .select()
  .from(myTable)
  .where(
    and(
      eq(myTable.isActive, true),
      like(myTable.name, '%test%')
    )
  )
  .orderBy(desc(myTable.createdAt))
```

### Queries with Relations

```typescript
// Using query API (recommended for relations)
const conversations = await db.query.chatConversations.findMany({
  where: eq(chatConversations.userId, userId),
  orderBy: [desc(chatConversations.createdAt)],
  with: {
    messages: {
      orderBy: [desc(chatMessages.createdAt)],
      limit: 10,
    },
  },
})

// Single item with relations
const conversation = await db.query.chatConversations.findFirst({
  where: eq(chatConversations.id, id),
  with: {
    messages: true,
  },
})
```

### Complex Queries

```typescript
// Get with counts
const conversations = await db.query.chatConversations.findMany({
  with: {
    messages: true,
  },
})

const conversationsWithCounts = conversations.map(conversation => ({
  ...conversation,
  messageCount: conversation.messages.length,
}))

// Get latest for each
const items = await db.query.myTable.findMany()
const itemIds = items.map(i => i.id)

const latestEvents = await db.query.myEvents.findMany({
  where: and(
    eq(myEvents.type, 'update'),
    inArray(myEvents.itemId, itemIds)
  ),
  orderBy: [desc(myEvents.date)],
})

// Build lookup map
const eventMap = new Map<string, { type: string; date: string }>()
for (const event of latestEvents) {
  if (!eventMap.has(event.itemId)) {
    eventMap.set(event.itemId, {
      type: event.type,
      date: event.date,
    })
  }
}
```

## Type Safety

### Infer Types from Schema

```typescript
// Select type (read)
export type MyItem = typeof myTable.$inferSelect

// Insert type (write)
export type NewMyItem = typeof myTable.$inferInsert

// Use in services
export async function createItem(
  data: Omit<NewMyItem, 'id' | 'createdAt' | 'updatedAt'>
): Promise<MyItem> {
  const [item] = await db
    .insert(myTable)
    .values(data)
    .returning()

  return item
}
```

### Extended Types

```typescript
// Base type
export type ChatConversation = typeof chatConversations.$inferSelect

// Extended type with relations
export type ConversationWithMessages = ChatConversation & {
  messages: ChatMessage[]
  messageCount?: number
}
```

## Migration Workflow

### Development

```bash
# Push schema changes to database (development)
pnpm db:push

# Generate migration files
pnpm db:generate

# Run migrations
pnpm db:migrate

# Open Drizzle Studio (visual browser)
pnpm db:studio
```

### After Schema Changes

1. Update schema file in `lib/db/schemas/[app].ts`
2. Run `pnpm db:push` (development) or `pnpm db:generate` (production)
3. Verify in Drizzle Studio: `pnpm db:studio`

## Common Patterns

### Service Pattern with Database

```typescript
import { db, eq, desc } from '@/lib/db'
import { myTable, type MyItem, type NewMyItem } from '@/lib/db/schemas/my-app'

export const myService = {
  async getAll(): Promise<MyItem[]> {
    if (!db) throw new Error('Database not configured')

    return await db
      .select()
      .from(myTable)
      .orderBy(desc(myTable.createdAt))
  },

  async get(id: string): Promise<MyItem | null> {
    if (!db) throw new Error('Database not configured')

    const [item] = await db
      .select()
      .from(myTable)
      .where(eq(myTable.id, id))

    return item ?? null
  },

  async create(
    data: Omit<NewMyItem, 'id' | 'createdAt' | 'updatedAt'>
  ): Promise<MyItem> {
    if (!db) throw new Error('Database not configured')

    const [item] = await db
      .insert(myTable)
      .values(data)
      .returning()

    return item
  },

  async update(
    id: string,
    data: Partial<Omit<NewMyItem, 'id' | 'createdAt'>>
  ): Promise<MyItem> {
    if (!db) throw new Error('Database not configured')

    const [updated] = await db
      .update(myTable)
      .set({
        ...data,
        updatedAt: new Date(),
      })
      .where(eq(myTable.id, id))
      .returning()

    return updated
  },

  async delete(id: string): Promise<void> {
    if (!db) throw new Error('Database not configured')

    await db.delete(myTable).where(eq(myTable.id, id))
  },
}
```

## Common Mistakes

### ❌ Don't Forget to Check Database Configuration

```typescript
// ❌ Bad: No check
export async function getItems() {
  return await db.select().from(myTable)
}

// ✅ Good: Check configuration
export async function getItems() {
  if (!db) throw new Error('Database not configured')
  return await db.select().from(myTable)
}
```

### ❌ Don't Use `.returning()` Incorrectly

```typescript
// ❌ Bad: Missing .returning()
const result = await db.insert(myTable).values(data)
// result is undefined!

// ✅ Good: Use .returning() to get inserted row
const [item] = await db
  .insert(myTable)
  .values(data)
  .returning()
```

### ❌ Don't Forget updatedAt

```typescript
// ❌ Bad: Missing updatedAt
await db
  .update(myTable)
  .set({ name: 'New name' })
  .where(eq(myTable.id, id))

// ✅ Good: Always update updatedAt
await db
  .update(myTable)
  .set({
    name: 'New name',
    updatedAt: new Date(),
  })
  .where(eq(myTable.id, id))
```

## References

- See `.claude/skills/database.md` for detailed database patterns
- Drizzle ORM Docs: https://orm.drizzle.team
- See `lib/db/schemas/chat.ts` for comprehensive schema example

---

**Last Verified**: 2025-12-29
