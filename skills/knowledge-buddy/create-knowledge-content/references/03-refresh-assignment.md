# Step 3: Refresh Assignment

## Purpose

Refresh the assignment after case creation to prepare the form for configuration. This ensures the UI and data structures are synchronized before proceeding.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "<from-step-1>",  // e.g., "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!CREATEFORM_DEFAULT"
  actionID: "Create"
})
```

## Expected Response

The refresh call prepares the assignment for subsequent configuration. No specific data extraction is required from the response.

## When to Refresh

Refresh is required:
- After case creation (this step)
- After submitting configuration (Step 8)
- When switching between form views or updating embedded pages

## Next Step

Proceed to [04-query-datasources.md](04-query-datasources.md) to fetch data sources for the selected collection.
