---
priority: low
---

# App Creation Guide

**Manual Invocation** - Reference when creating new apps, or use `/new-app` command

## Purpose

This rule provides a complete step-by-step guide for creating new apps in the monorepo.

**Quick Option**: Use `/new-app [slug] [name]` command for automated scaffolding.

## Quick Start Checklist

1. ✅ Create `app/apps/[slug]/page.tsx`
2. ✅ Create `app/apps/[slug]/README.md` (required!)
3. ✅ Create `app/apps/[slug]/layout.tsx` (optional, for theme)
4. ✅ Create `app/apps/[slug]/actions.ts` (if needed)
5. ✅ Create theme file `styles/themes/[slug].css` (optional)
6. ✅ Create database schema `lib/db/schemas/[slug].ts` (if needed)
7. ✅ Update `config/apps.json` (optional, for global nav)

## Step 1: Create the Page

Create `app/apps/[slug]/page.tsx`:

```tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

export default function MyAppPage() {
  return (
    <div className="p-6 md:p-8 lg:p-12">
      <div className="mx-auto max-w-6xl space-y-8">
        <h1 className="text-2xl font-bold tracking-tight">My App</h1>

        <Card>
          <CardHeader>
            <CardTitle>Section Title</CardTitle>
          </CardHeader>
          <CardContent>
            <Button>Action</Button>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
```

## Step 2: Create README (Required!)

Create `app/apps/[slug]/README.md`:

```markdown
# My App

**Purpose**: Brief description of what this app does

## Features

- Feature 1
- Feature 2
- Feature 3

## Architecture

### Tech Stack
- Next.js 15 (Server Components)
- Drizzle ORM (if using database)
- Shadcn UI components

### Key Patterns
- Server Actions for mutations
- Services for business logic
- [App-specific patterns]

## Implementation Details

[Describe key technical decisions, data structures, algorithms, etc.]
```

## Step 3: Create Layout (Optional, for Theme)

Create `app/apps/[slug]/layout.tsx`:

```tsx
import '@/styles/globals.css'
import '@/styles/themes/my-app.css'

export default function MyAppLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div data-theme="my-app" className="min-h-screen bg-background">
      <main className="container mx-auto px-4 py-6">
        {children}
      </main>
    </div>
  )
}
```

## Step 4: Create Theme (Optional)

Create `styles/themes/[slug].css`:

```css
[data-theme="my-app"] {
  --primary: oklch(0.55 0.2 200);
  --primary-foreground: oklch(0.98 0 0);
  --accent: oklch(0.65 0.15 180);
  --accent-foreground: oklch(0.98 0 0);
  --background: oklch(0.99 0 0);
  --foreground: oklch(0.15 0 0);
}
```

**OKLCH Color Guide:**
- Lightness: 0-1 (0 = black, 1 = white)
- Chroma: 0-0.4 (0 = gray, higher = saturated)
- Hue: 0-360 (color wheel)

## Step 5: Create Actions (If Needed)

Create `app/apps/[slug]/actions.ts`:

```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { myService } from './_services/my.service'

const schema = z.object({
  name: z.string().min(1),
})

export async function createItem(formData: FormData) {
  const parsed = schema.safeParse({
    name: formData.get('name'),
  })

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

## Step 6: Create Service (If Needed)

Create `app/apps/[slug]/_services/my.service.ts`:

```typescript
import { db, eq } from '@/lib/db'
import { myTable } from '@/lib/db/schemas/my-app'
// No Next.js imports!

export const myService = {
  async create(data: { name: string }) {
    if (!db) throw new Error('Database not configured')

    const [item] = await db
      .insert(myTable)
      .values(data)
      .returning()

    return item
  },
}
```

## Step 7: Create Database Schema (If Needed)

Create `lib/db/schemas/[slug].ts`:

```typescript
import {
  pgTable,
  uuid,
  text,
  varchar,
  timestamp,
} from 'drizzle-orm/pg-core'

export const myTable = pgTable('app_my_app_items', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export type MyItem = typeof myTable.$inferSelect
export type NewMyItem = typeof myTable.$inferInsert
```

**Register schema** in `lib/db/schema.ts`:

```typescript
export * from './schemas/my-app' // Add this line
```

**Push schema** to database:

```bash
pnpm db:push
```

## Step 8: Create Components (As Needed)

Create `app/apps/[slug]/_components/MyComponent.tsx`:

```tsx
'use client' // Only if needed

import { Card } from '@/components/ui/card'

export function MyComponent() {
  return (
    <Card>
      <p>Component content</p>
    </Card>
  )
}
```

## Step 9: Register in Global Nav (Optional)

Add entry to `config/apps.json` (only if you want global navigation):

```json
{
  "slug": "my-app",
  "name": "My App",
  "description": "What my app does",
  "icon": "IconName",
  "category": "apps",
  "theme": "my-app"
}
```

**Fields:**
- `slug`: URL path (`/apps/my-app`)
- `name`: Display name
- `description`: Short description
- `icon`: Lucide React icon name (e.g., "MessageSquare", "Image")
- `category`: `"apps"` or `"utilities"`
- `theme`: Optional theme name (defaults to slug)

## Complete Example: Minimal App

**1. Page:**
```tsx
// app/apps/notes/page.tsx
export default function NotesPage() {
  return (
    <div className="container py-8">
      <h1 className="text-2xl font-bold">Notes</h1>
    </div>
  )
}
```

**2. README:**
```markdown
// app/apps/notes/README.md
# Notes

**Purpose**: Simple note-taking app

## Features
- Create notes
- View notes list

## Architecture
- Server Components only (no client state)
```

**3. Verify:**
- Route loads at `/apps/notes`
- README provides context for Claude Code

## Common Patterns

### App with Database

1. Create schema in `lib/db/schemas/[slug].ts`
2. Register in `lib/db/schema.ts`
3. Run `pnpm db:push`
4. Create service in `_services/`
5. Create actions in `actions.ts`
6. Use in page/components

### App with Forms

1. Create form component in `_components/`
2. Use `react-hook-form` + `zod`
3. Create server action in `actions.ts`
4. Use `toast` for feedback

### App with Theme

1. Create theme CSS in `styles/themes/[slug].css`
2. Import in `layout.tsx`
3. Apply `data-theme` attribute
4. Use CSS variables in components

## Verification Checklist

- [ ] Route loads at `/apps/[slug]`
- [ ] README.md exists with full context
- [ ] Theme applies correctly (if added)
- [ ] Database schema works (if added)
- [ ] Forms submit correctly (if added)
- [ ] No console errors

## Automation

**Use `/new-app` command** for automated scaffolding:

```bash
/new-app my-app "My App Name" "apps"
```

This will:
- Create all required files
- Generate README template
- Set up directory structure
- Optionally create theme and schema

## References

- See `.claude/skills/add-app.md` for detailed guide
- See `.claude/commands/new-app.md` for automation
- See `CLAUDE.md` for architecture
- See existing apps in `app/apps/` for examples

---

**Last Verified**: 2025-12-29
