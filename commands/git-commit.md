# Git Commit

Create clean, logical git commits from the current changes using conventional commits.

## Context

- Current status: `!git status`
- Staged and unstaged changes: `!git diff HEAD`
- Current branch: `!git branch --show-current`
- Recent commits: `!git log --oneline -10`

## Steps

1. **Analyze the changes**

   - Review the diff to understand what changed
   - Identify logical groupings (don't mix unrelated changes)
   - Check recent commits for style consistency

2. **Determine commit strategy**

   - **Single commit**: All changes are related to one concern
   - **Multiple commits**: Changes span different concerns (split them logically)
   - Stage files incrementally if splitting commits

3. **Write the commit message** using conventional commits:

   ```
   <type>(<scope>): <description>

   [optional body]

   [optional footer]
   ```

   **Types**:
   - `feat`: New feature
   - `fix`: Bug fix
   - `refactor`: Code change that neither fixes a bug nor adds a feature
   - `docs`: Documentation only
   - `test`: Adding or updating tests
   - `chore`: Maintenance tasks (deps, config, etc.)
   - `perf`: Performance improvement
   - `style`: Formatting, whitespace (no code change)

   **Scope**: Optional, indicates the area affected (e.g., `auth`, `api`, `ui`)

   **Description**:
   - Imperative mood ("add" not "added")
   - Lowercase, no period at end
   - Complete the sentence: "This commit will..."

   **Body** (when needed):
   - Explain *why*, not *what* (the diff shows what)
   - Wrap at 72 characters

4. **Create the commit(s)**

   - Stage the appropriate files
   - Commit with the prepared message
   - Repeat if multiple logical commits

## Examples

### Simple feature
```
feat(auth): add password reset flow
```

### Bug fix with context
```
fix(api): handle null response from external service

The upstream API occasionally returns null instead of an empty
array, causing the mapping function to fail silently.
```

### Breaking change
```
feat(config)!: require explicit environment in config

BREAKING CHANGE: Config no longer defaults to "development".
All environments must now specify their target explicitly.
```

### Chore
```
chore(deps): upgrade react to v19
```

## Guidelines

- Keep commits atomic: one logical change per commit
- Don't commit generated files, build artifacts, or secrets
- If changes are too intertwined to split, commit together with the dominant type
- Use `!` after type/scope for breaking changes
- Reference issues in footer when applicable: `Closes #123`
