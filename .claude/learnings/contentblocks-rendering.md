# ContentBlocks and Tool Result Rendering - Critical Learnings

**Date**: 2025-12-28
**Context**: Agent Builder BUILD/TEST modes, tool result rendering

## The Problem We Solved

### Issue 1: Preview Not Populating + Conversation Not Continuing
**Symptoms**:
- Agent calls `update_config` tool
- Preview panel doesn't populate OR conversation doesn't continue
- Can't have both working simultaneously

**Root Cause**: Three separate issues that compounded:

1. **Tool markers not reaching frontend** (Python → Next.js)
   - OpenAI SDK's `result.final_output` excludes tool results
   - When agent continues conversation, only text response was returned
   - `[UPDATE_CONFIG:{...}]` marker never reached frontend

2. **Text not preserved in final message** (Next.js streaming)
   - When `contentBlocks` existed, final message sent `content: ''` (empty)
   - Streamed text was displayed temporarily, then cleared by 'done' event
   - Text disappeared when final message arrived

3. **ContentBlocks without text block** (Message rendering)
   - `contentBlocks` only contained tool blocks: `[{type: 'tool_use', ...}]`
   - `message.content` had the text, but Message component ignores it when `contentBlocks` exist
   - Message component ONLY renders blocks from `contentBlocks` array

**Solution**:
1. Extract tool results in Python service (`main.py`) and append to content
2. Set `content: textContent` (not empty) in 'done' event
3. Add text block to `contentBlocks`: `contentBlocks.push({type: 'text', text: textContent})`

### Issue 2: ask_user Showing as Raw JSON in Test Mode
**Symptoms**:
- Test mode displays `[ASK_USER:{...}]` as raw JSON instead of interactive buttons
- Build mode works fine

**Root Cause**:
- Test Panel (`test-panel.tsx`) doesn't pass `renderToolResult` callback to `ChatMessages`
- Without callback, special formats aren't detected and rendered properly
- ChatMessages falls back to default ToolLine which shows raw JSON

## How ContentBlocks Work

### Message Structure
```typescript
type ChatSessionMessage = {
  id: string
  role: 'user' | 'assistant'
  content: string  // Text content (ignored if contentBlocks exist!)
  contentBlocks?: OrderedContentBlock[]  // Ordered rendering
  toolCalls?: ToolCall[]  // Legacy format
  createdAt: Date
}
```

### OrderedContentBlock Format
```typescript
type OrderedContentBlock =
  | { type: 'text'; text: string }
  | { type: 'tool_use'; id: string; name: string; input: any; result?: string }
```

### Critical Rule: ContentBlocks Override Content

**In `chat-messages.tsx` line 175:**
```typescript
function Message({ message, renderToolResult }) {
  // If contentBlocks exist, ONLY render those blocks
  if (message.contentBlocks && message.contentBlocks.length > 0) {
    return (
      <div className="space-y-2">
        {message.contentBlocks.map((block) => {
          if (block.type === 'text') return renderTextContent(block.text)
          if (block.type === 'tool_use') return renderTool(block)
        })}
      </div>
    )
  }

  // Otherwise, render content + toolCalls (legacy)
  return (
    <>
      {message.toolCalls && renderTools(message.toolCalls)}
      {message.content && renderTextContent(message.content)}
    </>
  )
}
```

**Key Insight**: When `contentBlocks` exist, `message.content` is **completely ignored**!

## Proper ContentBlocks Pattern

### ❌ WRONG: Only Tool Blocks
```typescript
// This will render ONLY the tool, text is lost!
contentBlocks = [
  { type: 'tool_use', id: '1', name: 'update_config', result: '[UPDATE_CONFIG:{...}]' }
]
message.content = "I created your agent..."  // ← IGNORED!
```

### ✅ CORRECT: Tool + Text Blocks
```typescript
// This renders both tool AND text
contentBlocks = [
  { type: 'tool_use', id: '1', name: 'update_config', result: '[UPDATE_CONFIG:{...}]' },
  { type: 'text', text: "I created your agent..." }  // ← RENDERED!
]
message.content = ""  // ← Not needed when using contentBlocks
```

## renderToolResult Callback Pattern

### Purpose
Custom rendering for special tool formats (UPDATE_CONFIG, ASK_USER, etc.)

### Implementation in AgentChatInterface
```typescript
const renderToolResult = useCallback((tool: StreamingTool) => {
  const { name, result } = tool

  // Detect special formats
  if (result.startsWith('[UPDATE_CONFIG:')) {
    // Extract data
    const updateData = JSON.parse(result.slice(15, -1))

    // Trigger callback (preview update)
    if (onConfigUpdate && !processedToolsRef.current.has(tool.id)) {
      processedToolsRef.current.add(tool.id)
      setTimeout(() => onConfigUpdate(updateData), 0)
    }

    // Return null to hide visual (or return component to show)
    return null
  }

  if (result.startsWith('[ASK_USER:')) {
    // Parse and render as interactive question
    const questionData = JSON.parse(result.slice(10, -1))
    return <InteractiveQuestion data={questionData} onSelect={handleSelect} />
  }

  // Default rendering
  return <ToolLine tool={tool} />
}, [onConfigUpdate])

// Pass to ChatMessages
<ChatMessages messages={messages} renderToolResult={renderToolResult} />
```

### Usage in ChatMessages
```typescript
// chat-messages.tsx
if (renderToolResult) {
  const customRender = renderToolResult(toolData)
  // Check !== undefined (not just truthiness!)
  // This allows returning null to explicitly hide
  return customRender !== undefined ? customRender : <ToolLine tool={toolData} />
}
```

## Common Mistakes to Avoid

### ❌ Mistake 1: Forgetting Text Block
```typescript
// Tool marker extracted and removed from text
contentBlocks = [{ type: 'tool_use', ... }]
// ❌ Forgot to add text block!
// Result: Tool renders (maybe hidden), but conversation disappears
```

**Fix**: Always add text block when you have both tools and text
```typescript
if (contentBlocks && textContent) {
  contentBlocks.push({ type: 'text', text: textContent })
}
```

### ❌ Mistake 2: Setting content to Empty String
```typescript
send({
  type: 'done',
  message: {
    content: contentBlocks ? '' : finalContent,  // ❌ Empty when contentBlocks exist
    contentBlocks,
  }
})
```

**Why it's wrong**: Even though content is ignored when contentBlocks exist, it's confusing and may break if Message component logic changes.

**Fix**: Set content to the actual text
```typescript
content: contentBlocks ? textContent : finalContent,
```

### ❌ Mistake 3: Missing renderToolResult Callback
```typescript
// ❌ No callback passed
<ChatMessages messages={messages} />
// Result: Special formats show as raw JSON
```

**Fix**: Always pass renderToolResult when using special format markers
```typescript
<ChatMessages messages={messages} renderToolResult={renderToolResult} />
```

### ❌ Mistake 4: Checking Truthiness Instead of !== undefined
```typescript
// ❌ Wrong check
if (customRender) return customRender
// Problem: null is falsy, falls through to default

// ✅ Correct check
if (customRender !== undefined) return customRender
// Allows null to mean "explicitly hide"
```

## Architecture Patterns

### Pattern 1: Build Mode (AgentChatInterface → ChatMessages)
- ✅ Has renderToolResult callback
- ✅ Detects special formats (UPDATE_CONFIG, ASK_USER, etc.)
- ✅ Handles contentBlocks properly
- ✅ Works correctly

### Pattern 2: Test Mode Page (AgentChatInterface)
- ✅ Uses AgentChatInterface directly
- ✅ Has all special format handling
- ✅ Works correctly

### Pattern 3: Test Mode Refine Panel (TestPanel → ChatMessages)
- ❌ Missing renderToolResult callback
- ❌ Shows raw JSON for special formats
- **Needs fix**: Add renderToolResult callback

## Files Modified to Fix Issues

### 1. services/agents/main.py (Lines 319-377)
**Change**: Extract tool results from SDK and append to content
```python
# Extract tool results for frontend detection
tool_results = []
if hasattr(result, 'new_items') and result.new_items:
    for item in result.new_items:
        if type(item).__name__ == 'ToolCallOutputItem':
            if hasattr(item, 'output'):
                output = str(item.output)
                if output.startswith('[UPDATE_CONFIG:') or \
                   output.startswith('[ASK_USER:'):
                    tool_results.append(output)

if tool_results:
    content = content + '\n' + '\n'.join(tool_results)
```

### 2. app/api/chat/stream/route.ts (Lines 273-287)
**Change**: Add text block to contentBlocks
```typescript
if (toolBlocks.length > 0) {
  contentBlocks = toolBlocks
}

textContent = textContent.trim()

// Add text block if we have text content
if (contentBlocks && textContent) {
  contentBlocks.push({
    type: 'text',
    text: textContent,
  })
}
```

### 3. app/api/chat/stream/route.ts (Line 302)
**Change**: Use textContent instead of empty string
```typescript
content: contentBlocks ? textContent : finalContent,
```

### 4. components/shared/chat/chat-messages.tsx (Lines 189-202)
**Change**: Check !== undefined instead of truthiness
```typescript
if (renderToolResult) {
  const customRender = renderToolResult(toolData)
  return customRender !== undefined ? customRender : <ToolLine ... />
}
```

## Fixed Issues

### Issue 3: Malformed JSON Showing as Raw Text ✅
**File**: `app/api/chat/stream/route.ts` (Line 252)

**Problem**: When ask_user or other tool markers contained malformed JSON, parsing would fail and the raw marker would show as text.

**Root Cause**:
1. Tool marker detected by regex
2. JSON parsing attempted: `JSON.parse(jsonStr)`
3. Parsing fails with syntax error
4. Exception caught, falls through
5. But marker was never removed from textContent!
6. Raw marker rendered as text

**Solution**: Move marker removal BEFORE JSON parsing
```typescript
// Remove marker FIRST (before parsing)
textContent = textContent.replace(match, '')

try {
  const parsedData = JSON.parse(jsonStr)
  // Create tool block...
} catch (parseError) {
  // JSON parsing failed - marker already removed, just log error
  console.error(`[ERROR] Failed to parse ${toolType} JSON:`, parseError)
}
```

This ensures markers are always removed, even if JSON is malformed.

### Issue 4: Regex Matching Nested JSON Incorrectly ✅
**File**: `app/api/chat/stream/route.ts` (Lines 234-265)

**Problem**: Regex `/\[([A-Z_]+):\{[^]*?\}\]/g` uses non-greedy matching which stops at the first `}]` it finds. For ask_user with nested options array, this matches partway through the JSON instead of at the end.

**Example**:
```
[ASK_USER:{"type": "ask_user", "options": [{"label": "Women"}], "allowMultiple": false}]
                                          stops here ----^
                                          should stop here -----------^
```

**Evidence**:
```
[DEBUG] Extracted 2 tool marker(s), remaining text: , "allowMultiple": false}]
```

**Solution**: Replaced regex with brace-counting algorithm
```typescript
const extractToolMarkers = (content: string): string[] => {
  const markers: string[] = []
  const toolNamePattern = /\[([A-Z_]+):\{/g
  let match: RegExpExecArray | null

  while ((match = toolNamePattern.exec(content)) !== null) {
    const startIndex = match.index
    let braceCount = 1
    let currentIndex = match.index + match[0].length // Start after [TOOL_NAME:{

    // Count braces to find the matching }]
    while (currentIndex < content.length && braceCount > 0) {
      const char = content[currentIndex]
      if (char === '{') {
        braceCount++
      } else if (char === '}') {
        braceCount--
        // Check if this is followed by ]
        if (braceCount === 0 && content[currentIndex + 1] === ']') {
          // Found the complete marker
          const endIndex = currentIndex + 2 // Include }]
          markers.push(content.substring(startIndex, endIndex))
          break
        }
      }
      currentIndex++
    }
  }

  return markers
}

const toolMatch = extractToolMarkers(finalContent)
```

This properly handles nested JSON structures by counting opening and closing braces.

## Remaining Issues to Fix

### Test Panel Missing renderToolResult
**File**: `app/apps/studio/[id]/refine/_components/test-panel.tsx`

**Current**:
```tsx
<ChatMessages
  messages={messages}
  onAskUserSelect={onAskUserSelect}
  // ❌ Missing renderToolResult
/>
```

**Needed**:
```tsx
<ChatMessages
  messages={messages}
  onAskUserSelect={onAskUserSelect}
  renderToolResult={renderToolResult}  // ← Add this
/>
```

Then define renderToolResult callback similar to AgentChatInterface.

## Key Takeaways

1. **ContentBlocks override content** - If contentBlocks exist, content is ignored
2. **Always include text block** - When you have tools + text, add both to contentBlocks
3. **renderToolResult is critical** - Without it, special formats show as raw JSON
4. **Check !== undefined** - Allows null to mean "explicitly hide"
5. **Extract tool results from SDK** - OpenAI SDK doesn't include them in final_output when agent continues

## Related Files

- `components/shared/agent/agent-chat-interface.tsx` - renderToolResult implementation
- `components/shared/chat/chat-messages.tsx` - ContentBlocks rendering logic
- `app/api/chat/stream/route.ts` - SSE streaming and contentBlocks creation
- `services/agents/main.py` - Tool result extraction from SDK
- `app/apps/studio/[id]/refine/_components/test-panel.tsx` - Needs fix

---

**Last Updated**: 2025-12-29
**Issue**: Agent Builder rendering bugs
**Status**:
- Build mode preview + conversation continuation ✅
- Tool markers properly extracted from Python SDK ✅
- ContentBlocks with text block rendering ✅
- Malformed JSON markers removed from text ✅
- Nested JSON tool markers properly extracted ✅
- Test mode needs renderToolResult callback ❌
