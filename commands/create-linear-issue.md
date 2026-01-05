# Create Linear Issue from Context

Create a Linear issue based on the current conversation context. Typically used when:

- You've discovered something that needs a separate PR while working on another task
- You want to track follow-up work before switching contexts
- You need to create an issue for work you're about to start

## Steps

1. **Gather context**

   - Review open files and recent edits in the session
   - Check conversation history for relevant details
   - Use any additional context provided after the command
   - Identify what problem or improvement was found and why it matters

2. **Create the issue** using the Linear MCP `create_issue` tool with:

   **Title**

   - Clear, concise description of what needs to be done
   - Use action-oriented language
   - Examples: "Fix missing error handling for address monitoring", "Add validation for empty user input"

   **Description** - Keep it straightforward and actionable:

   ```markdown
   ## Problem

   [2-3 sentences max. What's broken or missing? What's the user impact?]

   ## Root Cause

   [1-2 sentences. Why is this happening?]

   ## Fix

   [What needs to be done to resolve it?]
   ```

   **Guidelines**:

   - Be direct and concise - avoid lengthy explanations
   - Focus on what's wrong and what needs to happen
   - Don't include file paths, code snippets, or investigation details
   - Don't document the solution implementation - that belongs in the PR
   - Reference parent issues if this is follow-up work

   **Priority** (infer from context):

   - 1 (Urgent): Security vulnerabilities, data loss risks, production blockers
   - 2 (High): Bugs affecting users, broken critical functionality
   - 3 (Normal): Default for most work - features, fixes, improvements
   - 4 (Low): Minor improvements, code cleanup, nice-to-haves
   - When in doubt, use 3 (Normal)

   **Labels**:

   - Add "Bug" for bug fixes
   - Add "Enhancement" for new features or improvements
   - Add relevant area labels if identifiable

   **Assignee**: me

   **Project**:

   - Add to specified project if user mentions one
   - If working on a feature ticket, try to add to the same project
   - Otherwise, default to "Backlog"

3. **Report back** using this format:
   ```
   ✅ Created **[ISSUE-ID](link)**: [Issue title]
      - Priority: [priority level]
      - Assigned to: [user]
      - Project: [project name or "None"]
   ```
