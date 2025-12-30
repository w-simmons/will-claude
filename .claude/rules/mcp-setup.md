---
paths:
  - ".claude/**"
  - ".mcp.json"
priority: normal
---

# MCP Setup and Usage

**Auto-applies to**: Claude Code configuration files

## Purpose

This rule covers Model Context Protocol (MCP) setup, configuration, available servers, and usage patterns in Claude Code.

## What is MCP?

Model Context Protocol (MCP) is a standard for connecting AI assistants to external tools and data sources. MCP servers provide additional capabilities to Claude Code.

## Configured MCP Servers

These MCP servers are **configured in this project**:

### 1. Shadcn MCP (`mcp__shadcn__*`)

**Purpose**: Access Shadcn component registry

**Available Tools:**
- `mcp__shadcn__list_shadcn_components()` - List all available components
- `mcp__shadcn__get_component_details(component)` - Get component details
- `mcp__shadcn__get_component_examples(component)` - Get usage examples
- `mcp__shadcn__search_components(query)` - Search components

**Usage:**
```
"Add the Dialog component from shadcn"
"Show me the Button component examples"
"What shadcn components are available for forms?"
```

### 2. Filesystem MCP (`mcp__filesystem__*`)

**Purpose**: Advanced file system operations

**Available Tools:**
- `mcp__filesystem__read_text_file(path)` - Read file contents
- `mcp__filesystem__write_file(path, content)` - Write to file
- `mcp__filesystem__edit_file(path, edits)` - Make line-based edits
- `mcp__filesystem__list_directory(path)` - List directory contents
- `mcp__filesystem__search_files(path, pattern)` - Search for files
- `mcp__filesystem__directory_tree(path)` - Get tree structure

**Usage:**
```
"Read the contents of package.json"
"List files in the components directory"
"Search for TypeScript files in lib/"
```

### 3. Context7 MCP (`mcp__context7__*`)

**Purpose**: Library documentation lookup

**Available Tools:**
- `mcp__context7__resolve-library-id(libraryName)` - Resolve library ID
- `mcp__context7__get-library-docs(libraryID, topic)` - Get documentation

**Usage:**
```
"Show me the Next.js documentation for routing"
"How do I use react-hook-form with TypeScript?"
"Get Drizzle ORM docs for queries"
```

## Configuration

### Project Configuration (`.mcp.json`)

```json
{
  "mcpServers": {
    "shadcn": {
      "command": "npx",
      "args": ["-y", "shadcn-ui-mcp-server"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/williamsimmons/Desktop/Projects/monorepo"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp"]
    }
  }
}
```

### Claude Code Settings (`.claude/settings.json`)

```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": [
    "shadcn",
    "filesystem",
    "context7"
  ]
}
```

## Usage Patterns

### When to Use MCP Tools vs Direct Operations

**Use MCP Tools when:**
- Claude Code needs to interact with external services
- You want AI to discover available components/features
- You need AI to perform operations on your behalf
- Working with library documentation

**Use Direct Operations when:**
- Building application features (not AI assistance)
- You need fine-grained control
- Performance is critical
- You're in production code

### Example: Using Shadcn MCP

```
User: "Add the Dialog component from shadcn"

Claude Code: [Uses mcp__shadcn__get_component_details('dialog')]
             [Installs component if needed]
             Here's how to use the Dialog component...
```

### Example: Using Context7 MCP

```
User: "How do I use Drizzle ORM with relations?"

Claude Code: [Uses mcp__context7__resolve-library-id('drizzle-orm')]
             [Uses mcp__context7__get-library-docs(id, 'relations')]
             Here's how to use relations in Drizzle ORM...
```

### Example: Using Filesystem MCP

```
User: "Show me the project structure"

Claude Code: [Uses mcp__filesystem__directory_tree('/')]
             Here's your project structure...
```

## Troubleshooting

### MCP Not Working?

1. ✅ Verify `.mcp.json` exists in project root
2. ✅ Check `.claude/settings.json` has `enabledMcpjsonServers`
3. ✅ Ensure MCP server names match in both files
4. ✅ Restart Claude Code session
5. ✅ Check MCP server logs: `claude-code mcp logs`

### Filesystem MCP Path Issues?

**Error**: "Path not allowed" or "Permission denied"

**Solution**:
1. ✅ Verify path in `.mcp.json` is absolute
2. ✅ Ensure path points to project root
3. ✅ Check file/directory permissions
4. ✅ Update path and restart Claude Code

### Shadcn MCP Not Finding Components?

**Error**: "Component not found"

**Solution**:
1. ✅ Verify component name is correct (lowercase)
2. ✅ Check shadcn-ui-mcp-server is installed
3. ✅ Try searching: `mcp__shadcn__search_components('button')`
4. ✅ Restart Claude Code session

### Context7 MCP Documentation Not Loading?

**Error**: "Library not found" or "No documentation available"

**Solution**:
1. ✅ Use `resolve-library-id` first to get correct library ID
2. ✅ Verify library name spelling
3. ✅ Check internet connection (Context7 fetches docs remotely)
4. ✅ Try with a well-known library (e.g., 'react', 'next.js')

## Best Practices

### Security

- Only configure trusted MCP servers
- Review MCP server permissions
- Don't expose sensitive API keys in MCP configs
- Use environment variables for credentials

### Performance

- MCP tools add latency - use sparingly
- Cache results when possible
- Prefer direct operations in production code

### Organization

- Keep project MCP config in `.mcp.json` (project root)
- Keep Claude Code settings in `.claude/settings.json`
- Keep personal overrides in `.claude/settings.local.json` (gitignored)
- Document custom MCP servers in this rule

## Adding New MCP Servers

### 1. Add to `.mcp.json`

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "my-mcp-server"]
    }
  }
}
```

### 2. Enable in `.claude/settings.json`

```json
{
  "enabledMcpjsonServers": [
    "shadcn",
    "filesystem",
    "context7",
    "my-server"
  ]
}
```

### 3. Restart Claude Code

Fully restart Claude Code session to load new server.

### 4. Test Server

Ask Claude Code to use the new server's tools.

## Verification

### Test Shadcn MCP

1. Ask: **"What shadcn components are available?"**
   - Should list all components

2. Ask: **"Add the Dialog component"**
   - Should provide installation steps

### Test Filesystem MCP

1. Ask: **"List files in the lib directory"**
   - Should list files

2. Ask: **"Show me the project structure"**
   - Should provide directory tree

### Test Context7 MCP

1. Ask: **"Show me Next.js routing documentation"**
   - Should fetch and display docs

2. Ask: **"How do I use react-hook-form?"**
   - Should provide usage examples

### Check MCP Status

```bash
# View MCP server logs
claude-code mcp logs

# List configured servers
claude-code mcp list
```

## References

- MCP Specification: https://modelcontextprotocol.io/
- See `.mcp.json` for server configuration
- See `.claude/settings.json` for enabled servers
- See `CLAUDE.md` > "MCP Servers" for overview

---

**Last Verified**: 2025-12-29
