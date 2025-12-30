# Skill: Using the LLM Library
<!-- Triggers: LLM, lib/llm, call LLM, AI completion, generate text -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- Use `llm` from `@/lib/llm` - **never create new LLM clients**
- Default model is `claude-sonnet`
- Available functions: `ask`, `chat`, `json`, `stream`, `image`, `vision`
- Call from Server Actions or Services only (never client components)
- Mock responses when API keys are missing (development mode)

## Available Functions

| Function | Use Case |
|----------|----------|
| `llm.ask(prompt, opts?)` | Simple prompt → text |
| `llm.chat(messages, opts?)` | Multi-turn conversation |
| `llm.json<T>(prompt, schema, opts?)` | Get typed JSON |
| `llm.stream(prompt, opts?)` | Streaming response |
| `llm.image(prompt)` | Generate images (Gemini only) |
| `llm.vision(prompt, images, opts?)` | Analyze images (Gemini only) |

## Common Patterns

### Simple Completion

```typescript
import { llm } from '@/lib/llm'

// Simple ask (uses default model: claude-sonnet)
const response = await llm.ask('Explain quantum computing')
console.log(response.text)

// With specific model
const response = await llm.ask('Explain this', {
  model: 'claude-haiku'
})

// With options
const response = await llm.ask('Write a poem', {
  model: 'claude-sonnet',
  maxTokens: 200,
  temperature: 0.7,
  systemPrompt: 'You are a creative poet.',
})
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

const result = await llm.json(
  'Extract structured data from this text: ...',
  dataSchema,
  { model: 'claude-sonnet' }
)

// result is typed as { title: string; summary: string; tags: string[] }
```

### Multi-turn Conversation

```typescript
const messages = [
  { role: 'user' as const, content: 'Hello' },
  { role: 'assistant' as const, content: 'Hi there!' },
  { role: 'user' as const, content: 'What is React?' },
]

const response = await llm.chat(messages, {
  model: 'claude-sonnet',
  systemPrompt: 'You are a helpful assistant.',
})
```

### Vision Analysis (Gemini Only)

```typescript
const result = await llm.vision(
  'What do you see in this image?',
  [{ base64: '...', mimeType: 'image/jpeg' }],
  { model: 'gemini-flash' }
)

console.log(result.text)
```

### Image Generation (Gemini Only)

```typescript
const result = await llm.image('A red sports car in the desert')

if (result.error) {
  console.error(result.error)
} else {
  console.log(result.base64)
  console.log(result.mimeType)
}
```

## Templates

### In Service

```typescript
// app/apps/[slug]/_services/analysis.service.ts
import { llm } from '@/lib/llm'
// No Next.js imports - service is portable!

export const analysisService = {
  async analyzeData(data: string): Promise<string> {
    const prompt = `Analyze this data and provide insights: ${data}`

    try {
      const response = await llm.ask(prompt, {
        model: 'claude-sonnet',
        maxTokens: 500,
      })
      return response.text
    } catch (error) {
      console.error('LLM analysis error:', error)
      throw new Error('Failed to analyze data')
    }
  },
}
```

### In Server Action

```typescript
// app/apps/[slug]/actions.ts
'use server'

import { llm } from '@/lib/llm'
import { z } from 'zod'

const resultSchema = z.object({
  sentiment: z.enum(['positive', 'negative', 'neutral']),
  confidence: z.number(),
})

export async function analyzeSentiment(text: string) {
  try {
    const result = await llm.json(
      `Analyze the sentiment of this text: "${text}"`,
      resultSchema,
      { model: 'claude-haiku' }
    )

    return { data: result }
  } catch (error) {
    console.error('Sentiment analysis error:', error)
    return { error: 'Failed to analyze sentiment' }
  }
}
```

## Available Models

**Anthropic:**
- `claude-opus` - Most capable (claude-opus-4-5-20251101)
- `claude-sonnet` - Balanced, default (claude-sonnet-4-5-20250929)
- `claude-haiku` - Fast and cheap (claude-haiku-4-5-20251001)

**Google:**
- `gemini-pro` - Most capable (gemini-2.5-pro-preview-06-05)
- `gemini-flash` - Fast and cheap (gemini-2.5-flash)
- `gemini-image` - Image generation (gemini-3-pro-image-preview)

## Common Mistakes

❌ **Don't create new LLM clients**
```typescript
// Bad
import { Anthropic } from '@anthropic-ai/sdk'
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })
```

✅ **Use lib/llm**
```typescript
// Good
import { llm } from '@/lib/llm'
const response = await llm.ask('Hello')
```

❌ **Don't use Claude for vision**
```typescript
// Bad - won't work
const result = await llm.vision(prompt, images, { model: 'claude-sonnet' })
```

✅ **Use Gemini for vision**
```typescript
// Good
const result = await llm.vision(prompt, images, { model: 'gemini-flash' })
```

## References

- See `.claude/rules/llm-patterns.md` for comprehensive patterns
- See `lib/llm/index.ts` for all available functions
- See `lib/llm/models.ts` for model configurations
