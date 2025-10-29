# Step 1: Create Content Case

## Purpose

Create a new Content case to initiate the knowledge base article workflow. This is the first step after verifying Pega connection.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {
    pyAddCaseContextPage: {}
  },
  processID: "pyStartCase"
})
```

## Expected Response

Extract the following from the response:
- **caseID**: The case identifier (e.g., "KB-2020")
- **assignmentID**: First assignment ID from `assignments[0].ID`
- **Assignment action name**: "Create" (CREATEFORM_DEFAULT)

Example response:
```javascript
{
  caseID: "KB-2020",
  assignments: [{
    ID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!CREATEFORM_DEFAULT",
    name: "Create"
  }]
}
```

## Data to Capture

Store for subsequent steps:
- `caseID` - Required for all subsequent API calls
- `assignmentID` - Required for configuration actions

## Next Step

Proceed to [02-query-collections.md](02-query-collections.md) to fetch and select a collection.
