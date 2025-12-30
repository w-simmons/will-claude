---
name: "audit-rules"
description: "Audit all Claude Code rules and skills for accuracy against current codebase"
arguments: []
---

# Command: audit-rules

Comprehensive audit of all rules and skills to ensure they match the current codebase and are up-to-date.

## Usage

```bash
/audit-rules
```

No arguments needed - audits all rules and skills automatically.

## What This Command Does

Performs comprehensive audit checking:

1. **Rule Examples** - Verify code examples match current codebase
2. **File References** - Check referenced files still exist
3. **Pattern Accuracy** - Ensure patterns match reality
4. **Conflicting Guidance** - Find contradictions between rules
5. **Path Patterns** - Verify YAML frontmatter globs are accurate
6. **Outdated Content** - Identify stale information
7. **Missing Patterns** - Find common patterns not documented

## Implementation Steps

### 1. Read All Rules and Skills

```typescript
// Read all files
const ruleFiles = glob('.claude/rules/*.md')
const skillFiles = glob('.claude/skills/*.md')

// Parse each file
for (const file of [...ruleFiles, ...skillFiles]) {
  const content = readFile(file)
  // Analyze content
}
```

### 2. Check Rule Examples Against Codebase

For each code example in rules:

#### Extract Code Examples

Find all code blocks:
```markdown
```typescript
// Example code
```
```

#### Verify Against Real Code

- **If example references a file** (e.g., "from app/apps/chat/actions.ts")
  - Read the actual file
  - Check if pattern still exists
  - Flag if file doesn't exist or pattern changed

- **If example shows a pattern** (e.g., "Server Actions use Zod")
  - Search codebase for counter-examples
  - Verify pattern is still used
  - Flag if pattern is obsolete

#### Check Imports

- Verify import paths in examples are valid
- Check if imported modules still exist
- Flag if imports are broken

### 3. Verify File Paths

For each file path mentioned in rules:

```bash
# Check if paths exist
test -e [path]
```

**Paths to check:**
- Direct file references: "See `app/apps/chat/page.tsx`"
- Directory references: "Create in `lib/db/schemas/`"
- Example paths: "app/apps/[slug]/page.tsx"

**Flag if:**
- ❌ Specific file doesn't exist
- ❌ Directory doesn't exist
- ⚠️ Example path pattern seems wrong

### 4. Validate Path Patterns (YAML Frontmatter)

For each rule with `paths:` in frontmatter:

```yaml
paths:
  - "**/_components/**"
  - "**/page.tsx"
```

**Check:**
1. Do these glob patterns match actual files?
   ```bash
   # Find files matching pattern
   find . -path "**/_components/**" -type f
   ```

2. Are there files that should match but don't?
3. Are patterns too broad or too narrow?

**Flag if:**
- ❌ Pattern matches 0 files (likely wrong)
- ⚠️ Pattern matches 1000+ files (too broad)
- ⚠️ Obvious files don't match (pattern incomplete)

### 5. Find Conflicting Guidance

Compare rules for contradictions:

**Example conflicts:**
- Rule A says: "Always use X"
- Rule B says: "Prefer Y over X"

**Check:**
- Same topic covered differently in multiple rules
- Different recommendations for same scenario
- Contradictory examples

**How to detect:**
- Extract key recommendations from each rule
- Compare similar topics
- Use LLM to identify contradictions:
  ```typescript
  const result = await llm.ask(
    `Do these rules conflict? Rule 1: ${rule1} Rule 2: ${rule2}`,
    { model: 'claude-haiku' }
  )
  ```

### 6. Identify Missing Patterns

Search codebase for common patterns not documented:

**Patterns to look for:**

1. **Repeated code structures** (appear 3+ times)
   - Search for similar file structures
   - Look for repeated imports
   - Find common component patterns

2. **Undocumented conventions**
   - Naming patterns used but not documented
   - Directory structures not in rules
   - Configuration patterns

3. **New technologies/libraries**
   - Check package.json for new dependencies
   - Look for imports not covered in rules

**Examples:**
```bash
# Find all Server Actions
grep -r "'use server'" app/

# Find common import patterns
grep -r "^import.*from '@/lib/" --include="*.ts" | sort | uniq -c | sort -rn

# Find component patterns
find . -name "_components" -type d
```

### 7. Check Verification Dates

For each rule/skill:

```markdown
**Last Verified**: 2025-12-29
<!-- Last verified: 2025-12 -->
```

**Flag if:**
- ⚠️ No verification date
- ⚠️ Verification date > 3 months old
- 📝 Suggest updating after audit

### 8. Generate Audit Report

Create comprehensive report:

```markdown
## Rule Audit Results
**Generated**: [date]

### 📊 Summary
- **Rules Audited**: [count]
- **Skills Audited**: [count]
- **Total Issues**: [count]
- **Total Suggestions**: [count]

### ✅ Current and Accurate

**Rules:**
- `core-conventions.md` - All examples match codebase
- `frontend-patterns.md` - Patterns are current
- [List all good rules]

**Skills:**
- `add-app.md` - Up to date
- [List all good skills]

### ⚠️ Needs Updates

#### `backend-patterns.md`
- **Issue**: Server Action example references old pattern
- **Location**: Line 42
- **Fix**: Update to show current revalidatePath pattern
- **Example File**: See `app/apps/chat/actions.ts:15` for current pattern

#### `database-patterns.md`
- **Issue**: Missing example for relation queries
- **Suggestion**: Review other services for relation query examples

[Continue for each rule/skill needing updates]

### ❌ References Non-Existent Files/Patterns

#### `component-patterns.md`
- **Issue**: References `app/apps/old-app/` which no longer exists
- **Location**: Line 123
- **Fix**: Remove example or update to current app

#### `llm-patterns.md`
- **Issue**: Shows OpenAI pattern but OpenAI was removed
- **Fix**: Remove OpenAI examples, keep only Anthropic/Google

[Continue for each broken reference]

### ➕ Missing Rules/Patterns

#### Pattern: Error Boundaries
- **Found in**: Apps use error boundaries
- **Not documented in**: Any rule
- **Suggestion**: Add section to `frontend-patterns.md` or create new rule
- **Example Files**:
  - `app/apps/chat/error.tsx`

#### Pattern: Streaming Responses
- **Found in**: Used in chat app and others
- **Not documented in**: `llm-patterns.md`
- **Suggestion**: Add streaming examples

[Continue for each missing pattern]

### 🔄 Conflicting Guidance

#### frontend-patterns.md vs component-patterns.md
- **Conflict**: Different form validation approaches suggested
- **frontend-patterns.md**: Says use react-hook-form
- **component-patterns.md**: Shows manual validation
- **Resolution**: Standardize on react-hook-form, update component-patterns

[Continue for each conflict]

### 📂 Path Pattern Issues

#### `frontend-patterns.md`
```yaml
paths: ["**/_components/**", "**/page.tsx"]
```
- ✅ Matches 45 files
- ✅ Pattern seems accurate

#### `mcp-setup.md`
```yaml
paths: [".claude/**", ".mcp.json"]
```
- ⚠️ Only matches 12 files, might be too narrow
- **Suggestion**: Consider adding `.claude/settings.json` explicitly

[Continue for each rule with paths]

### 📅 Verification Dates

**Recently Verified (< 1 month):**
- `core-conventions.md` - 2025-12-24
- [List others]

**Needs Verification (> 3 months):**
- `old-rule.md` - 2025-08-15
- [List others]

**No Verification Date:**
- [List any without dates]

### 🎯 Recommended Actions

#### High Priority
1. Fix broken file references in `component-patterns.md`
2. Remove OpenAI examples from `llm-patterns.md`
3. Resolve form validation conflict between rules

#### Medium Priority
1. Add error boundary pattern to `frontend-patterns.md`
2. Update verification dates for rules > 3 months old
3. Add missing streaming examples to `llm-patterns.md`

#### Low Priority
1. Review path patterns for accuracy
2. Consider splitting large rules
3. Add more cross-references between rules

### 📋 Action Items

- [ ] Update `backend-patterns.md` line 42
- [ ] Remove `app/apps/old-app/` reference from `component-patterns.md`
- [ ] Add error boundary section to `frontend-patterns.md`
- [ ] Remove OpenAI from `llm-patterns.md`
- [ ] Standardize form validation across rules
- [ ] Update verification dates for 3 rules
- [ ] Add streaming examples to `llm-patterns.md`

---

**Audit completed**: [date]
**Next audit recommended**: [date + 1 month]
```

### 9. Interactive Mode (Optional)

Ask user after report:

```
Audit complete! Found [X] issues and [Y] suggestions.

Would you like me to:
1. Fix issues automatically
2. Update verification dates
3. Create detailed fix plan
4. Export report to file

Choose an option or type 'done' to finish.
```

## Success Criteria

**Good audit should:**
- ✅ Check every rule and skill
- ✅ Verify all code examples
- ✅ Check all file references
- ✅ Find contradictions
- ✅ Suggest missing patterns
- ✅ Provide actionable fixes

**Output should:**
- 📊 Clear summary of issues
- 🎯 Prioritized action items
- 📝 Specific line numbers and fixes
- 💡 Helpful suggestions
- ✅ List of rules that are good

## References

- See `.claude/rules/rule-maintenance.md` for rule lifecycle
- See existing rules in `.claude/rules/` for audit targets
- See existing skills in `.claude/skills/` for audit targets
