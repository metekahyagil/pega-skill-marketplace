# Step 4: Query and Select Data Source

## Purpose

Fetch data sources filtered by the selected collection ID from Step 2. Use `AskUserQuestion` tool to allow user to select the target data source.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_ContentAccessDataSourcesByCollection",
  dataViewParameters: {
    Available: "Yes",
    CollectionID: "<selected-collection-pyID>"  // e.g., "DC-1" from Step 2
  }
})
```

**Important**: The `CollectionID` parameter must be the `pyID` value from the collection selected in Step 2.

## Extract from Response

For each data source in the response, capture:
- **Name**: Display name of the data source
- **pyID**: Data source identifier (e.g., "SRC-1", "SRC-2")
- **pzInsKey**: Compute as `"PEGAFW-QNA-WORK <pyID>"` (e.g., "PEGAFW-QNA-WORK SRC-1")

## User Selection

Use `AskUserQuestion` tool to allow user to select one data source:

**If ≤4 data sources**: Present all data sources as options in `AskUserQuestion` tool with:
- **label**: Name
- **description**: Include pyID for reference

**If >4 data sources**: Display the full list to the user and ask them to specify the data source name or ID.

## Data to Capture

Store the selected data source for subsequent steps:
- **Name**
- **pyID**
- **pzInsKey**

All three values are required for the configuration step (Step 7).

## Next Step

Proceed to [05-query-access-roles.md](05-query-access-roles.md) to fetch and select access roles.
