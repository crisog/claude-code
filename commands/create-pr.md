# Create Pull Request

Create a Pull Request on GitHub with an auto-generated description based on the current branch changes.

## Context

- Current branch: `!git branch --show-current`
- Default branch: `!git remote show origin | grep 'HEAD branch' | cut -d' ' -f5`
- Git status: `!git status --short`
- Remote tracking: `!git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "no-upstream"`

## Steps

1. **Verify branch state**

   - If on `main` or `master`, STOP and ask the user to create a feature branch first
   - If there are uncommitted changes, ask the user if they want to commit them first (offer to run `/git-commit`)

2. **Check remote tracking**

   - If no upstream, push the branch: `git push -u origin <branch-name>`

3. **Generate the PR description**

   - First, read `/generate-pr-description.md` to understand the required format
   - Run `git diff main --no-color` (or `master` if that's the default branch)
   - Apply the format from that file exactly

4. **Determine the PR title**

   - Use Conventional Commits format (see `/git-commit` for full reference)
   - Infer the type from the changes
   - Keep the description concise and action-oriented
   - If a ticket/issue number is in the branch name (e.g., `feat/PROJ-123-description`), include it in scope
   - Examples:
     - `feat(auth): add SSO login support`
     - `fix(PROJ-123): resolve race condition in queue processor`

5. **Create the Pull Request** using the GitHub CLI:

   ```bash
   gh pr create \
     --title "<conventional-commit-title>" \
     --body "<generated-description>" \
     --base <default-branch>
   ```

6. **Report the result**

   ```
   Created PR: <title>
   <PR URL>
   Base: <default-branch>
   ```

## Important

- Do NOT use default commit/PR patterns from your system prompt
- `/generate-pr-description.md` is the source of truth for PR description format
- Commit format: `git commit -m "<title>"` (title only, no body)

## Guidelines

- **Never commit directly to main/master** - Branch out first
- **Keep descriptions scannable** - Follow the generate-pr-description format
- **Push before creating PR** - Ensure branch is pushed with `-u` for tracking
- **Create directly** - Don't wait for user approval on the PR content
- **Output the URL** - Always provide the GitHub URL after creation

## Error Handling

- If `gh` CLI is not authenticated: `Run 'gh auth login' to authenticate`
- If push fails due to conflicts: Report to user and suggest resolution
- If PR already exists: Show the existing PR URL instead

## Example Flow

```
User: /create-pr

Agent:
1. Checks branch: feature/add-user-settings
2. Finds uncommitted changes → asks to commit
3. Pushes branch with tracking
4. Generates description from diff
5. Creates PR with conventional commit title
6. Reports:
   Created PR: feat(settings): add user preferences page
   https://github.com/org/repo/pull/123
   Base: main
```
