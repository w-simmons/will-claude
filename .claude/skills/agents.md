# Skill: Agent SDKs
<!-- Triggers: agent SDK, Claude Agent SDK, anthropic agents, lib/agents -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- **Claude Agent SDK only** - OpenAI Agent SDK removed
- Use `runClaudeAgent()` from `@/lib/agents/anthropic`
- **Built-in tools**: Read, Write, Edit, Bash, Glob, WebSearch
- Custom tools in `app/apps/[slug]/_tools/`
- Register tools in `lib/agents/tool-registry.ts`

## Common Patterns

### Running an Agent

```typescript
import { runClaudeAgent } from '@/lib/agents/anthropic'

// Run with specific tools
const result = await runClaudeAgent(
  "Find all TODO comments in the codebase",
  ["Read", "Glob", "Bash"]
)

// Run with all available tools
const result = await runClaudeAgent(
  "Analyze the database schemas",
  ["Read", "Glob"]
)
```

### Built-in Tools

- **Read** - Read file contents
- **Write** - Write to files
- **Edit** - Make line-based edits
- **Bash** - Execute shell commands
- **Glob** - Find files by pattern
- **WebSearch** - Search the web

## Templates

### Creating a Custom Tool

```typescript
// app/apps/[slug]/_tools/my.tools.ts
export const myTool = {
  name: "get_data",
  description: "Fetches data from the database",
  parameters: {
    type: "object",
    properties: {
      id: {
        type: "string",
        description: "The ID to look up"
      }
    },
    required: ["id"]
  },
  execute: async ({ id }: { id: string }) => {
    const data = await myService.getById(id)
    return data ?? { error: "Not found" }
  }
}
```

### Registering Custom Tools

```typescript
// lib/agents/tool-registry.ts
import { myTool } from '@/app/apps/my-app/_tools/my.tools'

export const customTools = {
  'my-tool': myTool,
  // ... other tools
}
```

### Using Custom Tools

```typescript
import { runClaudeAgent } from '@/lib/agents/anthropic'

// Include custom tool by name
const result = await runClaudeAgent(
  "Get the data for user 123",
  ["Read", "my-tool"]
)
```

## Tool Development Best Practices

1. **Clear descriptions** - Agent needs to understand when to use the tool
2. **Typed parameters** - Use JSON Schema for validation
3. **Error handling** - Return `{ error: string }` on failure
4. **Async by default** - Tools can be async
5. **Keep focused** - One tool = one purpose

## References

- See `lib/agents/anthropic/index.ts` for implementation
- See `lib/agents/tool-registry.ts` for available tools
- See `lib/agents/builder-tools.ts` for builder tool examples
- Claude Agent SDK docs: https://github.com/anthropics/anthropic-sdk-typescript/tree/main/packages/agent-sdk
