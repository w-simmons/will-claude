---
priority: low
---

# Codebase Cleanliness

**Always Active (Low Priority)** - Background guidance for maintaining a clean codebase

## Purpose

This rule enforces codebase cleanliness by ensuring redundant files, unused code, and temporary artifacts are removed promptly.

## Key Principles

1. **Never leave redundant or old files**
2. **Remove unused imports, components, services**
3. **Delete deprecated code immediately**
4. **Clean up temporary files and test artifacts**
5. **Remove commented-out code** (use git history instead)

## File Cleanup Rules

### Remove Unused Files

**Check for:**
- Files with no imports/references
- Orphaned components/services
- Old test files
- Temporary scripts

**How to identify:**
```bash
# Find files with no imports (manual check)
# Search codebase for file name
# Check git history if unsure
```

**Action:**
- Delete immediately if confirmed unused
- Use git history if you need to reference later

### Remove Unused Imports

**Check:**
- Unused imports in all files
- Type-only imports that aren't used
- Duplicate imports

**Example:**
```typescript
// ❌ Bad: Unused imports
import { useState, useEffect, useMemo } from 'react'
// Only useState is used

// ✅ Good: Only import what's used
import { useState } from 'react'
```

### Remove Commented-Out Code

**Rule**: Never leave commented-out code

```typescript
// ❌ Bad: Commented code
// const oldFunction = () => {
//   // old implementation
// }

// ✅ Good: Delete it, use git history if needed
// Code removed - see git history for reference
```

**Why:**
- Git history preserves old code
- Commented code adds noise
- Makes code harder to read

### Remove Deprecated Code

**When code is deprecated:**
1. Remove it immediately
2. Don't leave "deprecated" comments
3. Use git history for reference
4. Update related rules/docs if needed

## Dependency Cleanup

### Remove Unused Dependencies

**Check `package.json`:**
- Dependencies not imported anywhere
- Dev dependencies not used
- Old/unused packages

**How to check:**
```bash
# Use tools like depcheck
npx depcheck
```

**Action:**
- Remove unused dependencies
- Run `pnpm install` to update lockfile

### Remove Unused Environment Variables

**Check `lib/env.ts`:**
- Environment variables not used in codebase
- Old API keys
- Deprecated configs

**Action:**
- Remove from `lib/env.ts`
- Remove from `.env` files
- Update documentation

## Component Cleanup

### Remove Unused Components

**Check:**
- Components with no imports
- App-specific components not used
- Shared components not used by any app

**Action:**
- Delete unused components
- Move to shared if needed by another app

### Consolidate Duplicate Functionality

**When you find:**
- Similar components doing the same thing
- Duplicate utility functions
- Repeated patterns

**Action:**
- Consolidate into one component/function
- Move to shared if used by multiple apps
- Update all references

## Service Cleanup

### Remove Unused Services

**Check:**
- Services with no imports
- Service functions not called
- Old service files

**Action:**
- Delete unused services
- Remove unused service functions

## Regular Cleanup Checklist

### After Feature Completion

- [ ] Remove temporary files
- [ ] Remove commented-out code
- [ ] Remove unused imports
- [ ] Remove unused components
- [ ] Remove unused services
- [ ] Remove test artifacts

### Before Committing

- [ ] No commented-out code
- [ ] No unused imports
- [ ] No temporary files
- [ ] No console.logs (unless intentional)
- [ ] No TODO comments (create issue instead)

### Periodic Review (Monthly)

- [ ] Check for orphaned files
- [ ] Review unused dependencies
- [ ] Check for duplicate code
- [ ] Review environment variables
- [ ] Check for deprecated patterns

## When to Archive vs Delete

### Delete Immediately

- Unused code
- Temporary files
- Test artifacts
- Commented code
- Old implementations

### Archive (Rare Cases)

- Large data files (move to separate storage)
- Historical documentation (move to docs/archive)
- Legacy configs (document, then delete)

**Rule**: Prefer delete over archive. Git history is your archive.

## Common Mistakes

### ❌ Leaving Temporary Files

```typescript
// ❌ Bad: Temporary test file left in codebase
// test-temp.ts
export function tempTest() { ... }

// ✅ Good: Delete immediately after use
```

### ❌ Commenting Instead of Deleting

```typescript
// ❌ Bad: Commented code
// function oldImplementation() {
//   // old code
// }

// ✅ Good: Delete it
// Removed - see git history
```

### ❌ Keeping Unused Dependencies

```json
// ❌ Bad: Unused dependency
{
  "dependencies": {
    "old-package": "^1.0.0" // Not used anywhere
  }
}

// ✅ Good: Remove it
```

## Claude Code Guidelines

When cleaning up code:
- **Suggest removal** of unused files/components
- **Flag commented-out code** for deletion
- **Identify duplicate functionality** for consolidation
- **Check if rules need updates** after cleanup
- **Verify no references** before suggesting deletion
- **Proactively clean up** as you work (background priority)

## References

- See `.claude/rules/rule-maintenance.md` for keeping rules updated
- Git history for reference: `git log --all --full-history -- <file>`

---

**Last Verified**: 2025-12-29
