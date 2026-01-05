# Claude Instructions

## Structure

```
commands/   # Custom slash commands (*.md files)
skills/     # Custom skills (SKILL.md in each folder)
agents/     # Custom agents (not yet)
```

## Slash Commands

Commands are Markdown files that define reusable prompts. Each command should:

- Have a clear `# Title` describing its purpose
- Include a `## Context` section with `!command` syntax for gathering runtime info
- Have a `## Steps` section with numbered, actionable instructions
- Include `## Guidelines` for important rules
- Provide `## Examples` when helpful

### Conventional Commits

`/git-commit` is the single source of truth for conventional commits format. Other commands that deal with commit messages or PR titles should reference it.

## Writing Commands

When creating or modifying commands:

1. Keep instructions clear and actionable
2. Use `!command` syntax in Context sections for runtime data
3. Reference other commands with `/command-name` instead of duplicating content
4. Include error handling guidance where applicable
