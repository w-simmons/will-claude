---
paths:
  - "docs/**"
  - "*.md"
  - "README.md"
priority: normal
---

# Documentation Patterns

**Auto-applies to**: Documentation files

## Purpose

This rule ensures all documentation stays current and accurate.

## Key Principles

1. **Keep content accurate** - Documentation must match the actual codebase
2. **Keep examples current** - Documentation should match actual codebase
3. **Update when patterns change** - Docs evolve with the codebase
4. **Clear structure** - Use consistent formatting and headings

## Documentation Locations

### Primary Documentation

- **CLAUDE.md** - Main project context, always loaded by Claude Code
- **README.md** - GitHub project description, quick start

### Documentation Directory (`docs/`)

- **docs/README.md** - Documentation overview and organization
- **docs/setup/** - Setup and configuration guides
- **docs/TROUBLESHOOTING.md** - Common issues

### Claude Code Configuration

- **.claude/rules/** - Contextual rules (this directory)
- **.claude/skills/** - Auto-discovering skills
- **.claude/commands/** - Slash commands
- **.claude/settings.json** - Claude Code configuration

## Correct Terminology

### ✅ Use These Terms

- **Claude Code** - The CLI tool
- **.claude/** - Configuration directory
- **.claude/rules/** - Rules directory
- **.claude/skills/** - Skills directory
- **CLAUDE.md** - Primary context file
- **YAML frontmatter** - Rule metadata
- **Path-triggered rules** - Auto-applying rules
- **Auto-discovering skills** - Skills that trigger based on context

### ❌ Never Use These Terms

- **Cursor** - Deprecated, never reference
- **.cursorrules** - Deprecated, never reference
- **.cursor/** - Deprecated, never reference
- **Manual @mention** - Old pattern, never reference

## Documentation Update Triggers

Update documentation when:

- **Project structure changes** - New directories, moved files
- **Commands change** - New commands, changed syntax
- **Setup process changes** - New steps, different tools
- **Architecture changes** - Major refactors, new patterns
- **Tool names change** - CLI tools, dependencies
- **File paths change** - Moved directories, renamed files

## When Editing Documentation

### Check for Stale References

When editing any documentation file:

1. **Search for deprecated terms** - Remove any references to Cursor, .cursorrules, .cursor/
2. **Check file paths** - Verify all paths exist and are correct
3. **Test commands** - Ensure all command examples work
4. **Verify links** - Check internal links resolve correctly
5. **Update dates** - Add "Last Updated" if significant changes

### Example: Updating docs/README.md

```markdown
# Documentation

## Documentation Layers

### 1. Claude Code Rules (`.claude/rules/`)
**Purpose**: AI context for Claude Code
**Audience**: AI assistant (automatically applied)
**Format**: Comprehensive rules with YAML frontmatter

### 2. Skills (`.claude/skills/`)
**Purpose**: Quick implementation guides
**Audience**: AI assistant (auto-discovering)
**Format**: Concise guides (~100-150 lines)

### 3. Documentation (`docs/`)
**Purpose**: Human-readable guides
**Audience**: Developers
**Format**: Markdown files
```

## Common Stale Content Patterns

### ❌ Stale: Outdated File Paths

```markdown
<!-- ❌ Bad: Path doesn't exist -->
**See**: `docs/guides/OLD_GUIDE.md` directory
```

### ✅ Current: Valid Paths

```markdown
<!-- ✅ Good: Path exists -->
**See**: `.claude/rules/` directory
```

### ❌ Stale: Broken Links

```markdown
<!-- ❌ Bad: Linking to deleted file -->
See [DELETED_FILE.md](./DELETED_FILE.md)
```

### ✅ Current: Valid Links

```markdown
<!-- ✅ Good: Link exists -->
See `CLAUDE.md` for project context
```

## Documentation Structure Best Practices

### Use Clear Headings

```markdown
# Main Title

## Section

### Subsection

#### Detail
```

### Provide Examples

Always include code examples when documenting patterns:

```markdown
## Using Server Actions

Example:
\`\`\`typescript
export async function createItem(formData: FormData) {
  // Implementation
}
\`\`\`
```

### Link to Related Docs

```markdown
See [CLAUDE.md](./CLAUDE.md) for architecture details.
See `.claude/rules/backend-patterns.md` for server action patterns.
```

### Keep Tables Readable

```markdown
| Feature | Description |
|---------|-------------|
| Rules | Auto-applying patterns |
| Skills | Quick reference guides |
```

## Verification Checklist

Before committing documentation changes:

- [ ] No references to deprecated tools or patterns
- [ ] All file paths exist and are correct
- [ ] All code examples are current
- [ ] All links resolve correctly
- [ ] Commands have been tested
- [ ] Formatting is consistent
- [ ] Tables are aligned
- [ ] Code blocks have language tags

## Claude Code Guidelines

When editing documentation:

1. **Check for stale references** - Remove deprecated terms and broken links
2. **Verify file paths** - Ensure all paths are current
3. **Update examples** - Match current codebase patterns
4. **Test commands** - Verify all commands work
5. **Add context** - Explain why, not just what

When you notice stale content:

```
I notice this documentation has outdated references. Should I update it?
```

## References

- See `CLAUDE.md` for project context
- See `.claude/rules/rule-maintenance.md` for rule maintenance
- See `.claude/rules/codebase-cleanliness.md` for cleanup guidelines

---

**Last Verified**: 2025-12-29
