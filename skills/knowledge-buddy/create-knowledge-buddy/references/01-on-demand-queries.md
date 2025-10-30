# On-Demand Data Queries

## Overview

Query Pega data views only when needed based on user selections. This approach minimizes unnecessary API calls and token usage.

## Query Patterns

### When User Selects "Clone"

**Query**: D_OriginalBuddyList

**When**: After user answers "Yes" to cloning question

**Purpose**: Present list of existing buddies to clone from

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_OriginalBuddyList",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" },
      { field: "pyDescription" }
    ],
    distinctResultsOnly: "true"
  },
  paging: { pageNumber: 1, pageSize: 50 }
})
```

**Extract**: Name, pyID, pyDescription for user selection

---

### When User Selects "Custom Access"

**Query**: D_BuddyAccessRoleList

**When**: After user answers "Custom" to access configuration question

**Purpose**: Present list of available roles for assignment

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  query: {
    select: [
      { field: "pyAccessRole" },
      { field: "pyLabel" }
    ]
  },
  dataViewParameters: { CaseId: "<CASE-ID>" },
  paging: { pageSize: 20 }
})
```

**Extract**: pyAccessRole (role name like "KnowledgeBuddy:Admin"), pyLabel (display name)

**Note**: CaseId parameter can be the current buddy case ID

---

### When User Customizes Context

**Query 1**: D_IndexList (Collections)

**When**: User selects "Edit default" or "Add custom contexts" for context data

**Purpose**: Present list of available collections

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: {
    select: [
      { field: "pyID" },
      { field: "CollectionName" }
    ],
    distinctResultsOnly: "true"
  },
  paging: { pageNumber: 1, pageSize: 50 }
})
```

**Extract**: pyID (e.g., "DC-1"), CollectionName

---

**Query 2**: D_DataSourceListByCollection (Data Sources)

**When**: After user selects a collection

**Purpose**: Present data sources available in selected collection

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_DataSourceListByCollection",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" }
    ],
    distinctResultsOnly: "true"
  },
  dataViewParameters: {
    pyID: "DC-1",  // Parameter name is "pyID", value is the selected collection's pyID (e.g., "DC-1")
    Available: "Yes"
  },
  paging: { maxResultsToFetch: "100" }
})
```

**Extract**: Name (display name), pyID (e.g., "SRC-2001")

**Important**: The `pyID` parameter must use the collection's pyID from Query 1, not "CollectionID".

---

**Query 3**: D_AttributeList (Response Attributes)

**When**: After user selects a collection

**Purpose**: Present available response attributes for selected collection

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_AttributeList",
  dataViewParameters: {
    indexName: "<COLLECTION-NAME>"  // e.g., "knowledge"
  }
})
```

**Extract**: Attribute names (e.g., "contentID", "contentKey", "contentSourceSystem")

**Note**: Use CollectionName (not pyID) for indexName parameter

---

### When User Selects "Specific GenAI Model"

**Query**: D_bxGenAIModelsForBuddy

**When**: User answers "Select specific model" for GenAI configuration

**Purpose**: Present list of available GenAI models

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
  dataViewParameters: { ModelId: "" }
})
```

**Extract**: pyID (model identifier), pyLabel (display name), ModelType (e.g., "GPT")

**Note**: Check ModelType to determine if OutputFormat question is needed (GPT models require format selection)

---

## Best Practices

1. **Query only when needed**: Don't query data unless user selects path requiring it
2. **Query in sequence**: For context configuration, query collections first, then data sources/attributes after collection is selected
3. **Store results**: Keep query results in memory for the current workflow session
4. **Handle empty results**: Inform user if no options available (e.g., no collections, no buddies to clone)

## Response Handling

All queries return similar structure:

```json
{
  "data": {
    "results": [
      { "field1": "value1", "field2": "value2" },
      { "field1": "value3", "field2": "value4" }
    ]
  }
}
```

Extract results array and present to user via AskUserQuestion options.

## Query Sequence Example

For a user choosing to customize context:

1. User selects "Edit default context" → Query D_IndexList
2. User selects collection "DC-1" → Query D_DataSourceListByCollection with pyID="DC-1"
3. User selects collection "DC-1" → Query D_AttributeList with indexName="knowledge"
4. Present data sources and attributes for user multi-select

## Next References

- [02a-initial-input-clone.md](02a-initial-input-clone.md) - Using D_OriginalBuddyList query results for cloning
- [04b-configure-access-custom.md](04b-configure-access-custom.md) - Using D_BuddyAccessRoleList for role queries
- [06b-configure-context-edit-default.md](06b-configure-context-edit-default.md) - Using collection/data source queries for editing context
- [06c-configure-context-add-custom.md](06c-configure-context-add-custom.md) - Using collection/data source queries for custom contexts
- [07b-configure-genai-custom.md](07b-configure-genai-custom.md) - Using D_bxGenAIModelsForBuddy for model selection
