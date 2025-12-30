# Will's Claude Code Config

Shareable Claude Code rules, skills, and commands.

## What's Here

- **CLAUDE.md** - Primary context file (project-specific example)
- **.claude/rules/** - Contextual rules with YAML frontmatter
- **.claude/skills/** - Auto-discovering implementation guides
- **.claude/commands/** - Slash commands for workflow automation
- **.claude/learnings/** - Captured learnings

## Usage

Copy the files you want to your project:

```bash
# Copy everything
cp -r .claude/ /path/to/your/project/

# Or just rules
cp -r .claude/rules/ /path/to/your/project/.claude/
```

## Key Rules

| Rule | Purpose |
|------|---------|
| `core-conventions.md` | Project structure, imports, naming |
| `frontend-patterns.md` | React, Shadcn, forms, theming |
| `backend-patterns.md` | Server Actions, services, validation |
| `database-patterns.md` | Drizzle ORM patterns |
| `llm-patterns.md` | LLM integration patterns |
| `component-patterns.md` | Component organization |
| `documentation-patterns.md` | Keeping docs current |
| `codebase-cleanliness.md` | Cleanup guidelines |
| `rule-maintenance.md` | Keeping rules updated |

## Key Skills

| Skill | Purpose |
|-------|---------|
| `add-app.md` | App scaffolding |
| `forms.md` | Form + validation patterns |
| `database.md` | Drizzle queries |
| `shadcn.md` | UI component usage |
| `llm-usage.md` | Calling LLMs |
| `agents.md` | OpenAI Agent SDK |

## Commands

- `/new-app` - Scaffold new app
- `/review-pr` - Code review
- `/audit-rules` - Verify rules match codebase
- `/add-skill` - Create new skill
