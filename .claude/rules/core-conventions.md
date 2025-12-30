---
priority: high
---

# Core Conventions

**Always Active** - This rule applies to all files in the project

## Purpose

This rule establishes the fundamental conventions for the App Playground monorepo. These patterns apply to all code and should be followed consistently.

## Key Principles

1. **App Isolation**: Each app is self-contained in `app/apps/[slug]/`
2. **Shared Infrastructure**: Common code lives in `lib/`, `components/shared/`, `styles/`
3. **Portable Services**: Business logic has no Next.js dependencies
4. **Type Safety**: TypeScript strict mode, Zod validation, Drizzle types

## Project Structure

### Directory Organization

```
/
├── app/
│   ├── apps/[slug]/           # App-specific routes
│   │   ├── page.tsx           # Main page (Server Component)
│   │   ├── layout.tsx         # App layout (imports theme)
│   │   ├── actions.ts         # Server actions (thin validation layer)
│   │   ├── README.md          # App documentation
│   │   ├── _components/       # App-specific components
│   │   ├── _services/        # Business logic (portable, no Next.js deps)
│   │   └── _tools/           # Agent tools (optional)
│   └── layout.tsx            # Root layout
│
├── lib/
│   ├── llm/                  # Unified LLM client
│   ├── db/                   # Drizzle ORM
│   │   ├── schemas/          # Per-app schemas
│   │   └── schema.ts         # Schema exports
│   ├── agents/               # Agent SDKs (Anthropic only)
│   ├── validation/           # Zod schemas
│   ├── storage/              # LocalStorage abstraction
│   └── env.ts                # Type-safe env
│
├── components/
│   ├── ui/                   # Shadcn components (always available)
│   └── shared/               # Shared across 2+ apps
│
├── styles/
│   ├── globals.css           # Base styles
│   └── themes/               # Per-app themes (optional)
│
├── .claude/
│   ├── rules/                # Contextual rules
│   ├── skills/               # Auto-discovering skills
│   ├── commands/             # Slash commands
│   └── settings.json         # Claude Code config
│
└── config/
    └── apps.json             # App registry (optional)
```

### App Structure Rules

- **New apps** go in `app/apps/[slug]/`
- **App-specific components** in `_components/`
- **App-specific services** in `_services/`
- **App-specific agent tools** in `_tools/`
- **App README** required - provides context for Claude Code
- **Move to `components/shared/`** when 2+ apps use something

### File Naming Conventions

- **Components**: PascalCase (e.g., `CollectionDashboard.tsx`)
- **Services**: kebab-case with `.service.ts` suffix (e.g., `measurements.service.ts`)
- **Actions**: `actions.ts` (always this name)
- **Pages**: `page.tsx` (Next.js convention)
- **Layouts**: `layout.tsx` (Next.js convention)
- **Schemas**: kebab-case (e.g., `chat.ts`, `foot-measure.ts`)

## Import Conventions

### Import Order

1. React/Next.js imports
2. Third-party libraries
3. UI components (`@/components/ui/*`)
4. Shared components (`@/components/shared/*`)
5. App-specific components (`./_components/*`)
6. Services (`./_services/*`)
7. Lib utilities (`@/lib/*`)
8. Types
9. Relative imports

### Import Examples

```typescript
// ✅ Good: Server Component
import { getItems } from './actions'
import { Dashboard } from './_components/dashboard'
import { Card } from '@/components/ui/card'

// ✅ Good: Client Component
'use client'
import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { toast } from 'sonner'

// ✅ Good: Service
import { db, eq } from '@/lib/db'
import { mySchema } from '@/lib/db/schemas/my-app'
```

### Path Aliases

- `@/` - Root of project (configured in `tsconfig.json`)
- Use `@/` for all imports from `lib/`, `components/`, `styles/`
- Use relative paths (`./`, `../`) for app-specific imports

## Code Organization Principles

### Separation of Concerns

1. **UI Layer** (`page.tsx`, `_components/`)
   - Presentation logic only
   - Server Components by default
   - Client Components when needed (`'use client'`)

2. **Validation Layer** (`actions.ts`)
   - Thin validation with Zod
   - Call services
   - Return `{ data }` or `{ error }`
   - Use `revalidatePath` after mutations

3. **Business Logic Layer** (`_services/`)
   - Portable (no Next.js dependencies)
   - Database operations
   - External API calls
   - Complex business logic
   - **Must use `.service.ts` suffix**

### Example: Proper Separation

```typescript
// ✅ actions.ts - Thin validation layer
'use server'
import { z } from 'zod'
import { revalidatePath } from 'next/cache'
import { dataService } from './_services/data.service'

const schema = z.object({
  name: z.string().min(1),
  description: z.string().min(1),
})

export async function createItem(formData: FormData) {
  const parsed = schema.safeParse({
    name: formData.get('name'),
    description: formData.get('description'),
  })

  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  try {
    const item = await dataService.create(parsed.data)
    revalidatePath('/apps/my-app')
    return { data: item }
  } catch (error) {
    return { error: 'Failed to create item' }
  }
}
```

```typescript
// ✅ _services/data.service.ts - Portable business logic
import { db, eq } from '@/lib/db'
import { myAppItems } from '@/lib/db/schemas/my-app'
// No Next.js imports!

export const dataService = {
  async create(data: { name: string; description: string }) {
    if (!db) throw new Error('Database not configured')

    const [item] = await db
      .insert(myAppItems)
      .values(data)
      .returning()

    return item
  },

  async getAll() {
    if (!db) return []
    return await db.select().from(myAppItems)
  },
}
```

## What Goes Where

### ✅ In `lib/`
- Shared utilities
- Database setup
- LLM clients
- Agent SDKs (Anthropic only)
- Type-safe env
- Validation schemas (reusable)
- Storage utilities

### ❌ NOT in `lib/`
- App-specific code
- App-specific components
- App-specific services

### ✅ In `components/shared/`
- Components used by 2+ apps
- Generic reusable components

### ✅ In `app/apps/[slug]/_components/`
- Components only used in one app
- App-specific UI logic

### ✅ In `app/apps/[slug]/_services/`
- Business logic specific to this app
- **Must be portable** (no Next.js imports)
- **Must use `.service.ts` suffix**

## Common Mistakes

### ❌ Don't Put App-Specific Code in `lib/`

```typescript
// ❌ Bad
// lib/chat-utils.ts
export function formatChatMessage(message: Message) {
  // This is chat-app specific!
}

// ✅ Good
// app/apps/chat/_services/chat.service.ts
export function formatChatMessage(message: Message) {
  // App-specific code stays in app
}
```

### ❌ Don't Import Next.js in Services

```typescript
// ❌ Bad - Service with Next.js imports
import { revalidatePath } from 'next/cache'
import { cookies } from 'next/headers'

export const myService = {
  async doSomething() {
    // revalidatePath('/apps/my-app') // NEVER in services!
  }
}

// ✅ Good - Service is portable
export const myService = {
  async doSomething() {
    // Pure business logic only
    return { success: true }
  }
}
```

### ❌ Don't Use Absolute Paths for App-Specific Imports

```typescript
// ❌ Bad
import { MyComponent } from '@/app/apps/my-app/_components/MyComponent'

// ✅ Good
import { MyComponent } from './_components/MyComponent'
```

### ❌ Don't Forget `.service.ts` Suffix

```typescript
// ❌ Bad - No suffix
// _services/data.ts

// ❌ Bad - Wrong suffix
// _services/utils.ts

// ✅ Good - Clear purpose
// _services/data.service.ts
// _services/llm.service.ts
```

### ❌ Don't Skip App README.md

```bash
# ❌ Bad - No context for Claude Code
app/apps/my-app/
├── page.tsx
├── layout.tsx
└── actions.ts

# ✅ Good - README provides context
app/apps/my-app/
├── README.md         # Required!
├── page.tsx
├── layout.tsx
└── actions.ts
```

## Portability Checklist for Services

Before committing a service, verify:

- [ ] No `import` from `next/cache`, `next/headers`, `next/navigation`
- [ ] No `revalidatePath`, `cookies()`, `headers()`, `redirect()`
- [ ] File name uses `.service.ts` suffix
- [ ] Only imports from `@/lib/*` or npm packages
- [ ] Pure functions or object with methods
- [ ] All business logic is self-contained

## App README Requirements

Every app must have `README.md` with:

1. **Purpose** - What problem does this app solve?
2. **Features** - What can users do?
3. **Architecture** - Key technical decisions
4. **Patterns** - Important implementation details

This helps Claude Code understand context when working on the app.

## References

- See `.claude/skills/add-app.md` for creating new apps
- See `.claude/commands/new-app.md` for automated scaffolding
- See `CLAUDE.md` for high-level patterns

---

**Last Verified**: 2025-12-29
