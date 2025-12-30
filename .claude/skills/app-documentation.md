# Skill: App Documentation
<!-- Triggers: app README, document app, README.md, app documentation -->
<!-- Last verified: 2025-12 -->

## Quick Reference

- **Every app needs README.md** at `app/apps/[slug]/README.md`
- Provides instant context for Claude Code when iterating
- Update "Current Work" section when starting new features
- Update "Known Issues" section when finding problems
- Move completed work from "Current Work" to "Features"

## Required Sections

```markdown
# [App Name]

## What This Is
One-paragraph description of what the app does and why it exists.

## Features
- Feature 1
- Feature 2
- Feature 3

## Goals
- Short-term goal
- Long-term vision

## Current Work / Iteration Areas ⭐ IMPORTANT
What are you actively working on? What's being iterated?
- Feature being built
- Area being refactored
- Experiment in progress

## Known Issues / Needs Work ⭐ IMPORTANT
What needs attention? What's broken or incomplete?
- Bug that needs fixing
- Performance issue
- Missing feature
- Technical debt

## Tech Stack
- Key technologies specific to this app
- Any special libraries or patterns

## Architecture
Brief overview of how the app is structured (services, components, etc.)

## Usage Notes
- How to use the app
- Any gotchas or important details
```

## When to Create

**Always** create README.md when you:
1. Create a new app
2. Make significant changes to an existing app
3. Notice the README is outdated

## When to Update

Update the README when:
- Adding/removing features
- Changing architecture
- Discovering important usage notes
- Goals evolve
- **Starting new work** - Add to "Current Work" section
- **Finishing work** - Move from "Current Work" to "Features"
- **Finding issues** - Add to "Known Issues" section
- **Fixing issues** - Remove from "Known Issues"

## Template: Complete Example

```markdown
# Agent Chat

## What This Is
Chat interface with AI agents that can use tools. Uses Anthropic Claude Agent SDK.

## Features
- Multi-turn conversation with AI
- Session persistence and management
- Agent tool integration (web search, file operations, etc.)
- Custom agent configurations with tool selection

## Goals
- Provide a sandbox for custom tool development
- Eventually: multi-agent workflows and handoffs

## Current Work / Iteration Areas
- Adding streaming support for agent responses
- Implementing tool result caching

## Known Issues / Needs Work
- Session list doesn't update after creating new session
- Long responses cause layout shift

## Tech Stack
- `@anthropic-ai/claude-agent-sdk` - Claude Agents
- Agent tools in `_tools/`
- Database for session storage

## Architecture
- `_components/` - Chat UI (agent-selector, chat-container, chat-sidebar)
- `_services/` - No services yet (agents handle logic)
- `_tools/` - Custom agent tools
- API routes in `/app/api/chat/`

## Usage Notes
- Requires ANTHROPIC_API_KEY
- Sessions stored in database
- Tools are registered in `lib/agents/tool-registry.ts`
```

## Benefits

✅ **For AI**: Instant context without searching through code
✅ **For Developers**: Quick onboarding to any app
✅ **For You**: Remember why you built something 6 months later
✅ **For Iteration**: Claude Code can reference goals and suggest improvements
✅ **For Focus**: "Current Work" section helps AI prioritize relevant suggestions
✅ **For Tracking**: "Known Issues" prevents forgetting about problems

## References

- See `.claude/rules/app-creation.md` for README creation in new apps
- See existing app READMEs in `app/apps/*/README.md` for examples
