# Skill: Shadcn UI Components
<!-- Triggers: shadcn, UI component, add component, shadcn component -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- **Always use Shadcn** - Never invent UI components
- Add via CLI: `npx shadcn@latest add [component]`
- Or ask Claude Code: "Add the Dialog component from shadcn" (via MCP)
- Components install to `components/ui/[component-name].tsx`
- 43+ components already installed
- Server Components by default (most work without 'use client')

## Adding Components

### Via CLI (Preferred)

```bash
# Single component
npx shadcn@latest add button

# Multiple components
npx shadcn@latest add button dialog form table

# From URL (custom components)
npx shadcn@latest add https://ui.shadcn.com/r/components/[component-path]
```

### Via MCP (If Configured)

Ask Claude Code:
- "Add the Dialog component from shadcn"
- "What shadcn components are available?"
- "Show me the Button component examples"

## Common Patterns

### Basic Usage

```tsx
import { Button } from '@/components/ui/button'
import {
  Card,
  CardHeader,
  CardTitle,
  CardContent
} from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

export function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Form</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <div>
          <Label htmlFor="name">Name</Label>
          <Input id="name" placeholder="Enter name" />
        </div>
        <Button>Submit</Button>
      </CardContent>
    </Card>
  )
}
```

### Dialog Pattern

```tsx
'use client'
import { useState } from 'react'
import {
  Dialog,
  DialogContent,
  DialogTrigger,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'

export function MyDialog() {
  const [open, setOpen] = useState(false)

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Open Dialog</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Dialog Title</DialogTitle>
        </DialogHeader>
        <p>Dialog content</p>
      </DialogContent>
    </Dialog>
  )
}
```

### Form with Shadcn

```tsx
import { useForm } from 'react-hook-form'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue
} from '@/components/ui/select'

export function MyForm() {
  const { register, handleSubmit } = useForm()

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Label htmlFor="name">Name</Label>
        <Input id="name" {...register('name')} />
      </div>

      <div>
        <Label htmlFor="category">Category</Label>
        <Select>
          <SelectTrigger>
            <SelectValue placeholder="Select" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="option1">Option 1</SelectItem>
            <SelectItem value="option2">Option 2</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <Button type="submit">Submit</Button>
    </form>
  )
}
```

## Available Components (43+ installed)

**Layout:**
- Card, Separator, AspectRatio, Resizable, Accordion

**Forms:**
- Button, Input, Select, Checkbox, RadioGroup, Switch, Slider, Textarea, Toggle

**Overlays:**
- Dialog, Sheet, Popover, Tooltip, AlertDialog, Drawer, HoverCard

**Data:**
- Table, Badge, Avatar, Calendar, Carousel, Chart

**Navigation:**
- Tabs, Breadcrumb, Pagination, Menubar, NavigationMenu

**Feedback:**
- Alert, Progress, Skeleton, Toast (via sonner)

## Customization

```tsx
// Use className for one-off styling
<Button className="bg-purple-500 hover:bg-purple-600">
  Custom Button
</Button>

// Edit component file directly for permanent changes
// components/ui/button.tsx

// Create wrappers in components/shared/ for reusable customizations
export function PrimaryButton(props) {
  return <Button className="bg-primary" {...props} />
}
```

## Best Practices

✅ **Always use Shadcn components** - Never reinvent
✅ **Check props at** https://ui.shadcn.com/docs/components
✅ **Server Components by default** - Most work without 'use client'
✅ **Use className for variants** - Shadcn uses CVA (class-variance-authority)

❌ **Don't create custom UI primitives**
```tsx
// Bad
<button className="px-4 py-2 bg-blue-500 rounded">Click</button>

// Good
import { Button } from '@/components/ui/button'
<Button>Click</Button>
```

## References

- See `.claude/rules/component-patterns.md` for component organization
- Shadcn docs: https://ui.shadcn.com/docs/components
- Browse components: `components/ui/`
