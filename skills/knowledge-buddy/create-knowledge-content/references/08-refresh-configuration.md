# Step 8: Refresh After Configuration

## Purpose

Refresh the case view after configuration is submitted to synchronize the UI and prepare for content authoring.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__refresh_case_view({
  caseID: "<from-step-1>",  // e.g., "PEGAFW-KB-WORK-ARTICLE KB-2020"
  viewID: "pyDetailsTabContent"
})
```

**Alternative API endpoint** (as shown in Flow.md):
```
GET /prweb/app/KnowledgeBuddy/api/application/v2/cases/<caseID>/views/pyDetailsTabContent/refresh
```

## Expected Response

The refresh call ensures the case view reflects the configuration changes. No specific data extraction is required from the response.

## Why Refresh Here

After submitting configuration with pageInstructions in Step 7:
- The case transitions from Create (PRIM0) to Draft (PRIM2) stage
- Embedded pages (Collection, Datasource) are now populated
- Access roles are associated
- Chunking parameters are set (if advanced mode)

Refreshing ensures the authoring action in Step 9 has access to the updated case data.

## Next Step

Proceed to [09-author-content.md](09-author-content.md) to author the article content.
