---
description: Manage Jira tickets - create, update, comment, search, and transition issues
---

# Jira - Ticket Management

You are tasked with managing Jira tickets, including creating issues, updating existing tickets, searching with JQL, and transitioning issue statuses.

## Initial Setup

First, verify that Jira MCP tools are available by checking if any `mcp__atlassian__` tools exist. If not, respond:
```
I need access to Jira tools to help with ticket management. Please run the `/mcp` command to enable the Atlassian MCP server, then try again.
```

If tools are available, you need to get the Cloud ID first:
1. Use `mcp__atlassian__getAccessibleAtlassianResources` to get available Atlassian sites
2. Store the cloudId for subsequent API calls
3. If multiple sites are available, ask the user which one to use

Then respond based on the user's request:

### For general requests:
```
I can help you with Jira tickets. What would you like to do?
1. Create a new issue
2. Search for issues (using JQL or natural language)
3. Update an existing issue
4. Add a comment to an issue
5. Transition an issue to a different status
6. Log work on an issue
```

### For specific create requests:
```
I'll help you create a Jira issue. Please provide:
1. The project key (e.g., PROJ, DEV, SUPPORT)
2. Issue type (Task, Bug, Story, Epic, etc.)
3. Summary (title) of the issue
4. Description (optional but recommended)
```

Then wait for the user's input.

## Core Concepts

### Cloud ID
Every Jira API call requires a `cloudId`. Always fetch this first using `mcp__atlassian__getAccessibleAtlassianResources` and cache it for the session.

### Issue Keys
Jira issues are identified by keys like `PROJ-123`. These combine the project key and issue number.

### JQL (Jira Query Language)
JQL is the powerful query language for searching Jira. Examples:
- `project = PROJ` - All issues in project PROJ
- `assignee = currentUser()` - Issues assigned to you
- `status = "In Progress"` - Issues in progress
- `created >= -7d` - Issues created in the last 7 days
- `summary ~ "bug"` - Issues with "bug" in summary
- `labels = urgent AND priority = High` - Urgent high-priority issues

## Action-Specific Instructions

### 1. Creating Issues

#### Steps to follow after receiving the request:

1. **Get Cloud ID (if not already cached):**
   ```
   mcp__atlassian__getAccessibleAtlassianResources
   ```

2. **Get available projects:**
   ```
   mcp__atlassian__getVisibleJiraProjects with:
   - cloudId: [cached cloudId]
   ```
   Show projects to user if they haven't specified one.

3. **Get issue types for the project:**
   ```
   mcp__atlassian__getJiraProjectIssueTypesMetadata with:
   - cloudId: [cached cloudId]
   - projectIdOrKey: [project key]
   ```
   Show available issue types if user hasn't specified.

4. **Get field metadata (if needed):**
   For custom fields or required fields:
   ```
   mcp__atlassian__getJiraIssueTypeMetaWithFields with:
   - cloudId: [cached cloudId]
   - projectIdOrKey: [project key]
   - issueTypeId: [issue type ID]
   ```

5. **Create the issue:**
   ```
   mcp__atlassian__createJiraIssue with:
   - cloudId: [cached cloudId]
   - projectKey: [project key]
   - issueTypeName: [Task, Bug, Story, etc.]
   - summary: [issue title]
   - description: [markdown description]
   - assignee_account_id: [optional, user's account ID]
   - parent: [optional, for subtasks]
   - additional_fields: {optional custom fields}
   ```

6. **Post-creation:**
   - Show the created issue key and URL
   - Ask if user wants to:
     - Add more details or comments
     - Assign the issue
     - Set priority or labels

### 2. Searching for Issues

When user wants to find issues:

1. **Determine search method:**
   - If user provides JQL, use it directly
   - For natural language, use `mcp__atlassian__search` (Rovo Search)
   - For complex queries, help construct JQL

2. **Using JQL search:**
   ```
   mcp__atlassian__searchJiraIssuesUsingJql with:
   - cloudId: [cached cloudId]
   - jql: [JQL query]
   - fields: ["summary", "description", "status", "issuetype", "priority", "created", "assignee"]
   - maxResults: 20
   ```

3. **Using Rovo Search (natural language):**
   ```
   mcp__atlassian__search with:
   - query: [search text]
   ```

4. **Present results:**
   - Show issue key, summary, status, assignee
   - Include priority and type
   - Group by project if multiple projects

### Common JQL patterns:
```
# My open issues
assignee = currentUser() AND resolution = Unresolved

# Recently updated in a project
project = PROJ AND updated >= -7d ORDER BY updated DESC

# Bugs by priority
project = PROJ AND issuetype = Bug ORDER BY priority ASC

# Sprint issues
project = PROJ AND sprint in openSprints()

# Unassigned issues
project = PROJ AND assignee IS EMPTY

# Text search
text ~ "search term"
```

### 3. Updating Issues

When user wants to update an issue:

1. **Get current issue details:**
   ```
   mcp__atlassian__getJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   ```
   Show current state to user.

2. **Update the issue:**
   ```
   mcp__atlassian__editJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   - fields: {
       "summary": "New title",
       "description": "New description",
       "priority": {"name": "High"},
       "labels": ["label1", "label2"],
       "assignee": {"accountId": "user-account-id"}
     }
   ```

3. **Confirm the update:**
   - Fetch the issue again to confirm changes
   - Show updated fields to user

### 4. Adding Comments

When user wants to comment on an issue:

1. **Get issue context (if needed):**
   ```
   mcp__atlassian__getJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   ```

2. **Add the comment:**
   ```
   mcp__atlassian__addCommentToJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   - commentBody: [markdown comment]
   - commentVisibility: {optional, for restricted comments}
   ```

3. **Comment formatting:**
   - Use markdown for formatting
   - Include code blocks with syntax highlighting
   - Reference other issues with [PROJ-123]
   - Mention users with account IDs

### 5. Transitioning Issues

When user wants to change issue status:

1. **Get available transitions:**
   ```
   mcp__atlassian__getTransitionsForJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   ```
   Show available transitions to user.

2. **Execute the transition:**
   ```
   mcp__atlassian__transitionJiraIssue with:
   - cloudId: [cached cloudId]
   - issueIdOrKey: [PROJ-123]
   - transition: {"id": "transition-id"}
   - fields: {optional fields required by transition}
   ```

3. **Verify the transition:**
   - Fetch issue to confirm new status
   - Note any workflow restrictions or required fields

### 6. Logging Work

When user wants to log time:

```
mcp__atlassian__addWorklogToJiraIssue with:
- cloudId: [cached cloudId]
- issueIdOrKey: [PROJ-123]
- timeSpent: "2h" or "30m" or "4d"
- visibility: {optional, for restricted worklogs}
```

### 7. Looking Up Users

To find user account IDs for assignments:

```
mcp__atlassian__lookupJiraAccountId with:
- cloudId: [cached cloudId]
- searchString: [user name or email]
```

### 8. Getting Remote Links

To see external links attached to an issue:

```
mcp__atlassian__getJiraIssueRemoteIssueLinks with:
- cloudId: [cached cloudId]
- issueIdOrKey: [PROJ-123]
```

## Field Reference

### Standard Fields
- `summary` - Issue title (required)
- `description` - Detailed description (markdown)
- `issuetype` - Task, Bug, Story, Epic, Sub-task
- `priority` - Highest, High, Medium, Low, Lowest
- `assignee` - User account ID
- `reporter` - User account ID
- `labels` - Array of label strings
- `components` - Array of component objects
- `fixVersions` - Array of version objects
- `duedate` - Date string (YYYY-MM-DD)

### Description Formatting (ADF vs Markdown)
Jira accepts markdown in descriptions. For complex formatting:
- Use `#` for headings
- Use `*` or `-` for bullets
- Use triple backticks for code blocks
- Use `[text](url)` for links

## Important Notes

- Always get the cloudId first before making any other API calls
- Issue keys are case-insensitive but typically uppercase (PROJ-123)
- JQL is case-insensitive for keywords but case-sensitive for values
- Transitions are workflow-specific - available transitions depend on current status
- Some fields may be required depending on project configuration
- Custom fields vary by project and may need field metadata lookup
- Use `mcp__atlassian__search` for natural language queries, JQL for precise queries
- The `fetch` tool can retrieve detailed info using ARIs from search results

## Error Handling

Common errors and solutions:
- **"Field 'X' cannot be set"** - Field may be read-only or not on the screen
- **"Issue does not exist"** - Check issue key spelling and project access
- **"Transition is not valid"** - Issue isn't in a state that allows that transition
- **"User not found"** - Use `lookupJiraAccountId` to find correct account ID

## Example Workflows

### Create a bug report:
1. Get cloudId
2. Create issue with type "Bug"
3. Add reproduction steps in description
4. Set priority based on severity
5. Assign to appropriate developer

### Triage incoming issues:
1. Search for issues in "Open" or "To Do" status
2. Review each issue details
3. Add comments for clarification
4. Transition to appropriate status
5. Assign to team members

### Sprint planning:
1. Search for backlog items
2. Review estimates and priorities
3. Move selected items to sprint
4. Assign to team members
