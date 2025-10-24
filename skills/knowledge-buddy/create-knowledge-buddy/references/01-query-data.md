# Step 1: Query Available Data

## Purpose

Query Pega data pages to retrieve collections, GenAI models, access roles, and existing buddies for user selection.

## Data Pages to Query

1. **D_IndexList** - Collections
2. **D_bxGenAIModelsForBuddy** - GenAI models
3. **D_BuddyAccessRoleList** - Access roles
4. **D_OriginalBuddyList** - Existing buddies (for cloning)

## MCP Tool

Use `mcp__pega-dx-mcp__get_list_data_view` for all queries. Query all in parallel for better performance.

### Query 1: Collections

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: {
    select: [
      { field: "CollectionName" },
      { field: "pyID" },
      { field: "pzInsKey" }
    ]
  },
  paging: { pageSize: 100 }
})
```

**Extract**: CollectionName, pyID, pzInsKey

### Query 2: GenAI Models

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_bxGenAIModelsForBuddy",
  query: {
    select: [
      { field: "pyID" },
      { field: "pyLabel" },
      { field: "ModelType" }
    ]
  },
  dataViewParameters: { ModelId: "" },
  paging: { pageSize: 50 }
})
```

**Extract**: pyID, pyLabel, ModelType

### Query 3: Access Roles

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  query: {
    select: [
      { field: "pyAccessRole" },
      { field: "pyLabel" }
    ]
  },
  dataViewParameters: { CaseId: "" },
  paging: { pageSize: 20 }
})
```

**Extract**: pyAccessRole, pyLabel

### Query 4: Existing Buddies

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_OriginalBuddyList",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" },
      { field: "pyDescription" }
    ]
  },
  paging: { pageSize: 50 }
})
```

**Extract**: Name, pyID, pyDescription

## Response Formats

See [response-structures.md](response-structures.md) for complete response formats.

## Usage

Store queried data for use in Step 2 (user input) and Steps 4-7 (configuration).

## Next Step

[02-user-input.md](02-user-input.md)
