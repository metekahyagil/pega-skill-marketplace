# Step 3: Create Case

## Purpose

Create a new Knowledge Buddy case.

## MCP Tool

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-QnA-Work-KnowledgeQnA",
  content: {
    pyAddCaseContextPage: {}
  },
  processID: "pyStartCase"
})
```

## Extract from Response

- **Case ID**: `data.caseInfo.ID` (e.g., "PEGAFW-QNA-WORK BUDDY-1002")
- **Business ID**: `data.caseInfo.businessID` (e.g., "BUDDY-1002")
- **Assignment ID**: `data.caseInfo.assignments[0].ID`
- **Action ID**: `data.caseInfo.assignments[0].actions[0].ID` (should be "Create")

Case will be in "Create" stage (PRIM0).

## Response Format

See [response-structures.md](response-structures.md#create_case)

## Next Step

[04-configure-secure.md](04-configure-secure.md)
