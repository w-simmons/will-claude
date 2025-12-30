---
name: "review-pr"
description: "Comprehensive code review for a pull request"
arguments:
  - name: "pr-number"
    description: "Pull request number to review (optional, defaults to current branch)"
    required: false
---

# Command: review-pr

Perform a comprehensive code review of a pull request, checking against project rules and best practices.

## Usage

```bash
/review-pr [pr-number]
/review-pr  # Reviews current branch against main
```

**Examples:**
- `/review-pr 42` - Review PR #42
- `/review-pr` - Review current branch changes

## What This Command Does

Performs automated code review checking:

1. **Code Quality** - Adherence to project patterns
2. **Rule Compliance** - Matches `.claude/rules/` guidelines
3. **Test Coverage** - Tests exist for new features
4. **Documentation** - README updates for app changes
5. **Code Cleanliness** - No commented code, unused imports
6. **Best Practices** - Follows established patterns

## Implementation Steps

### 1. Get PR Information

**If pr-number provided:**
```bash
gh pr view [pr-number] --json number,title,body,files
```

**If no pr-number (review current branch):**
```bash
# Get current branch
git branch --show-current

# Get diff against main
git diff main...HEAD
```

### 2. Analyze Changed Files

For each changed file:

#### Check File Type and Apply Relevant Rules

- `**/page.tsx`, `**/layout.tsx` → Check against `frontend-patterns.md`
- `**/actions.ts` → Check against `backend-patterns.md`
- `**/_services/**` → Check against `backend-patterns.md`, `llm-patterns.md`
- `lib/db/schemas/**` → Check against `database-patterns.md`
- `**/_components/**` → Check against `component-patterns.md`
- `components/**` → Check against `component-patterns.md`

#### Common Checks for All Files

1. **No commented-out code**
   - Search for large blocks of `//` or `/* */` comments
   - Flag if found (exceptions: TODO comments, JSDoc)

2. **No unused imports**
   - Check for imports not used in file
   - Flag if found

3. **No console.log in production code**
   - Search for `console.log`, `console.warn` (not in `console.error`)
   - Flag if found in non-development code

4. **Proper error handling**
   - Server Actions have try/catch
   - Services throw/return errors properly

5. **TypeScript best practices**
   - No `any` types (exceptions: rare edge cases)
   - Proper type definitions
   - Uses inferred types from schemas

### 3. Check for Tests

If new features added:
- Check for corresponding test files
- Verify tests cover new functionality
- Flag if missing

### 4. Check Documentation

If app files changed:
- Verify `app/apps/[slug]/README.md` exists
- Check if README updated for new features
- Verify README sections are complete

If shared components added:
- Check if component documented (JSDoc or README)

### 5. Check Specific Patterns

#### Server Actions (`**/actions.ts`)

✅ Has 'use server' directive
✅ Uses Zod validation
✅ Calls services (not direct DB access)
✅ Uses revalidatePath after mutations
✅ Returns { data } or { error }

#### Services (`**/_services/**`)

✅ No Next.js imports (portable)
✅ Uses `.service.ts` suffix
✅ Checks `if (!db)` before DB access
✅ Proper error handling

#### Components (`**/_components/**`, `components/**`)

✅ Uses Shadcn components (not custom UI primitives)
✅ 'use client' only when needed
✅ Proper TypeScript props interfaces

#### Database Schemas (`lib/db/schemas/**`)

✅ Exported from `lib/db/schema.ts`
✅ Follows naming: `app_[slug]_[table]`
✅ Has type exports: `Type` and `NewType`

#### LLM Usage

✅ Uses `llm` from `@/lib/llm` (not creating new clients)
✅ Proper model selection
✅ Error handling for missing API keys

### 6. Generate Review Report

Create comprehensive report:

```markdown
## Code Review: [PR Title or Branch Name]

### Summary
- **Files Changed**: [count]
- **Lines Added**: [count]
- **Lines Removed**: [count]

### ✅ Looks Good
- [List things that follow patterns well]
- [Highlight good practices]

### ⚠️ Suggestions
- [List minor improvements]
- [Non-blocking issues]

### ❌ Issues Found
- [List problems that should be fixed]
- [Rule violations]

### 📋 Checklist

- [ ] All files follow relevant patterns from `.claude/rules/`
- [ ] No commented-out code
- [ ] No unused imports
- [ ] Tests added for new features
- [ ] README.md updated (if app changed)
- [ ] Server Actions use revalidatePath
- [ ] Services are portable (no Next.js imports)
- [ ] Components use Shadcn (no custom UI primitives)
- [ ] Database schemas registered in schema.ts
- [ ] LLM calls use lib/llm (not creating clients)

### 📝 File-by-File Review

#### [file-path]
- **Pattern**: [Which rule applies]
- **Status**: ✅ Good / ⚠️ Suggestions / ❌ Issues
- **Notes**: [Specific feedback]

[Repeat for each file]

### 🚀 Next Steps

1. [Action item 1]
2. [Action item 2]
3. [Action item 3]

### 📚 References

- Relevant rules: [List rules that apply]
- See examples: [Point to similar code in codebase]
```

### 7. Post Review (If PR Number Provided)

**Option 1:** Create review comment
```bash
gh pr review [pr-number] --comment -b "[review content]"
```

**Option 2:** Just output to console (let user decide)

Ask user: "Would you like me to post this review as a PR comment?"

## Example Output

```
📊 Code Review Complete!

✅ Looks Good:
- Server Actions properly use Zod validation
- Services are portable (no Next.js imports)
- Components use Shadcn UI properly

⚠️ Suggestions:
- Consider adding error boundary in page.tsx:15
- Could extract repeated form logic to shared component

❌ Issues Found:
- actions.ts:42 - Missing revalidatePath after mutation
- my.service.ts:18 - Uses Next.js import (should be portable)
- MyComponent.tsx:25 - Custom button instead of Shadcn Button

📋 Overall: 3 issues, 2 suggestions

See full report above for details and next steps.
```

## References

- See `.claude/rules/` for all pattern guidelines
- See `.claude/commands/audit-rules.md` for rule verification
- Use GitHub CLI (`gh`) for PR interaction
