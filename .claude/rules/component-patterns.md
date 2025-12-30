---
paths:
  - "**/_components/**"
  - "components/**"
priority: normal
---

# Component Patterns

**Auto-applies to**: App-specific and shared components

## Purpose

This rule covers component development patterns, including when to use app-specific vs shared components, component structure, and Shadcn integration.

## Key Principles

1. **Shadcn for ALL UI primitives** - Never invent components
2. **App-specific** components in `_components/`
3. **Shared** components in `components/shared/` (when 2+ apps use)
4. **Server Components by default** - Client Components only when needed

## Component Location Rules

### App-Specific Components

**Location**: `app/apps/[slug]/_components/`

**Use when:**
- Component is only used in one app
- Component has app-specific logic
- Component is tied to app's data structure

**Example:**
```tsx
// app/apps/chat/_components/message-list.tsx
import { Card } from '@/components/ui/card'
import type { Message } from '../_services/chat.service'

export function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="space-y-4">
      {messages.map(msg => (
        <Card key={msg.id}>
          <p>{msg.content}</p>
        </Card>
      ))}
    </div>
  )
}
```

### Shared Components

**Location**: `components/shared/`

**Use when:**
- Component is used by 2+ apps
- Component is generic/reusable
- Component has no app dependencies

**Example:**
```tsx
// components/shared/image-uploader.tsx
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'

export function ImageUploader({ onUpload }: { onUpload: (file: File) => void }) {
  // Generic image upload logic
}
```

### When to Move to Shared

| Keep in `_components/` | Move to `shared/` |
|------------------------|-------------------|
| Used in 1 app | Used in 2+ apps |
| App-specific logic | Generic/reusable |
| Tied to app's data | No app dependencies |

## Component Structure

### Server Component Pattern

```tsx
// ✅ Good: Server Component (default)
import { getItems } from './actions'
import { Card } from '@/components/ui/card'

export async function ItemList() {
  const result = await getItems()
  const items = result.data ?? []

  return (
    <div className="space-y-4">
      {items.map(item => (
        <Card key={item.id}>
          <p>{item.name}</p>
        </Card>
      ))}
    </div>
  )
}
```

### Client Component Pattern

```tsx
// ✅ Good: Client Component when needed
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'

export function InteractiveList({ items }: { items: Item[] }) {
  const [selected, setSelected] = useState<string | null>(null)

  return (
    <div className="space-y-4">
      {items.map(item => (
        <Card
          key={item.id}
          onClick={() => setSelected(item.id)}
        >
          <p>{item.name}</p>
        </Card>
      ))}
    </div>
  )
}
```

### Component with Props

```tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

interface MyComponentProps {
  title: string
  items: Array<{ id: string; name: string }>
  onAction?: (id: string) => void
}

export function MyComponent({ title, items, onAction }: MyComponentProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{title}</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="space-y-2">
          {items.map(item => (
            <div key={item.id} className="flex justify-between">
              <span>{item.name}</span>
              {onAction && (
                <Button onClick={() => onAction(item.id)}>
                  Action
                </Button>
              )}
            </div>
          ))}
        </div>
      </CardContent>
    </Card>
  )
}
```

## Shadcn Component Usage

### Always Use Shadcn

```tsx
// ✅ Good: Use Shadcn components
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card } from '@/components/ui/card'

export function MyForm() {
  return (
    <Card>
      <Input placeholder="Enter name" />
      <Button>Submit</Button>
    </Card>
  )
}
```

### Never Invent Components

```tsx
// ❌ Bad: Custom button
<button className="px-4 py-2 bg-blue-500 text-white rounded">
  Click me
</button>

// ✅ Good: Use Shadcn Button
import { Button } from '@/components/ui/button'
<Button>Click me</Button>
```

### Adding New Shadcn Components

```bash
# Ask Claude Code via MCP shadcn server (recommended)
# "Add the Dialog component from shadcn"

# Or manual
npx shadcn@latest add button

# Multiple components
npx shadcn@latest add dialog form table

# From URL
npx shadcn@latest add https://ui.shadcn.com/r/components/accordion
```

## Common Patterns

### List/Grid Pattern

```tsx
export function ItemGrid({ items }: { items: Item[] }) {
  return (
    <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {items.map(item => (
        <Card key={item.id}>
          <CardContent>
            <p>{item.name}</p>
          </CardContent>
        </Card>
      ))}
    </div>
  )
}
```

### Empty State Pattern

```tsx
import { Card } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

export function EmptyState({ onAdd }: { onAdd: () => void }) {
  return (
    <div className="text-center py-16 border border-dashed rounded-lg">
      <p className="text-lg font-medium mb-2">No items yet</p>
      <p className="text-muted-foreground mb-6">
        Add your first item to get started!
      </p>
      <Button onClick={onAdd}>
        <Plus className="h-4 w-4 mr-2" />
        Add Item
      </Button>
    </div>
  )
}
```

### Dialog Pattern

```tsx
'use client'

import { useState } from 'react'
import { Dialog, DialogContent, DialogTrigger } from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'

export function ItemDialog() {
  const [open, setOpen] = useState(false)

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Open Dialog</Button>
      </DialogTrigger>
      <DialogContent>
        <p>Dialog content</p>
      </DialogContent>
    </Dialog>
  )
}
```

## Common Mistakes

### ❌ Don't Put App-Specific Components in Shared

```tsx
// ❌ Bad: App-specific component in shared
// components/shared/chat-message.tsx
export function ChatMessage({ message }: { message: ChatMessage }) {
  // This is chat-app specific!
}

// ✅ Good: App-specific component in app
// app/apps/chat/_components/chat-message.tsx
export function ChatMessage({ message }: { message: ChatMessage }) {
  // App-specific code stays in app
}
```

### ❌ Don't Create Custom UI Primitives

```tsx
// ❌ Bad: Custom input
<input className="px-3 py-2 border rounded" />

// ✅ Good: Use Shadcn Input
import { Input } from '@/components/ui/input'
<Input />
```

### ❌ Don't Use Client Components Unnecessarily

```tsx
// ❌ Bad: Client component for static content
'use client'
export function StaticCard({ title }: { title: string }) {
  return <Card><CardTitle>{title}</CardTitle></Card>
}

// ✅ Good: Server component
export function StaticCard({ title }: { title: string }) {
  return <Card><CardTitle>{title}</CardTitle></Card>
}
```

## References

- Shadcn Docs: https://ui.shadcn.com/docs/components
- See `.claude/rules/frontend-patterns.md` for form patterns
- See `CLAUDE.md` for architecture

---

**Last Verified**: 2025-12-29
