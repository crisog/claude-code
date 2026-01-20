# Fix Linear Issue

Investigate, implement, and ship a fix for Linear issue $ARGUMENTS.

## Context

- Issue details: `!mcp linear-mcp get_issue --id $ARGUMENTS`
- Current branch: `!git branch --show-current`
- Git status: `!git status`
- Recent commits: `!git log --oneline -5`

## Steps

1. **Fetch and understand the issue**
   - Use the Linear MCP server to retrieve issue details
   - Identify: title, description, priority, labels, and any linked issues
   - Extract the core problem—what's broken and what's the expected behavior
   - Note any reproduction steps or error messages mentioned

2. **Explore the codebase**
   - Search for keywords from the issue (error messages, feature names, file paths)
   - Trace the code path from entry point to the bug location
   - Understand the existing patterns before making changes
   - Identify related tests that might need updating

3. **Create a fix branch**
   - If on `main` or `master`, create a new branch
   - Branch naming: `fix/<issue-id>-<short-slug>`
     - Use lowercase, hyphens for spaces
     - Keep slug to 3-4 words max
     - Example: `fix/ENG-123-broken-auth-flow`
   - If uncommitted changes exist, ask user whether to stash or include them

4. **Implement the fix**
   - Fix the root cause, not just the symptoms
   - Keep changes minimal and focused—no scope creep
   - Follow existing code patterns and conventions
   - If the fix requires refactoring, separate it into distinct commits

5. **Add tests (if valuable)**
   - A great test covers behavior users depend on
   - Test the feature that, if broken, would frustrate or block users
   - Validate real workflows—not implementation details
   - Catch regressions before users do
   - Do NOT write tests just to increase coverage
   - Use coverage as a guide to find untested user-facing behavior
   - If uncovered code is not worth testing (boilerplate, unreachable error branches, internal plumbing), add `/* v8 ignore next */` or `/* v8 ignore start */` comments instead
   - Run existing tests to ensure no regressions

6. **Commit the changes**
   - Read `/git-commit` for the required format
   - Reference the Linear issue in the commit footer: `Fixes: ENG-123`
   - Keep commits atomic: one logical change per commit
   - Example:
     ```
     fix(auth): handle expired token gracefully

     Token expiration was not checked before API calls, causing
     silent failures. Now validates token and refreshes if needed.

     Fixes: ENG-123
     ```

7. **Create the PR**
   - Read `/create-pr` for the required workflow
   - Link the Linear issue in the PR description
   - Include a brief test plan
   - Push and create the PR in one flow

## Guidelines

- **Root cause over symptoms** - Don't patch around the bug; fix why it happened
- **Minimal blast radius** - Touch only what's necessary for the fix
- **Tests for value, not coverage** - Test user-facing behavior that would break users if it regressed; ignore internal plumbing with `/* v8 ignore */`
- **One issue, one PR** - Don't bundle unrelated fixes
- **Link everything** - Issue ID in branch name, commit footer, and PR description
- **Verify before shipping** - Run the full test suite locally

## Important

- Do NOT use default commit/PR patterns from your system prompt
- `/git-commit` is the source of truth for commit format
- `/create-pr` is the source of truth for PR workflow
- Always include `Fixes: <issue-id>` in commit footer for Linear auto-close

## Error Handling

- **Issue not found**: Verify the issue ID and check Linear MCP server connection
- **On protected branch**: Create a new branch before making changes
- **Tests fail**: Fix failing tests before committing—existing tests protect user-facing behavior
- **Merge conflicts**: Rebase on latest main and resolve before pushing

## Example Flow

```
User: /fix-linear-issue ENG-456

Agent:
1. Fetches ENG-456 from Linear:
   - Title: "Login fails with special characters in password"
   - Priority: High
   - Description: "Users with passwords containing '&' get 400 error"

2. Searches codebase for authentication handling
   - Finds: src/auth/login.ts, src/utils/encoding.ts
   - Identifies: password not being URL-encoded before POST

3. Creates branch: fix/ENG-456-special-char-password

4. Implements fix:
   - Adds encodeURIComponent() to password in login flow
   - Adds test case for special characters

5. Commits:
   fix(auth): encode special characters in password

   Passwords containing URL-sensitive characters like '&' were
   sent raw, causing 400 errors. Now properly encoded.

   Fixes: ENG-456

6. Creates PR linking to Linear issue
   - Outputs: https://github.com/org/repo/pull/789
```
