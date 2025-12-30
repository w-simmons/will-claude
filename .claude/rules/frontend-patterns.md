---
paths:
  - "**/_components/**"
  - "**/page.tsx"
  - "**/layout.tsx"
  - "components/**"
priority: normal
---

# Frontend Patterns

**Auto-applies to**: Components, pages, layouts

## Purpose

This rule covers all frontend development patterns in the App Playground monorepo, including component usage, form handling, theming, and responsive design.

## Key Principles

1. **Server Components by default** - Use Client Components only when needed
2. **Shadcn for ALL UI primitives** - Never invent components
3. **Forms**: react-hook-form + zod + server action
4. **Toasts**: use sonner (toast.success, toast.error)
5. **Themes**: Per-app theming with OKLCH colors

## Shadcn Component Usage

### Core Rule

**Shadcn for ALL UI primitives** - 43+ components installed in `components/ui/`

- **Never invent components** — always use Shadcn or ask about alternatives
- To add Shadcn components: Ask Claude Code via MCP shadcn server
- Or manual: `npx shadcn@latest add [component-name]`
- For custom components from URLs: `npx shadcn@latest add https://ui.shadcn.com/r/components/...`
- Check component docs: https://ui.shadcn.com/docs/components

### Available Components

| Category | Components |
|----------|------------|
| **Layout** | accordion, aspect-ratio, separator, resizable |
| **Navigation** | breadcrumb, menubar, navigation-menu, pagination, tabs |
| **Forms** | button, checkbox, form, input, label, radio-group, select, slider, switch, textarea, toggle, toggle-group |
| **Feedback** | alert, alert-dialog, dialog, drawer, hover-card, popover, progress, sheet, skeleton, tooltip |
| **Data** | avatar, badge, calendar, card, carousel, chart, command, context-menu, dropdown-menu, table |

### Using Shadcn Components

```tsx
// ✅ Good: Import from @/components/ui
import { Button } from '@/components/ui/button'
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Dialog, DialogTrigger, DialogContent } from '@/components/ui/dialog'

export function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Title</CardTitle>
      </CardHeader>
      <CardContent>
        <Dialog>
          <DialogTrigger asChild>
            <Button>Open</Button>
          </DialogTrigger>
          <DialogContent>Content</DialogContent>
        </Dialog>
      </CardContent>
    </Card>
  )
}
```

### Common Patterns

**Page Layout:**
```tsx
<div className="p-6 md:p-8 lg:p-12">
  <div className="mx-auto max-w-6xl space-y-8">
    {/* Content */}
  </div>
</div>
```

**Cards Grid:**
```tsx
<div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
  <Card>...</Card>
</div>
```

**Stats Row:**
```tsx
<div className="grid gap-4 sm:grid-cols-3">
  <Card>
    <CardHeader className="pb-2">
      <CardDescription>Label</CardDescription>
    </CardHeader>
    <CardContent>
      <span className="text-2xl font-bold">Value</span>
    </CardContent>
  </Card>
</div>
```

## Server vs Client Components

### Default: Server Components

```tsx
// ✅ Good: Server Component (default)
import { getItems } from './actions'
import { Dashboard } from './_components/dashboard'

export default async function MyPage() {
  const result = await getItems()
  const items = result.data ?? []

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">My Page</h1>
      <Dashboard items={items} />
    </div>
  )
}
```

### When to Use Client Components

Use `'use client'` only when you need:
- React hooks (useState, useEffect, etc.)
- Event handlers (onClick, onChange, etc.)
- Browser APIs (localStorage, window, etc.)
- Third-party client libraries

```tsx
// ✅ Good: Client Component when needed
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'

export function InteractiveComponent() {
  const [count, setCount] = useState(0)

  return (
    <Button onClick={() => setCount(count + 1)}>
      Count: {count}
    </Button>
  )
}
```

### Decision Tree

1. **Does it need interactivity?** → Client Component
2. **Does it fetch data?** → Server Component
3. **Does it use hooks?** → Client Component
4. **Is it just presentation?** → Server Component

## Forms

### Standard Form Pattern

Forms use: **react-hook-form + zod + server action**

```tsx
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { toast } from 'sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import {
  Form,
  FormField,
  FormItem,
  FormLabel,
  FormControl,
  FormMessage
} from '@/components/ui/form'
import { createItem } from '../actions'

const schema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
})

type FormData = z.infer<typeof schema>

export function MyForm() {
  const form = useForm<FormData>({
    resolver: zodResolver(schema)
  })

  async function onSubmit(data: FormData) {
    const result = await createItem(data)
    if (result.error) {
      toast.error(result.error)
    } else {
      toast.success('Created!')
      form.reset()
    }
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={form.formState.isSubmitting}>
          {form.formState.isSubmitting ? 'Saving...' : 'Submit'}
        </Button>
      </form>
    </Form>
  )
}
```

### Alternative: FormData Pattern (for complex forms)

For forms with many fields or file uploads, you can use FormData directly:

```tsx
'use client'

import { useTransition } from 'react'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { toast } from 'sonner'
import { createItem } from '../actions'

export function SimpleForm() {
  const [isPending, startTransition] = useTransition()

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const formData = new FormData(e.currentTarget)

    startTransition(async () => {
      const result = await createItem(formData)
      if (result.error) {
        toast.error(result.error)
      } else {
        toast.success('Item created')
      }
    })
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="name">Name *</Label>
        <Input id="name" name="name" required />
      </div>

      <Button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Submit'}
      </Button>
    </form>
  )
}
```

## Toast Notifications

### Using Sonner

```tsx
import { toast } from 'sonner'

// Success
toast.success('Item created successfully')

// Error
toast.error('Failed to create item')

// Info
toast.info('Processing...')

// Loading
const toastId = toast.loading('Saving...')
// Later...
toast.success('Saved!', { id: toastId })
```

### Common Pattern in Forms

```tsx
async function onSubmit(data: FormData) {
  const result = await createItem(data)
  if (result.error) {
    toast.error(result.error)
  } else {
    toast.success('Created!')
    form.reset()
  }
}
```

## Theme Usage

### Creating a Theme

1. **Create theme file**: `styles/themes/[app-slug].css`

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

2. **Import in layout**: `app/apps/[slug]/layout.tsx`

```tsx
import '@/styles/globals.css'
import '@/styles/themes/my-app.css'

export default function MyAppLayout({ children }: { children: React.ReactNode }) {
  return (
    <div data-theme="my-app" className="min-h-screen bg-background">
      {children}
    </div>
  )
}
```

### OKLCH Color Format

```css
oklch(lightness chroma hue)
```

- **Lightness**: 0-1 (0 = black, 1 = white)
- **Chroma**: 0-0.4 (0 = gray, higher = saturated)
- **Hue**: 0-360 (color wheel)

```css
/* Green */  --primary: oklch(0.55 0.2 140);
/* Blue */   --primary: oklch(0.55 0.2 240);
/* Red */    --primary: oklch(0.55 0.2 20);
/* Purple */ --primary: oklch(0.55 0.2 280);
```

### Available CSS Variables

| Variable | Purpose |
|----------|---------|
| `--primary` | Primary brand color |
| `--primary-foreground` | Text on primary |
| `--secondary` | Secondary color |
| `--accent` | Accent color |
| `--muted` | Muted background |
| `--destructive` | Error/danger |
| `--background` | Page background |
| `--foreground` | Main text |
| `--card` | Card background |
| `--border` | Border color |
| `--ring` | Focus ring |
| `--radius` | Border radius |

## Responsive Design Patterns

### Container Patterns

```tsx
// Standard container
<div className="container mx-auto px-4 py-6">
  {children}
</div>

// With max-width
<div className="mx-auto max-w-6xl space-y-8">
  {children}
</div>
```

### Grid Patterns

```tsx
// Responsive grid
<div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
  {items.map(item => <Card key={item.id}>{item.content}</Card>)}
</div>

// Stats grid
<div className="grid gap-4 sm:grid-cols-3">
  <Card>Stat 1</Card>
  <Card>Stat 2</Card>
  <Card>Stat 3</Card>
</div>
```

### Spacing Patterns

```tsx
// Vertical spacing
<div className="space-y-6">
  <Section1 />
  <Section2 />
</div>

// Horizontal spacing
<div className="flex gap-4">
  <Button>Action 1</Button>
  <Button>Action 2</Button>
</div>
```

## Common Mistakes

### ❌ Don't Create Custom UI Primitives

```tsx
// ❌ Bad: Custom button
<button className="px-4 py-2 bg-blue-500 text-white rounded">
  Click me
</button>

// ✅ Good: Use Shadcn Button
import { Button } from '@/components/ui/button'
<Button>Click me</Button>
```

### ❌ Don't Use Client Components Unnecessarily

```tsx
// ❌ Bad: Client component for static content
'use client'
export function StaticContent() {
  return <p>Hello</p>
}

// ✅ Good: Server component
export function StaticContent() {
  return <p>Hello</p>
}
```

### ❌ Don't Hardcode Colors

```tsx
// ❌ Bad: Hardcoded color
<div className="bg-blue-500 text-white">

// ✅ Good: Use CSS variables
<div className="bg-primary text-primary-foreground">
```

## References

- Shadcn Docs: https://ui.shadcn.com/docs/components
- Shadcn Registry: https://ui.shadcn.com/r
- See `.claude/skills/forms.md` for detailed form patterns
- See `.claude/skills/shadcn.md` for Shadcn usage

---

**Last Verified**: 2025-12-29
