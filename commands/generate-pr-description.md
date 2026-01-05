# Generate PR Description

Create a concise PR description that explains **what** changed and **why**, with implementation details at a high level.

## Steps

1. **Analyze the changes**

   - Run `git diff master --no-color` to see what changed
   - Identify the scope of changes and core problem being solved
   - Understand the user/developer impact

2. **Write the description** following this structure:

   ```markdown
   [1-2 sentences: Why was this change needed?]

   This PR [main change in one sentence].

   [Optional: 1-3 bullets for complex PRs with multiple distinct aspects]

   ## Breaking Change

   [If applicable: Before/after code example]
   [If none: Omit this section entirely]
   ```

3. **Guidelines**:

   - **Lead with motivation**: Why did this need to change?
   - **State the outcome**: What does this PR achieve?
   - **Stay high-level**: Focus on architecture and key decisions, not every code change
   - **Bullet points**: Use for complex PRs with multiple distinct aspects; prefer prose for simple changes
   - **If using bullets**: 3 max
   - **Skip obvious details**: No need to mention "updated tests" unless noteworthy
   - **Keep it flat**: No sections except Breaking Change

4. **FORBIDDEN patterns** (never include these):

   - Sections like `## Problem`, `## Solution`, `## Changes`, `## Testing`, `## Impact`
   - File paths or package names (`packages/server/src/...`, `next.config.mjs`)
   - Code-level details ("Changed default from X to Y", "Added `.default(0.1)`")
   - Emoji icons
   - Changelog-style lists of every file changed
   - More than 3 bullet points
   - Nested bullets

5. **Present the description** in a clean markdown block

## Examples

### Config fix

```markdown
Error tracking was disabled in production because the sample rate wasn't configured.

This PR adds default sample rates and explicit environment configuration to ensure errors are always captured.
```

### Simple refactor

```markdown
Direct database access was inconsistent with the repository pattern used elsewhere.

This PR moves user session creation to the repository layer for consistency.
```

### Feature with breaking change

```markdown
Large file uploads were failing because the entire file had to be loaded into memory.

This PR adds chunked upload support, allowing files to be uploaded in parts and resumed after interruption.

## Breaking Change

API clients must use the new multipart upload flow:

\`\`\`typescript
// Before
{ fileData, fileName }

// After
{ uploadId, chunkIndex, chunkData }
// Initialize with: POST /uploads/init { fileName, totalChunks }
\`\`\`
```

### Bug fix

```markdown
Background jobs were failing silently when the external API returned a timeout.

This PR adds proper timeout handling with user-friendly error messages and automatic retry for idempotent operations.
```
