---
paths:
  - ".claude/rules/**"
  - ".claude/skills/**"
priority: high
---

# Rule Maintenance and Evolution

**Auto-applies to**: Claude Code rules and skills

## Purpose

This rule establishes a living documentation system for keeping Claude Code rules updated as the codebase evolves. It provides processes for capturing new patterns, identifying outdated rules, and conducting rule audits.

## Key Principles

1. **Rules must evolve** with the codebase
2. **Capture new patterns** during development
3. **Remove outdated patterns** promptly
4. **Regular audits** ensure accuracy
5. **Rules reference real code** from the codebase

## Update Triggers

### When Rules Need Updating

Rules should be updated when:

- **New pattern emerges** in 2+ places
- **Existing pattern changes** or is deprecated
- **Dependency/API changes** affect patterns
- **Breaking changes** in framework
- **New best practices** discovered
- **Codebase structure changes**
- **File paths change**

### Examples

```typescript
// New pattern detected in 2+ places
// Pattern: Using a new form validation approach
// Action: Add to frontend-patterns rule

// Existing pattern changes
// Pattern: Server Actions now use different return format
// Action: Update backend-patterns rule

// Dependency change
// Pattern: Drizzle API changed
// Action: Update database-patterns rule
```

## Capture Process

### During Development

**When you implement something:**
1. **Flag it**: "This pattern should be in rules"
2. **Add immediately** or create TODO
3. **Update related skills/docs**
4. **Verify** with existing codebase examples

### Step-by-Step

1. **Identify new pattern**
   - Notice you're doing something new
   - See pattern repeated 2+ times
   - Discover better approach

2. **Document immediately**
   - Add to appropriate rule
   - Include code example
   - Reference actual files

3. **Update related docs**
   - Update skills if needed
   - Update CLAUDE.md if major change
   - Update README.md if architectural

4. **Verify accuracy**
   - Check examples work
   - Ensure patterns match codebase
   - Test with real code

## Review Checklist

### Periodic Rule Review

**Check each rule:**

- [ ] Rule examples match current codebase
- [ ] File paths in rules still exist
- [ ] Rule descriptions match reality
- [ ] No conflicting guidance between rules
- [ ] Path patterns (YAML frontmatter) still accurate
- [ ] Code examples are current
- [ ] Patterns are still used in codebase

### Frequency

- **After major refactors**: Review all rules
- **Monthly**: Quick review of all rules
- **On-demand**: When patterns change
- **Use `/audit-rules` command** for automated audit

## Rule Lifecycle

### New Pattern Detected

**Action**: Add to appropriate rule or create new rule

```markdown
## New Pattern: [Pattern Name]

[Description]

```typescript
// Example from app/apps/example/...
```

**When to use**: [Guidance]
```

### Pattern Changes

**Action**: Update rule with new examples

```markdown
## Pattern: [Pattern Name]

**Last Updated**: 2025-12-29

[New description]

```typescript
// New example
```

**Previous approach**: See git history for old pattern
```

### Pattern Removed

**Action**: Remove from rule entirely

Don't leave deprecated content - use git history for reference.

### Rule Becomes Too Large

**Action**: Split into focused sub-rules

- Identify distinct topics
- Create new rule files with proper YAML frontmatter
- Update rule descriptions
- Cross-reference between rules

### Rules Overlap

**Action**: Merge or clarify boundaries

- Identify overlap
- Decide which rule owns the pattern
- Remove duplicate content
- Add cross-references

## Claude Code Guidelines

### When Implementing Something Different from Rules

**Action**: Suggest rule update

```
I notice we're implementing this differently than the rules suggest.
The current pattern is: [pattern]
Should I update the [rule-name] rule to reflect this?
```

### When Rules Conflict with Codebase

**Action**: Flag for review

```
The [rule-name] rule suggests [pattern], but the codebase uses [different-pattern].
Should I update the rule to match the codebase?
```

### When New Pattern Appears

**Action**: Propose adding to rules

```
I see this pattern appearing in multiple places:
[pattern description]
Should I add this to the [rule-name] rule?
```

### When Cleaning Up Code

**Action**: Check if rules need updates

```
I'm removing [component/file/pattern]. Should I also update the rules that reference it?
```

### When Removing Files

**Action**: Verify rules don't reference them

```
I'm deleting [file]. Let me check if any rules reference it...
[Check results]
```

## Rule Audit Process

### How to Request Audit

**Use `/audit-rules` command** or ask:
- "Review rules against codebase"
- "Audit rules"
- "Check if rules are up to date"

### Claude Code Review Steps

1. **Check each rule's examples** against actual codebase files
   - Verify file paths exist
   - Check code examples are current
   - Ensure patterns match reality

2. **Verify all file paths** in rules still exist
   - Check imports in rules
   - Verify directory structures
   - Confirm file locations

3. **Identify patterns** in codebase not documented in rules
   - Find repeated patterns
   - Check if they should be in rules
   - Suggest additions

4. **Find rules** that reference non-existent patterns/files
   - Check for broken references
   - Identify outdated examples
   - Flag for removal/update

5. **Check for conflicting guidance** between rules
   - Compare rule recommendations
   - Identify contradictions
   - Suggest clarifications

6. **Validate path patterns** (YAML frontmatter) match current structure
   - Check glob patterns
   - Verify they match file structure
   - Update if needed

7. **Suggest updates, deletions, or new rules**
   - Provide specific recommendations
   - Include code examples
   - Reference actual files

8. **Provide summary report** of findings

### Output Format

**Summary Report:**

```
## Rule Audit Results

### ✅ Current and Accurate
- core-conventions: All examples match codebase
- frontend-patterns: Patterns are current

### ⚠️ Needs Updates
- backend-patterns: Server Action pattern changed (see app/apps/chat/actions.ts)
- database-patterns: Missing example for relations query

### ❌ References Non-Existent Patterns
- component-patterns: References removed component (old-form.tsx)

### ➕ Missing Rules
- New pattern: Error boundary usage (appears in 3+ places)

### 🔄 Conflicting Guidance
- frontend-patterns vs component-patterns: Different form patterns suggested
```

### Frequency

- **On-demand**: User requests audit via `/audit-rules`
- **After major refactors**: Suggested automatically
- **Monthly**: Periodic review recommended

## Rule Versioning

### Change Tracking

**In rule files:**
- Add "Last Verified: YYYY-MM-DD" footer
- Document in git commits
- Reference git history for changes

```markdown
---
paths: ["**/*.ts"]
priority: normal
---

# Rule Name

[Content]

---

**Last Verified**: 2025-12-29
```

## Best Practices

### Keep Rules Focused

- One rule = one concern
- Don't mix unrelated patterns
- Split large rules

### Use Real Examples

- Reference actual files
- Use code from codebase
- Keep examples current

### Regular Maintenance

- Review monthly
- Update immediately when patterns change
- Remove outdated content promptly

### YAML Frontmatter

- Use proper path patterns for auto-application
- Set appropriate priority levels
- Test glob patterns match target files

## References

- See `.claude/rules/codebase-cleanliness.md` for cleanup guidelines
- See `CLAUDE.md` for high-level patterns
- Git history for rule changes: `git log -- .claude/rules/`
- Use `/audit-rules` command for automated audits

---

**Last Verified**: 2025-12-29
