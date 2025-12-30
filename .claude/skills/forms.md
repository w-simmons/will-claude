# Skill: Form Patterns
<!-- Triggers: form, react-hook-form, zod validation, form validation, Server Actions -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- Use `react-hook-form` + `zod` for validation
- Use `zodResolver` for schema integration
- Server Actions handle submission
- Use `toast` from `sonner` for feedback
- Always use Shadcn form components

## Common Patterns

### Standard Form Setup

```tsx
'use client'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { toast } from 'sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

const schema = z.object({
  title: z.string().min(1, 'Title is required'),
  email: z.string().email('Invalid email'),
  description: z.string().optional(),
})

type FormData = z.infer<typeof schema>

export function MyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset
  } = useForm<FormData>({
    resolver: zodResolver(schema)
  })

  async function onSubmit(data: FormData) {
    const result = await myServerAction(data)

    if (result.error) {
      toast.error(result.error)
    } else {
      toast.success('Saved!')
      reset()
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Label htmlFor="title">Title</Label>
        <Input id="title" {...register('title')} />
        {errors.title && (
          <p className="text-sm text-red-500">{errors.title.message}</p>
        )}
      </div>

      <div>
        <Label htmlFor="email">Email</Label>
        <Input id="email" type="email" {...register('email')} />
        {errors.email && (
          <p className="text-sm text-red-500">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Label htmlFor="description">Description</Label>
        <Input id="description" {...register('description')} />
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </Button>
    </form>
  )
}
```

## Templates

### Server Action Pattern

```typescript
// app/apps/[slug]/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { myService } from './_services/my.service'

const schema = z.object({
  title: z.string().min(1),
  email: z.string().email(),
  description: z.string().optional(),
})

export async function myServerAction(input: z.infer<typeof schema>) {
  const parsed = schema.safeParse(input)

  if (!parsed.success) {
    return { error: 'Invalid input' }
  }

  try {
    const result = await myService.create(parsed.data)
    revalidatePath('/apps/[slug]')
    return { data: result }
  } catch (error) {
    console.error('myServerAction error:', error)
    return { error: 'Failed to save' }
  }
}
```

### Form with Select

```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

// In form:
<div>
  <Label htmlFor="category">Category</Label>
  <Select onValueChange={(value) => setValue('category', value)}>
    <SelectTrigger>
      <SelectValue placeholder="Select category" />
    </SelectTrigger>
    <SelectContent>
      <SelectItem value="apps">Apps</SelectItem>
      <SelectItem value="utilities">Utilities</SelectItem>
    </SelectContent>
  </Select>
  {errors.category && (
    <p className="text-sm text-red-500">{errors.category.message}</p>
  )}
</div>
```

### Form with Textarea

```tsx
import { Textarea } from '@/components/ui/textarea'

// In form:
<div>
  <Label htmlFor="description">Description</Label>
  <Textarea
    id="description"
    {...register('description')}
    rows={4}
  />
  {errors.description && (
    <p className="text-sm text-red-500">{errors.description.message}</p>
  )}
</div>
```

### Form with Checkbox

```tsx
import { Checkbox } from '@/components/ui/checkbox'

// In form:
<div className="flex items-center space-x-2">
  <Checkbox
    id="isActive"
    {...register('isActive')}
  />
  <Label htmlFor="isActive">Active</Label>
</div>
```

## Common Validation Patterns

```typescript
const schema = z.object({
  // Required string
  name: z.string().min(1, 'Name is required'),

  // Email
  email: z.string().email('Invalid email'),

  // Optional string
  description: z.string().optional(),

  // Number
  age: z.number().min(0).max(120),

  // Boolean
  isActive: z.boolean(),

  // Enum
  category: z.enum(['apps', 'utilities']),

  // Array
  tags: z.array(z.string()),

  // Custom validation
  password: z.string().min(8).regex(/[A-Z]/, 'Must contain uppercase'),
})
```

## References

- See `.claude/rules/frontend-patterns.md` for comprehensive form patterns
- See existing forms in `app/apps/*/components/` for examples
- react-hook-form docs: https://react-hook-form.com/
- Zod docs: https://zod.dev/
