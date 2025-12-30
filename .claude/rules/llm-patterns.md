---
paths:
  - "**/_services/**"
  - "app/api/**"
  - "lib/llm/**"
priority: normal
---

# LLM Integration Patterns

**Auto-applies to**: Services, API routes, LLM library

## Purpose

This rule covers LLM integration patterns using the unified `lib/llm` library. Never create new LLM clients or wrappers.

## Key Principles

1. **Use lib/llm functions** - Never reinvent the LLM wheel
2. **Default model** is `claude-sonnet`
3. **Never create new LLM clients** - use existing infrastructure
4. **Mock responses** when API keys are missing (for development)

## Using lib/llm

### Import Pattern

```typescript
import { llm } from '@/lib/llm'
```

### Available Functions

```typescript
// Simple prompt → text
const response = await llm.ask('What is React?')

// Multi-turn conversation
const response = await llm.chat([
  { role: 'user', content: 'Hello' },
  { role: 'assistant', content: 'Hi there!' },
  { role: 'user', content: 'What is React?' },
])

// Typed JSON response
const data = await llm.json<MyType>(
  'Extract user info from this text: ...',
  myZodSchema
)

// Streaming response
for await (const chunk of llm.stream('Tell me a story')) {
  console.log(chunk)
}

// Image generation (Gemini only)
const image = await llm.image('A red sports car')

// Vision: Analyze images (Gemini only)
const result = await llm.vision(
  'What do you see in this image?',
  [{ base64: '...', mimeType: 'image/jpeg' }],
  { model: 'gemini-flash' }
)
```

## Model Selection

### Available Models

```typescript
// Anthropic
"claude-opus"      // claude-opus-4-5-20251101
"claude-sonnet"    // claude-sonnet-4-5-20250929 (default)
"claude-haiku"     // claude-haiku-4-5-20251001

// Google
"gemini-pro"       // gemini-2.5-pro-preview-06-05
"gemini-flash"     // gemini-2.5-flash
"gemini-image"     // gemini-3-pro-image-preview
```

### Specifying Model

```typescript
// Use default (claude-sonnet)
const response = await llm.ask('Hello')

// Specify model
const response = await llm.ask('Hello', {
  model: 'gemini-flash'
})

// With options
const response = await llm.ask('Hello', {
  model: 'claude-sonnet',
  maxTokens: 1000,
  temperature: 0.7,
  systemPrompt: 'You are a helpful assistant.',
})
```

## Common Patterns

### Simple Completion

```typescript
import { llm } from '@/lib/llm'

export async function generateDescription(item: {
  name: string
  category: string
}) {
  const prompt = `Write a brief description of ${item.name} in the ${item.category} category.`

  const response = await llm.ask(prompt, {
    maxTokens: 200,
  })

  return response.text
}
```

### Typed JSON Response

```typescript
import { z } from 'zod'
import { llm } from '@/lib/llm'

const dataSchema = z.object({
  title: z.string(),
  summary: z.string(),
  tags: z.array(z.string()),
})

export async function analyzeContent(
  content: string
): Promise<z.infer<typeof dataSchema>> {
  const prompt = `Analyze this content and extract structured data: ${content}`

  const result = await llm.json(
    prompt,
    dataSchema,
    {
      model: 'claude-sonnet',
    }
  )

  return result
}
```

### Vision Analysis

```typescript
import { llm } from '@/lib/llm'

export async function analyzeImage(
  prompt: string,
  images: Array<{ base64: string; mimeType: string }>
) {
  const result = await llm.vision(
    prompt,
    images,
    {
      model: 'gemini-flash', // Vision only works with Gemini
    }
  )

  return result.text
}
```

### Conversation Pattern

```typescript
import { llm } from '@/lib/llm'

export async function chatWithAssistant(
  messages: Array<{ role: 'user' | 'assistant'; content: string }>,
  userMessage: string
) {
  const response = await llm.chat([
    ...messages,
    { role: 'user', content: userMessage },
  ], {
    model: 'claude-sonnet',
    systemPrompt: 'You are a helpful assistant.',
  })

  return response.text
}
```

## Service Integration

### LLM in Services

```typescript
// app/apps/my-app/_services/analysis.service.ts
import { llm } from '@/lib/llm'
// No Next.js imports - service is portable!

export const analysisService = {
  async analyzeData(data: string): Promise<string> {
    const prompt = `Analyze this data: ${data}`

    try {
      const response = await llm.ask(prompt)
      return response.text
    } catch (error) {
      console.error('LLM analysis error:', error)
      throw new Error('Failed to analyze data')
    }
  },
}
```

### Error Handling

```typescript
export async function generateContent(prompt: string) {
  try {
    const response = await llm.ask(prompt)
    return { data: response.text }
  } catch (error) {
    console.error('LLM error:', error)

    // Check if it's a configuration error
    if (error.message?.includes('API key')) {
      return { error: 'LLM not configured' }
    }

    return { error: 'Failed to generate content' }
  }
}
```

## Common Mistakes

### ❌ Don't Create New LLM Clients

```typescript
// ❌ Bad: Creating new client
import { Anthropic } from '@anthropic-ai/sdk'
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })

// ✅ Good: Use lib/llm
import { llm } from '@/lib/llm'
const response = await llm.ask('Hello')
```

### ❌ Don't Hardcode API Keys

```typescript
// ❌ Bad: Hardcoded API key
const client = new Anthropic({ apiKey: 'sk-ant-...' })

// ✅ Good: Use lib/llm (handles env vars)
import { llm } from '@/lib/llm'
```

### ❌ Don't Use Wrong Model for Vision

```typescript
// ❌ Bad: Vision with non-Gemini model
const result = await llm.vision(
  'What is in this image?',
  [{ base64: '...', mimeType: 'image/jpeg' }],
  { model: 'claude-sonnet' } // Won't work!
)

// ✅ Good: Use Gemini for vision
const result = await llm.vision(
  'What is in this image?',
  [{ base64: '...', mimeType: 'image/jpeg' }],
  { model: 'gemini-flash' }
)
```

## References

- See `.claude/skills/llm-usage.md` for detailed LLM patterns
- See `lib/llm/index.ts` for all available functions
- See `lib/llm/models.ts` for available models

---

**Last Verified**: 2025-12-29
