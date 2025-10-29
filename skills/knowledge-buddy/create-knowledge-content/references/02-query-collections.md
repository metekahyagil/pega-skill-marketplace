# Step 2: Query and Select Collection

## Purpose

Fetch available collections from the system and use `AskUserQuestion` tool to allow user to select the target collection.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  dataViewParameters: {},
  paging: {
    pageNumber: 1,
    pageSize: 50
  },
  query: {
    select: [
      { field: "pyID" },
      { field: "CollectionName" }
    ],
    distinctResultsOnly: "true"
  }
})
```

## Extract from Response

For each collection in the response, capture:
- **CollectionName**: Display name of the collection
- **pyID**: Collection identifier (e.g., "DC-1")
- **pzInsKey**: Compute as `"PEGAFW-QNA-WORK <pyID>"` (e.g., "PEGAFW-QNA-WORK DC-1")

## User Selection

Use `AskUserQuestion` tool to allow user to select one collection:

**If ≤4 collections**: Present all collections as options in `AskUserQuestion` tool with:
- **label**: CollectionName
- **description**: Include pyID for reference

**If >4 collections**: Display the full list to the user and ask them to specify the collection name or ID.

## Data to Capture

Store the selected collection for subsequent steps:
- **CollectionName**
- **pyID**
- **pzInsKey**

All three values are required for the configuration step (Step 7).

## Next Step

Proceed to [03-refresh-assignment.md](03-refresh-assignment.md) to refresh the assignment.
