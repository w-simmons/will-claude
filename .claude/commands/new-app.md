---
name: "new-app"
description: "Scaffold a complete new app with all required files and structure"
arguments:
  - name: "slug"
    description: "URL-friendly app identifier (e.g., 'my-app')"
    required: true
  - name: "name"
    description: "Display name for the app (e.g., 'My App')"
    required: true
  - name: "category"
    description: "App category: 'apps' or 'utilities'"
    required: false
---

# Command: new-app

Create a complete app scaffolding with all required files and structure.

## Usage

```bash
/new-app [slug] [name] [category]
```

**Examples:**
- `/new-app todo-list "Todo List" "apps"`
- `/new-app calculator "Calculator" "utilities"`
- `/new-app notes "Notes"`

## What This Command Does

Creates a complete app structure with:

1. **Page** - `app/apps/[slug]/page.tsx`
2. **README** - `app/apps/[slug]/README.md` (REQUIRED)
3. **Layout** - `app/apps/[slug]/layout.tsx` (optional, for theme)
4. **Actions** - `app/apps/[slug]/actions.ts` (if needed)
5. **Directories** - `_services/`, `_components/` (if needed)
6. **Theme** - `styles/themes/[slug].css` (optional)
7. **Schema** - `lib/db/schemas/[slug].ts` (if needed)
8. **Config** - Updates `config/apps.json` (optional)

## Implementation Steps

### 1. Validate Arguments

- `slug`: Must be kebab-case, no spaces, lowercase
- `name`: Can be any display name
- `category`: Defaults to "apps" if not provided

### 2. Prompt for Additional Options

Ask user:
- **Create theme?** (yes/no)
- **Need database?** (yes/no)
- **Add to global nav?** (yes/no)

Use `AskUserQuestion` tool for interactive prompts.

### 3. Create Required Files

**Always create:**

#### `app/apps/[slug]/page.tsx`

```tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

export default function [PascalCaseName]Page() {
  return (
    <div className="p-6 md:p-8 lg:p-12">
      <div className="mx-auto max-w-6xl space-y-8">
        <h1 className="text-2xl font-bold tracking-tight">[name]</h1>

        <Card>
          <CardHeader>
            <CardTitle>Get Started</CardTitle>
          </CardHeader>
          <CardContent>
            <p>Welcome to [name]!</p>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
```

#### `app/apps/[slug]/README.md`

```markdown
# [name]

**Purpose**: [Brief description - prompt user for this]

## Features

- Feature 1 (coming soon)
- Feature 2 (coming soon)

## Goals

- [Short-term goal - prompt user]
- [Long-term vision - prompt user]

## Current Work / Iteration Areas

- Initial setup and scaffolding

## Known Issues / Needs Work

- [ ] Complete initial implementation

## Tech Stack

- Next.js 15 (Server Components)
- Shadcn UI components
[- Drizzle ORM (if database)]
[- Custom theme (if theme)]

## Architecture

- **Pages**: Main app interface in `page.tsx`
- **Components**: App-specific UI in `_components/`
- **Services**: Business logic in `_services/`
- **Actions**: Server Actions in `actions.ts`

## Usage Notes

- Route: `/apps/[slug]`
[- Theme: `data-theme="[slug]"` (if theme)]
```

### 4. Create Optional Files

#### If theme: `app/apps/[slug]/layout.tsx`

```tsx
import '@/styles/globals.css'
import '@/styles/themes/[slug].css'

export default function [PascalCaseName]Layout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div data-theme="[slug]" className="min-h-screen bg-background">
      <main className="container mx-auto px-4 py-6">
        {children}
      </main>
    </div>
  )
}
```

#### If theme: `styles/themes/[slug].css`

```css
[data-theme="[slug]"] {
  /* Primary color - adjust as needed */
  --primary: oklch(0.55 0.2 [hue]);
  --primary-foreground: oklch(0.98 0 0);

  /* Accent color - adjust as needed */
  --accent: oklch(0.65 0.15 [hue + 20]);
  --accent-foreground: oklch(0.98 0 0);

  /* Background */
  --background: oklch(0.99 0 0);
  --foreground: oklch(0.15 0 0);
}
```

Prompt user for preferred hue (0-360, suggest random or based on app type).

#### If database: `lib/db/schemas/[slug].ts`

```typescript
import {
  pgTable,
  uuid,
  varchar,
  text,
  timestamp,
} from 'drizzle-orm/pg-core'

export const [camelCase]Table = pgTable('app_[slug]_items', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export type [PascalCase]Item = typeof [camelCase]Table.$inferSelect
export type New[PascalCase]Item = typeof [camelCase]Table.$inferInsert
```

Then update `lib/db/schema.ts`:
```typescript
export * from './schemas/[slug]'
```

Then run: `pnpm db:push`

#### If global nav: Update `config/apps.json`

```json
{
  "slug": "[slug]",
  "name": "[name]",
  "description": "[Ask user for description]",
  "icon": "[Suggest icon based on app type, or ask user]",
  "category": "[category]",
  "theme": "[slug]"
}
```

### 5. Create Directories

Always create:
- `app/apps/[slug]/_components/` (empty, for future use)
- `app/apps/[slug]/_services/` (empty, for future use)

### 6. Verify and Report

- ✅ Confirm all files created
- ✅ List created files with paths
- ✅ Provide next steps:
  - Visit `/apps/[slug]` to see app
  - Edit `README.md` to add details
  - Run `pnpm db:push` if database was created
  - Start building features!

## Success Message

```
✅ App '[name]' created successfully!

📁 Files created:
- app/apps/[slug]/page.tsx
- app/apps/[slug]/README.md
[- app/apps/[slug]/layout.tsx]
[- styles/themes/[slug].css]
[- lib/db/schemas/[slug].ts]
[- config/apps.json (updated)]

🚀 Next steps:
1. Visit /apps/[slug] to see your app
2. Edit README.md to add app details
[3. Run `pnpm db:push` to create database tables]
4. Start building!

💡 Use `.claude/skills/add-app.md` for implementation guidance.
```

## References

- See `.claude/rules/app-creation.md` for detailed patterns
- See `.claude/skills/add-app.md` for quick reference
- See existing apps in `app/apps/` for examples
