# App Playground

Personal monorepo for rapid app prototyping with Claude Code optimization.

---

## Table of Contents

1. [What This Is](#what-this-is)
2. [Working with Claude Code](#working-with-claude-code)
3. [Key Technical Decisions](#key-technical-decisions)
4. [Patterns That Work](#patterns-that-work)
5. [Common Gotchas](#common-gotchas)
6. [Rules and Skills](#rules-and-skills)
7. [Apps](#apps)
8. [Common Tasks](#common-tasks)
9. [Development Workflow](#development-workflow)
10. [Troubleshooting](#troubleshooting)

---

## What This Is

A Next.js 15 monorepo designed for rapid app prototyping. Each app lives in `app/apps/[slug]/` with shared infrastructure for LLM integration, database access, UI components, and theming.

**Philosophy**: Build fast, extract later. Apps share infrastructure but remain isolated enough to extract into standalone projects when needed.

---

## Working with Claude Code

### Memory Hierarchy

Claude Code loads context in this order:

1. **CLAUDE.md** (this file) - Primary project context, always loaded
2. **.claude/rules/*.md** - Contextual rules (path-triggered or always-active)
3. **User Memory** - Your conversation history and preferences

**Best Practice**: Keep CLAUDE.md concise but comprehensive. Detailed patterns go in rules. Quick references go in skills.

### MCP Servers

Three Model Context Protocol servers are configured:

1. **shadcn** - Shadcn/ui component documentation and installation
2. **filesystem** - Advanced file operations (read, write, search)
3. **context7** - Library documentation lookup (npm packages, frameworks)

These are auto-enabled via `.mcp.json` and `.claude/settings.json`.

---

## Key Technical Decisions

### Framework & Architecture

- **Next.js 15** with App Router for modern React patterns
- **Drizzle ORM** for type-safe database access with Postgres
- **Vercel Postgres** for managed database (or local with DATABASE_URL)
- **Tailwind CSS 4** with OKLCH colors for per-app theming
- **pnpm** for fast, efficient package management

### LLM Integration

- **Python Agent Service** (`services/agents/`) with OpenAI Agents SDK:
  - **Session Management** - Automatic conversation history tracking
  - **Guardrails** - Input/output validation
  - FastAPI service called from Next.js via REST API
  - Solves agent looping issues with proper state management
- **Unified LLM library** (`lib/llm/`) supporting multiple providers:
  - Anthropic (Claude models) via `@anthropic-ai/sdk`
  - Google (Gemini models) via `@google/generative-ai`
- **Model selection** via dropdown in apps (persisted to localStorage)
- **Mock responses** when API keys are missing (graceful degradation)

### Code Organization

- **Server Actions + Services pattern**:
  - Server Actions (`actions.ts`) - validation, auth, revalidation
  - Services (`_services/*.service.ts`) - business logic, portable
- **Per-app isolation** - Each app has own routes, components, services, schemas
- **Shared libraries** - `lib/` for validation, storage, LLM, agents, utilities

### Agent SDK

- **OpenAI Agents** - Two implementations:
  1. **Python Service** (`services/agents/`) - Recommended for production
     - Proper **Session management** for conversation state
     - **Input/Output Guardrails** for validation
     - TypeScript client in `lib/agents/openai/client.ts`
     - Run concurrently with Next.js: `pnpm dev`
  2. **Direct SDK** (`lib/agents/openai/`) - For advanced use cases
     - OpenAI Agents SDK with tool calling
     - Used in builder/refiner agents
     - Custom tools defined per-app in `_services/tools.ts`
- **Streaming responses** with React Suspense patterns
- Located in `lib/agents/` for shared agent utilities

---

## Patterns That Work

### Services Are Portable

```typescript
// ✅ Good - Portable service
export async function processData(input: string): Promise<Result> {
  // Pure business logic, no Next.js imports
  return { data: input.toUpperCase() }
}

// ❌ Bad - Coupled to Next.js
import { revalidatePath } from 'next/cache'
export async function processData(input: string) {
  const result = { data: input.toUpperCase() }
  revalidatePath('/app') // Don't do this in services
  return result
}
```

### Server Actions Handle Validation

```typescript
// actions.ts
'use server'
import { revalidatePath } from 'next/cache'
import { mySchema } from './schemas'
import { processData } from './_services/data.service'

export async function submitData(formData: FormData) {
  const validated = mySchema.parse(Object.fromEntries(formData))
  const result = await processData(validated.input)
  revalidatePath('/apps/my-app')
  return result
}
```

### Mock Responses for Missing Keys

```typescript
// _services/llm.service.ts
import { llm } from '@/lib/llm'

export async function generateText(prompt: string) {
  try {
    return await llm.ask(prompt, { model: 'claude-sonnet' })
  } catch (error) {
    // Return mock when API key missing
    return { text: '[Mock response - add ANTHROPIC_API_KEY]' }
  }
}
```

### Every App Has README.md

Each app README provides:
- **Purpose** - What problem it solves
- **Features** - What it can do
- **Architecture** - How it's built
- **Patterns** - Key implementation details

This helps Claude Code understand context when iterating on apps.

---

## Common Gotchas

### Always Revalidate After Mutations

```typescript
// ✅ Good
export async function updateData() {
  await db.update(...)
  revalidatePath('/apps/my-app')
}

// ❌ Bad - stale cache
export async function updateData() {
  await db.update(...)
  // Missing revalidatePath!
}
```

### Services Must Be Portable

- **No Next.js imports** in `.service.ts` files
- **No `revalidatePath`, `cookies()`, `headers()`**
- **Pure business logic only**
- This makes extraction to standalone projects easy

### Service File Naming

```bash
# ✅ Good - AI can recognize patterns
_services/data.service.ts
_services/llm.service.ts

# ❌ Bad - unclear purpose
_services/utils.ts
_services/helpers.ts
```

### Theme CSS Files Are Optional

```typescript
// Only import theme if app has custom CSS
import '@/styles/themes/my-app.css'
```

Most apps use Tailwind utilities only. Custom theme CSS is for:
- App-specific color palettes
- Unique animation needs
- Custom component styling

---

## Rules and Skills

### Rules (`.claude/rules/`)

**Rules provide contextual knowledge** to Claude Code. They auto-apply based on file paths or are always active.

#### Always Active Rules

- **core-conventions.md** - Project structure, imports, naming conventions
- **codebase-cleanliness.md** - Removing redundant files, unused code

#### Path-Triggered Rules

- **frontend-patterns.md** - Paths: `**/_components/**`, `**/page.tsx`, `**/layout.tsx`
- **backend-patterns.md** - Paths: `**/actions.ts`, `**/_services/**`
- **database-patterns.md** - Paths: `lib/db/schemas/**`
- **llm-patterns.md** - Paths: `**/_services/**`, `app/api/**`, `lib/llm/**`
- **component-patterns.md** - Paths: `components/**`, `**/_components/**`
- **mcp-setup.md** - Paths: `.claude/**`, `.mcp.json`
- **rule-maintenance.md** - Paths: `.claude/rules/**`, `.claude/skills/**`

#### Manual Invocation Rules

- **app-creation.md** - Reference when creating new apps (or use `/new-app` command)

**Note**: Rules are comprehensive (250-500 lines) with real code examples from the codebase.

### Skills (`.claude/skills/`)

**Skills provide quick implementation guides** for common tasks. They auto-discover based on context.

| Skill | Triggers | Purpose |
|-------|----------|---------|
| `add-app.md` | "new app", "create app" | App scaffolding steps |
| `app-documentation.md` | editing README.md | README template |
| `agents.md` | "agent SDK", lib/agents/ | OpenAI Agent SDK |
| `database.md` | "schema", "migration" | Drizzle patterns |
| `forms.md` | "form", "react-hook-form" | Form + validation |
| `llm-usage.md` | "LLM", lib/llm | Calling LLMs |
| `shadcn.md` | "shadcn", "UI component" | Shadcn installation |

**Note**: Skills are concise (~100-150 lines) for quick reference. Full details in rules.

### Custom Commands (`.claude/commands/`)

**Slash commands for workflow automation**:

- `/new-app [slug] [name] [category]` - Scaffold complete app structure
- `/review-pr [pr-number]` - Comprehensive PR review
- `/add-skill [skill-name]` - Create new skill from template
- `/audit-rules` - Audit all rules for accuracy

---

## Apps

Each app has full documentation in its README.md:

### studio (app/apps/studio/)
Agent Studio - build, test, and refine AI agents using OpenAI Agent SDK.

**Key Features**:
- Manage Agents: Central hub for all agents
- Build Mode: Conversational agent creation with spec view
- Test Mode: Ephemeral chat for testing agents
- Refine Mode: Improve agents with Agent Architect
- Admin: View system prompts and tool registry

**Tech**: OpenAI Agent SDK, Server Actions, streaming, Server-Sent Events

### foot-measure (app/apps/foot-measure/)
Computer vision foot measurement using OpenCV.js for accurate size recommendations.

**Key Features**:
- Camera-based foot scanning
- Aruco marker calibration
- Foot length/width measurements
- Size recommendations

**Tech**: OpenCV.js, canvas, Server Actions

**See individual app READMEs for full architecture and implementation details.**

---

## Common Tasks

### Adding a New App

**Quick**: Use `/new-app [slug] [name]` command

**Manual**: Read `.claude/skills/add-app.md` for step-by-step guide

**Steps**:
1. Create directory: `app/apps/[slug]/`
2. Add `page.tsx`, `layout.tsx`, `actions.ts`
3. Create `_services/` and `_components/` directories
4. Write `README.md` from template
5. Update `config/apps.json` (optional - for global nav)
6. Add theme CSS in `styles/themes/[slug].css` (optional)
7. Create DB schema in `lib/db/schemas/[slug].ts` (optional)

### Working with Database

```bash
# Generate migration
pnpm db:generate

# Push schema to database
pnpm db:push

# Open Drizzle Studio
pnpm db:studio
```

**Patterns**: See `.claude/rules/database-patterns.md` or `.claude/skills/database.md`

### Adding Shadcn Components

```bash
# Use MCP server (recommended)
# Ask: "Add the Button component from shadcn"

# Or manual
npx shadcn@latest add button
```

**Patterns**: See `.claude/skills/shadcn.md`

### Calling LLMs

```typescript
import { llm } from '@/lib/llm'

const response = await llm.ask('Explain quantum computing', {
  model: 'claude-sonnet',
  systemPrompt: 'You are a physics teacher.',
})
```

**Patterns**: See `.claude/skills/llm-usage.md` or `.claude/rules/llm-patterns.md`

### Creating Forms

```typescript
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { mySchema } from './schemas'

const form = useForm({
  resolver: zodResolver(mySchema),
})
```

**Patterns**: See `.claude/skills/forms.md` or `.claude/rules/frontend-patterns.md`

---

## Development Workflow

### Local Setup

```bash
# Install dependencies
pnpm install

# Set up Python agent service (first time only)
cd services/agents
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cd ../..

# Set up environment
pnpm setup
# (Interactive prompt for API keys and database)

# Start dev server (runs Next.js + Python service concurrently)
pnpm dev
```

Visit `http://localhost:3000` (Next.js) and `http://localhost:8000` (Python service)

### Environment Variables

Required in `.env.local` (Next.js):

```bash
DATABASE_URL="postgresql://..." # Vercel Postgres or local
ANTHROPIC_API_KEY="sk-ant-..." # Required for LLM features
GOOGLE_AI_API_KEY="..." # Optional for Gemini models
AGENT_SERVICE_URL="http://localhost:8000" # Python agent service (optional, defaults to localhost:8000)
```

Required in `services/agents/.env` (Python service):

```bash
OPENAI_API_KEY="sk-..." # Required for OpenAI agents
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feat/my-feature

# Make changes, commit
git add .
git commit -m "feat: add new feature"

# Push and create PR
git push -u origin feat/my-feature
gh pr create
```

### Database Workflow

```bash
# Edit schema in lib/db/schemas/[app].ts

# Generate migration
pnpm db:generate

# Push to database
pnpm db:push

# Open Drizzle Studio to inspect
pnpm db:studio
```

### Code Quality

- **TypeScript** - Strict mode enabled
- **ESLint** - Next.js recommended config
- **Prettier** - Auto-format on save (if configured)

**Note**: No tests currently - this is a prototyping monorepo

---

## Troubleshooting

### Claude Code Not Loading Rules

**Issue**: Rules in `.claude/rules/` not applying

**Solution**:
1. Check YAML frontmatter format
2. Verify `paths:` glob patterns match your files
3. Ensure rule files end in `.md`
4. Restart Claude Code session

### MCP Servers Not Working

**Issue**: `shadcn`, `filesystem`, or `context7` tools unavailable

**Solution**:
1. Check `.mcp.json` exists in project root
2. Verify `.claude/settings.json` has `enabledMcpjsonServers`
3. Restart Claude Code
4. Check MCP server logs: `claude-code mcp logs`

### Database Connection Errors

**Issue**: "Connection refused" or "Invalid DATABASE_URL"

**Solution**:
1. Verify `DATABASE_URL` in `.env.local`
2. Check Vercel Postgres is created: `vercel postgres create`
3. Pull env vars: `vercel env pull .env.local`
4. Test connection: `pnpm db:studio`

### Build Errors

**Issue**: `next build` fails with TypeScript errors

**Solution**:
1. Run `pnpm install` to ensure deps are current
2. Check `tsconfig.json` for strict mode issues
3. Fix type errors in files (often in `actions.ts` or services)
4. Ensure all imports are correct

### LLM Calls Failing

**Issue**: "API key not found" or "Model not available"

**Solution**:
1. Add `ANTHROPIC_API_KEY` to `.env.local`
2. Verify API key is valid (not expired)
3. Check model name matches available models
4. Restart dev server to load new env vars

### Theme CSS Not Loading

**Issue**: Custom app theme not applying

**Solution**:
1. Verify theme file exists: `styles/themes/[app].css`
2. Import in app layout: `import '@/styles/themes/[app].css'`
3. Check Tailwind config includes app styles
4. Restart dev server

---

## Shared Libraries

Located in `lib/`:

- **validation/** - Zod schemas for common patterns (email, phone, images, etc.)
- **storage/** - Type-safe localStorage/sessionStorage abstraction
- **llm/** - Unified LLM interface (Anthropic, Google)
- **agents/** - OpenAI Agent SDK utilities and custom tools
- **db/** - Drizzle ORM setup and schemas
- **utils.ts** - Common utilities (cn, formatters, etc.)

**Usage**: Import from `@/lib/[library]` in any app or component

---

## Quick Reference

### File Structure

```
app/
├── apps/[slug]/          # Per-app routes
│   ├── page.tsx          # Main UI
│   ├── layout.tsx        # App layout + metadata
│   ├── actions.ts        # Server Actions
│   ├── README.md         # App documentation
│   ├── _components/      # App-specific components
│   └── _services/        # App services (.service.ts)
├── api/                  # API routes (rare - prefer actions)
components/               # Shared UI components (shadcn)
lib/                      # Shared libraries
├── db/schemas/           # Drizzle schemas
├── llm/                  # LLM integration
├── agents/               # Agent SDK utilities
├── validation/           # Zod schemas
└── storage/              # Storage utilities
.claude/
├── rules/                # Contextual rules
├── skills/               # Auto-discovering skills
├── commands/             # Slash commands
└── settings.json         # Claude Code config
styles/themes/            # Per-app theme CSS (optional)
```

### Key Commands

```bash
# Development
pnpm dev          # Start dev server
pnpm build        # Build for production
pnpm lint         # Run ESLint

# Database
pnpm db:generate  # Generate migration
pnpm db:push      # Push schema
pnpm db:studio    # Open Drizzle Studio

# Setup
pnpm setup        # Interactive setup (API keys, DB)

# Testing (calibration system)
pnpm calibrate    # Run calibration tests
```

### Environment Setup

1. Clone repo
2. `pnpm install`
3. `pnpm setup` (interactive)
4. `pnpm dev`

That's it!

---

**Last Updated**: 2025-12-29
