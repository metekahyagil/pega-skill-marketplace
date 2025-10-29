# Step 5: Query and Select Access Roles

## Purpose

Fetch available access roles and use `AskUserQuestion` tool with multi-select to allow user to choose one or more access roles for the content.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  dataViewParameters: {}
})
```

## Extract from Response

For each access role in the response, capture:
- **AccessRoleName**: Role identifier (e.g., "KnowledgeBuddy:Public", "KnowledgeBuddy:Admin")
- **Label**: Display name of the role

## Common Access Roles

| AccessRoleName                    | Label                   | Purpose                    |
|-----------------------------------|-------------------------|----------------------------|
| KnowledgeBuddy:Public             | Knowledge buddy public  | Public access              |
| KnowledgeBuddy:Internal           | Internal                | Internal users only        |
| KnowledgeBuddy:Author             | Knowledge buddy author  | Content authors            |
| KnowledgeBuddy:BuddyManager       | Knowledge buddy manager | Knowledge base managers    |
| KnowledgeBuddy:DataSourceManager  | Data source manager     | Data source administrators |
| KnowledgeBuddy:Admin              | Buddy administrator     | Full administrative access |

## User Selection

Use `AskUserQuestion` tool with **multiSelect: true** to allow user to select one or more roles:

**If ≤4 roles**: Present all roles as options in `AskUserQuestion` tool with:
- **label**: Label or AccessRoleName
- **description**: Brief purpose of the role
- **multiSelect**: true (allows multiple selections)

**If >4 roles**: Display the full list to the user and ask them to specify the role names (comma-separated).

**Default**: If no selection is made, use `["KnowledgeBuddy:Public"]`.

## Data to Capture

Store the selected access roles as an array:
- **AccessRoles**: Array of AccessRoleName strings (e.g., `["KnowledgeBuddy:Admin", "KnowledgeBuddy:Author"]`)

This array will be used to create multiple pageInstructions entries in Step 7.

## Next Step

Proceed to [06-configure-chunking.md](06-configure-chunking.md) to configure chunking settings.
