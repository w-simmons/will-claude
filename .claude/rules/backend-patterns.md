---
paths:
  - "**/actions.ts"
  - "**/_services/**"
  - "app/api/**"
priority: normal
---

# Backend Patterns

**Auto-applies to**: Server Actions, services, API routes

## Purpose

This rule covers backend development patterns including Server Actions, service layer architecture, error handling, and cache revalidation.

## Key Principles

1. **Server Actions** in `actions.ts` — thin, just validation + service call
2. **Business logic** in `_services/` — portable, no Next.js deps
3. **Return** `{ data }` or `{ error }` from actions
4. **Always use `revalidatePath`** after mutations

## Server Actions Structure

### File Location

Server Actions live in `app/apps/[slug]/actions.ts`

### Basic Pattern

```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { myService } from './_services/my.service'

const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function createItem(formData: FormData) {
  // 1. Parse and validate input
  const parsed = schema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  })

  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  // 2. Call service
  try {
    const item = await myService.create(parsed.data)

    // 3. Revalidate cache
    revalidatePath('/apps/my-app')

    // 4. Return success
    return { data: item }
  } catch (error) {
    console.error('createItem error:', error)
    return { error: 'Failed to create item' }
  }
}
```

### With Object Input

```typescript
export async function createItem(input: z.infer<typeof schema>) {
  const parsed = schema.safeParse(input)
  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  try {
    const item = await myService.create(parsed.data)
    revalidatePath('/apps/my-app')
    return { data: item }
  } catch (error) {
    console.error('createItem error:', error)
    return { error: 'Failed to create item' }
  }
}
```

### Update Pattern

```typescript
export async function updateItem(id: string, formData: FormData) {
  const parsed = schema.partial().safeParse({
    name: formData.get('name') || undefined,
    email: formData.get('email') || undefined,
  })

  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  try {
    const item = await myService.update(id, parsed.data)
    revalidatePath('/apps/my-app')
    revalidatePath(`/apps/my-app/${id}`)
    return { data: item }
  } catch (error) {
    console.error('updateItem error:', error)
    return { error: 'Failed to update item' }
  }
}
```

### Delete Pattern

```typescript
export async function deleteItem(id: string) {
  try {
    await myService.delete(id)
    revalidatePath('/apps/my-app')
    return { success: true }
  } catch (error) {
    console.error('deleteItem error:', error)
    return { error: 'Failed to delete item' }
  }
}
```

### Read Pattern (No Mutation)

```typescript
export async function getItems() {
  try {
    const items = await myService.getAll()
    return { data: items }
  } catch (error) {
    console.error('getItems error:', error)
    return { error: 'Failed to get items' }
  }
}
```

## Service Layer Patterns

### File Location

Services live in `app/apps/[slug]/_services/[name].service.ts`

### Core Rule: Portable Services

**Services must be portable** — no Next.js dependencies!

```typescript
// ✅ Good: Portable service
import { db, eq } from '@/lib/db'
import { myTable } from '@/lib/db/schemas/my-app'
// No Next.js imports!

export const myService = {
  async create(data: { name: string; email: string }) {
    if (!db) throw new Error('Database not configured')

    const [item] = await db
      .insert(myTable)
      .values(data)
      .returning()

    return item
  },

  async getAll() {
    if (!db) throw new Error('Database not configured')

    return await db.select().from(myTable)
  },

  async update(id: string, data: Partial<{ name: string; email: string }>) {
    if (!db) throw new Error('Database not configured')

    const [updated] = await db
      .update(myTable)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(myTable.id, id))
      .returning()

    return updated
  },

  async delete(id: string) {
    if (!db) throw new Error('Database not configured')

    await db.delete(myTable).where(eq(myTable.id, id))
  },
}
```

### Service with Relations

```typescript
export const itemService = {
  async getItems(): Promise<ItemWithRelations[]> {
    if (!db) throw new Error('Database not configured')

    const items = await db.query.myItems.findMany({
      orderBy: [desc(myItems.createdAt)],
      with: {
        category: true,
        tags: {
          limit: 10,
        },
      },
    })

    return items.map(item => ({
      ...item,
      category: item.category ? {
        id: item.category.id,
        name: item.category.name,
      } : null,
      tags: item.tags ?? [],
    }))
  },
}
```

## Error Handling

### Return Value Pattern

**Always return `{ data }` or `{ error }`**

```typescript
// ✅ Good: Consistent return pattern
export async function getItem(id: string) {
  try {
    const item = await myService.get(id)
    if (!item) {
      return { error: 'Item not found' }
    }
    return { data: item }
  } catch (error) {
    console.error('getItem error:', error)
    return { error: 'Failed to get item' }
  }
}
```

### Error Handling in Actions

```typescript
export async function createItem(formData: FormData) {
  try {
    // Validation
    const parsed = schema.safeParse({
      name: formData.get('name'),
    })

    if (!parsed.success) {
      return { error: 'Invalid input' }
    }

    // Service call
    const item = await myService.create(parsed.data)

    // Revalidation
    revalidatePath('/apps/my-app')

    return { data: item }
  } catch (error) {
    // Log error for debugging
    console.error('createItem error:', error)

    // Return user-friendly error
    return { error: 'Failed to create item' }
  }
}
```

### Service Error Handling

```typescript
export const myService = {
  async create(data: { name: string }) {
    // Check configuration
    if (!db) throw new Error('Database not configured')

    try {
      const [item] = await db
        .insert(myTable)
        .values(data)
        .returning()

      return item
    } catch (error) {
      // Log and rethrow for action to handle
      console.error('Service create error:', error)
      throw error
    }
  },
}
```

## revalidatePath Usage

### When to Revalidate

**Always use `revalidatePath` after mutations:**

- After create → revalidate list page
- After update → revalidate list page + detail page
- After delete → revalidate list page

### Examples

```typescript
// Create
export async function createItem(formData: FormData) {
  const item = await myService.create(parsed.data)
  revalidatePath('/apps/my-app') // Revalidate list
  return { data: item }
}

// Update
export async function updateItem(id: string, formData: FormData) {
  const item = await myService.update(id, parsed.data)
  revalidatePath('/apps/my-app') // Revalidate list
  revalidatePath(`/apps/my-app/${id}`) // Revalidate detail
  return { data: item }
}

// Delete
export async function deleteItem(id: string) {
  await myService.delete(id)
  revalidatePath('/apps/my-app') // Revalidate list
  return { success: true }
}
```

### Multiple Paths

```typescript
export async function updateItem(id: string, formData: FormData) {
  const item = await myService.update(id, parsed.data)

  // Revalidate all affected pages
  revalidatePath('/apps/my-app')
  revalidatePath(`/apps/my-app/${id}`)
  revalidatePath('/apps/my-app/dashboard')

  return { data: item }
}
```

## Common Mistakes

### ❌ Don't Put Business Logic in Actions

```typescript
// ❌ Bad: Business logic in action
export async function createItem(formData: FormData) {
  const data = parseFormData(formData)

  // Business logic here - BAD!
  const processed = data.map(item => {
    // Complex processing...
  })

  await db.insert(myTable).values(processed)
  revalidatePath('/apps/my-app')
  return { data: processed }
}

// ✅ Good: Business logic in service
export async function createItem(formData: FormData) {
  const parsed = schema.safeParse({
    name: formData.get('name'),
  })

  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  const item = await myService.create(parsed.data) // Service handles logic
  revalidatePath('/apps/my-app')
  return { data: item }
}
```

### ❌ Don't Import Next.js in Services

```typescript
// ❌ Bad: Next.js import in service
import { revalidatePath } from 'next/cache' // NO!

// ✅ Good: No Next.js imports in services
import { db, eq } from '@/lib/db'
```

### ❌ Don't Skip Error Handling

```typescript
// ❌ Bad: No error handling
export async function createItem(formData: FormData) {
  const item = await myService.create(parsed.data)
  return { data: item }
}

// ✅ Good: Proper error handling
export async function createItem(formData: FormData) {
  try {
    const parsed = schema.safeParse({
      name: formData.get('name'),
    })

    if (!parsed.success) {
      return { error: 'Invalid input' }
    }

    const item = await myService.create(parsed.data)
    revalidatePath('/apps/my-app')
    return { data: item }
  } catch (error) {
    console.error('createItem error:', error)
    return { error: 'Failed to create item' }
  }
}
```

### ❌ Don't Forget revalidatePath

```typescript
// ❌ Bad: Missing revalidatePath
export async function createItem(formData: FormData) {
  const item = await myService.create(parsed.data)
  return { data: item } // Cache won't update!
}

// ✅ Good: Always revalidate after mutations
export async function createItem(formData: FormData) {
  const item = await myService.create(parsed.data)
  revalidatePath('/apps/my-app')
  return { data: item }
}
```

## References

- See `.claude/skills/forms.md` for form integration
- See `CLAUDE.md` for architecture details
- Next.js Server Actions: https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions

---

**Last Verified**: 2025-12-29
