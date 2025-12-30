---
name: "add-skill"
description: "Create a new Claude Code agent skill from template"
arguments:
  - name: "skill-name"
    description: "Name of the skill (e.g., 'testing', 'deployment')"
    required: true
---

# Command: add-skill

Create a new Claude Code agent skill file from template with proper structure.

## Usage

```bash
/add-skill [skill-name]
```

**Examples:**
- `/add-skill testing` - Create testing skill
- `/add-skill deployment` - Create deployment skill
- `/add-skill api-design` - Create API design skill

## What This Command Does

Creates a new skill file at `.claude/skills/[skill-name].md` with:

1. Proper frontmatter with triggers
2. Template structure (Quick Reference, Common Patterns, Templates)
3. Verification date set to current month
4. Ready for editing with placeholder content

## Implementation Steps

### 1. Validate Skill Name

- Must be kebab-case (lowercase, hyphens only)
- Must not already exist in `.claude/skills/`
- Suggest correction if invalid format

### 2. Prompt for Skill Details

Use `AskUserQuestion` to gather:

**Question 1: What triggers should activate this skill?**
- Prompt: "What keywords or contexts should trigger this skill?"
- Example: "API, REST, endpoint, routes"
- This becomes the trigger comment

**Question 2: What's the primary purpose?**
- Prompt: "Brief description of what this skill helps with (1 sentence)"
- Example: "Patterns for designing REST APIs with proper error handling"

**Question 3: What category?**
- Options:
  - "Implementation" - Code patterns and templates
  - "Architecture" - Design and structure guidance
  - "Workflow" - Process and development flow
  - "Integration" - Third-party services and APIs

### 3. Create Skill File

Create `.claude/skills/[skill-name].md`:

```markdown
# Skill: [Title Case Name]
<!-- Triggers: [user-provided triggers] -->
<!-- Last verified: [YYYY-MM] -->

## Quick Reference

- [Key principle 1]
- [Key principle 2]
- [Key principle 3]
- [Add your quick reference points here]

## Common Patterns

### Pattern Name

[Describe the pattern]

```[language]
// Add code example here
```

[When to use this pattern]

## Templates

### Template Name

```[language]
// Add template code here
```

**Usage:**
- [Step 1]
- [Step 2]
- [Step 3]

## References

- See `.claude/rules/[related-rule].md` for comprehensive patterns
- [Add other references]
```

### 4. Set Proper Metadata

- **Triggers**: User-provided keywords (comma-separated)
- **Last verified**: Current date in YYYY-MM format (e.g., "2025-12")
- **Title**: Convert kebab-case to Title Case (e.g., "api-design" → "API Design")

### 5. Open for Editing

After creation:

```
✅ Skill created: .claude/skills/[skill-name].md

📝 Template created with:
- Triggers: [triggers]
- Purpose: [purpose]
- Category: [category]

Next steps:
1. Open the file and fill in the template sections
2. Add relevant code examples and patterns
3. Include references to related rules
4. Test triggers by asking Claude Code about [trigger keywords]

File ready for editing!
```

Use `Read` tool to display the created file to user.

## Example Skill Template

For `/add-skill testing` with triggers "test, testing, jest, vitest":

```markdown
# Skill: Testing
<!-- Triggers: test, testing, jest, vitest, unit test, integration test -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- Use Vitest for unit and integration tests
- Test files: `*.test.ts` or `*.spec.ts`
- Mock external dependencies (DB, APIs, LLMs)
- Test both success and error cases

## Common Patterns

### Unit Test Pattern

```typescript
import { describe, it, expect } from 'vitest'
import { myFunction } from './my-module'

describe('myFunction', () => {
  it('should return expected result', () => {
    const result = myFunction('input')
    expect(result).toBe('expected')
  })

  it('should handle edge cases', () => {
    const result = myFunction('')
    expect(result).toBe('default')
  })
})
```

### Service Test with DB Mock

```typescript
import { describe, it, expect, vi } from 'vitest'
import { myService } from './my.service'

// Mock database
vi.mock('@/lib/db', () => ({
  db: {
    select: vi.fn().mockReturnValue({
      from: vi.fn().mockReturnValue({
        where: vi.fn().mockResolvedValue([{ id: '1', name: 'Test' }])
      })
    })
  },
  eq: vi.fn()
}))

describe('myService', () => {
  it('should fetch items', async () => {
    const items = await myService.getAll()
    expect(items).toHaveLength(1)
    expect(items[0].name).toBe('Test')
  })
})
```

## Templates

### Test File Template

```typescript
import { describe, it, expect, beforeEach, afterEach } from 'vitest'

describe('[Module Name]', () => {
  beforeEach(() => {
    // Setup before each test
  })

  afterEach(() => {
    // Cleanup after each test
  })

  describe('[Function/Feature Name]', () => {
    it('should [expected behavior]', () => {
      // Arrange
      const input = 'test'

      // Act
      const result = myFunction(input)

      // Assert
      expect(result).toBe('expected')
    })

    it('should handle [edge case]', () => {
      // Test edge case
    })

    it('should throw [error] when [condition]', () => {
      expect(() => myFunction(null)).toThrow('Error message')
    })
  })
})
```

## References

- See project test files for examples
- Vitest docs: https://vitest.dev/
```

## Skill Best Practices

Skills should be:

✅ **Concise** - ~100-150 lines, quick reference not comprehensive docs
✅ **Action-oriented** - Templates and patterns, not theory
✅ **Trigger-focused** - Clear keywords for auto-discovery
✅ **Example-rich** - Real code examples from the project
✅ **Referenced** - Link to related rules for deep dives

❌ **Not novels** - Keep comprehensive docs in rules
❌ **Not outdated** - Update verification date when editing
❌ **Not vague** - Specific, actionable guidance

## References

- See existing skills in `.claude/skills/` for examples
- See `.claude/rules/rule-maintenance.md` for skill lifecycle
- Skill format: Quick Reference → Common Patterns → Templates → References
