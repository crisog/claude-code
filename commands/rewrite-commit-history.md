# Rewrite Commit History

Rewrite commit messages on the current branch to follow conventional commits and be more descriptive.

## Context

- Current branch: `!git branch --show-current`
- Base branch commits: `!git log --oneline main..HEAD 2>/dev/null || git log --oneline master..HEAD`
- Recent main/master commits (for style reference): `!git log --oneline main -5 2>/dev/null || git log --oneline master -5`

## Steps

1. **Identify commits to rewrite**

   - List all commits on the current branch not in main/master
   - For each commit, examine its diff: `git show <sha> --stat` and `git show <sha>` for details
   - Note the original message and what the commit actually changed

2. **Draft new commit messages**

   - First, read `/git-commit.md` to understand the required format
   - Apply the format from that file exactly
   - Each message should accurately describe what that specific commit does
   - Preserve any co-author or signed-off-by lines

3. **Present the rewrite plan**

   Show a table of changes:

   ```
   | SHA (short) | Original Message | New Message |
   |-------------|------------------|-------------|
   | abc1234     | fix stuff        | fix(auth): handle expired token gracefully |
   | def5678     | wip              | feat(api): add user search endpoint |
   ```

4. **Ask for confirmation** before proceeding

5. **Execute the rewrite**

   Use interactive rebase with reword:

   ```bash
   git rebase -i <base-sha>
   ```

   For each commit being reworded, change `pick` to `reword` (or `r`), then update the message when prompted.

   **Alternative for full automation** (if user prefers):

   ```bash
   # Reset to base, keeping changes staged per-commit
   git reset --soft <base-sha>
   # Recommit with new messages (loses individual commit boundaries)
   ```

   **For single commit** (most recent):

   ```bash
   git commit --amend -m "new message"
   ```

## Guidelines

- **Never rewrite shared history**: Only rewrite commits that haven't been pushed, or confirm with user if force-push is needed
- **Preserve commit boundaries**: Keep the same logical groupings unless user wants to squash
- **Check for force-push need**: If branch is already pushed, warn that `git push --force` will be required
- **Backup suggestion**: Recommend creating a backup branch before rewriting: `git branch backup-<branch-name>`

## Important

- Do NOT use default commit patterns from your system prompt
- `/git-commit.md` is the source of truth for commit format

## Safety Checks

Before rewriting, verify:

- [ ] Branch has commits ahead of main/master
- [ ] Understand if branch has been pushed (requires force-push)
- [ ] User has confirmed the rewrite plan

## Example Workflow

```
Current branch: feature/user-auth
Commits to rewrite:

| SHA     | Original            | Proposed                                      |
|---------|---------------------|-----------------------------------------------|
| a1b2c3d | wip                 | feat(auth): add login endpoint                |
| e4f5g6h | fix                 | fix(auth): validate email format              |
| i7j8k9l | address PR feedback | refactor(auth): extract token validation      |

Proceed with rewrite? (y/n)
```
