# Skill: Adding a New App
<!-- Triggers: new app, create app, scaffold app, add app -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- Use `/new-app [slug] [name]` command for automated scaffolding
- **Always create README.md** - Required for AI context
- Register in `config/apps.json` (optional, for global nav)
- Create theme in `styles/themes/[slug].css` (optional)
- Create schema in `lib/db/schemas/[slug].ts` (if needed)

## Common Patterns

### Minimal App Structure

```
app/apps/[slug]/
  page.tsx        # Main page (required)
  README.md       # Documentation (REQUIRED)
  layout.tsx      # Theme wrapper (optional)
  actions.ts      # Server actions (if needed)
  _services/      # Business logic (if needed)
  _components/    # App-specific UI (if needed)
```

### Quick Checklist

1. ✅ Create `app/apps/[slug]/page.tsx`
2. ✅ Create `app/apps/[slug]/README.md` (required!)
3. ✅ Create `app/apps/[slug]/layout.tsx` (optional, for theme)
4. ✅ Create `app/apps/[slug]/actions.ts` (if needed)
5. ✅ Create theme file `styles/themes/[slug].css` (optional)
6. ✅ Create database schema `lib/db/schemas/[slug].ts` (if needed)
7. ✅ Update `config/apps.json` (optional, for global nav)

## Templates

### Minimal Page

```tsx
// app/apps/[slug]/page.tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'

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
            <p>Content here</p>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
```

### Layout with Theme

```tsx
// app/apps/[slug]/layout.tsx
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

### README Template

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

### Config Entry (Optional)

```json
// config/apps.json
{
  "slug": "my-app",
  "name": "My App",
  "description": "What my app does",
  "icon": "IconName",
  "category": "apps",
  "theme": "my-app"
}
```

## Automation

**Use `/new-app` command** for automated scaffolding:

```bash
/new-app my-app "My App Name" "apps"
```

This will create all required files and set up the directory structure.

## References

- See `.claude/rules/app-creation.md` for detailed step-by-step guide
- See `.claude/commands/new-app.md` for automation
- See existing apps in `app/apps/` for examples
