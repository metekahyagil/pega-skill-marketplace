# Configure Context Data - Edit Default SEARCHRESULTS

## Overview

Customize the default SEARCHRESULTS context with collection, data sources, and response attributes.

---

## Query Flow

Queries are executed on-demand as the user progresses through configuration:
1. Query D_IndexList → Present collections
2. User selects collection → Query D_DataSourceListByCollection and D_AttributeList
3. Present data sources and attributes for user selection

**CRITICAL**: When querying D_DataSourceListByCollection, the parameter name MUST be `pyID`, NOT `CollectionID`. See inline query examples below.

---

## Step 1: Select Collection

**Atomic question**: "Select collection"

```javascript
AskUserQuestion({
  questions: [{
    question: "Select collection for SEARCHRESULTS context:",
    header: "Collection",
    multiSelect: false,
    options: [
      { label: "Knowledge Base", description: "ID: DC-1" },
      { label: "Documentation", description: "ID: DC-2" }
      // ... populated from D_IndexList query
    ]
  }]
})
```

**Store**: Selected collection's `pyID` (e.g., "DC-1")

---

## Query Data Sources and Attributes

**After collection is selected**, query data sources and attributes:

**Query Data Sources**:
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
    pyID: "DC-1",  // CRITICAL: Parameter name is "pyID", NOT "CollectionID"
    Available: "Yes"
  },
  paging: { maxResultsToFetch: "100" }
})
```

**Query Attributes**:
```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_AttributeList",
  dataViewParameters: {
    indexName: "knowledge"  // Use CollectionName (not pyID) here
  }
})
```

---

## Step 2: Select Data Sources

**Atomic question**: "How do you want to select data sources?"

```javascript
AskUserQuestion({
  questions: [{
    question: "How do you want to select data sources for SEARCHRESULTS context?",
    header: "Data sources",
    multiSelect: false,
    options: [
      { label: "Select all", description: "Use all available data sources" },
      { label: "Select none", description: "No filtering - defaults to all sources" },
      { label: "Enter specific data sources", description: "Specify which data sources to include" }
    ]
  }]
})
```

**If "Select all"**: Use all data source IDs from D_DataSourceListByCollection query

**If "Select none"**: Empty array (no filtering, defaults to all)

**If "Enter specific data sources"**: User will use "Type something" to provide comma-separated data source IDs (e.g., "SRC-2001,SRC-2,SRC-3")

**Store**: Array of data source IDs (e.g., `["SRC-2001", "SRC-2"]` or all IDs or `[]`)

---

## Step 3: Select Response Attributes

**Atomic question**: "How do you want to select response attributes?"

```javascript
AskUserQuestion({
  questions: [{
    question: "How do you want to select response attributes?",
    header: "Attributes",
    multiSelect: false,
    options: [
      { label: "Select all", description: "Use all available attributes" },
      { label: "Select none", description: "No filtering - defaults to all attributes" },
      { label: "Enter specific attributes", description: "Specify which attributes to include" }
    ]
  }]
})
```

**If "Select all"**: Use all attribute names from D_AttributeList query

**If "Select none"**: Empty array (no filtering, defaults to all)

**If "Enter specific attributes"**: User will use "Type something" to provide comma-separated attribute names (e.g., "contentID,contentKey,contentSourceSystem")

**Store**: Array of attribute names (e.g., `["contentID", "contentKey", "contentSourceSystem"]` or all names or `[]`)

---

## Step 4: Advanced Settings (Optional)

**Atomic question**: "Do you want advanced settings?"

```javascript
AskUserQuestion({
  questions: [{
    question: "Do you want to configure advanced settings?",
    header: "Advanced",
    multiSelect: false,
    options: [
      { label: "No", description: "Use defaults (MaxChunk: 5, MaxChunkTotalSize: 5000, MinSimilarityScore: 80)" },
      { label: "Yes", description: "Customize search parameters" }
    ]
  }]
})
```

**If "Yes"**, ask atomic questions:

1. "Search must return results?"
   - Options: "Yes" (true) / "No" (false)
   - Default: true

2. "Maximum number of chunks?"
   - User enters number (default: 5)

3. "Maximum total size of chunks?"
   - User enters number (default: 5000)

4. "Minimum similarity score?" (0-100)
   - User enters number (default: 80)

---

## Step 5: Build pageInstructions

Build pageInstructions for SEARCHRESULTS context:

```javascript
pageInstructions: [
  // Update SEARCHRESULTS context
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: {
        pyID: "DC-1"  // Selected collection
      },
      MustReturnResults: true,  // From advanced settings
      MaxChunk: 5,
      MaxChunkTotalSize: 5000,
      DataSourcesCSV: "",
      ResponseAttributesCSV: "",
      MinSimilarityScore: 80
    }
  },

  // Delete existing data sources
  {
    instruction: "DELETE ALL",
    target: ".ContextDataConfigs(1).DataSources"
  },

  // Insert each selected data source
  {
    target: ".ContextDataConfigs(1).DataSources",
    content: { pyID: "SRC-2001" },
    listIndex: 1,
    instruction: "INSERT"
  },
  {
    target: ".ContextDataConfigs(1).DataSources",
    content: { pyID: "SRC-2" },
    listIndex: 2,
    instruction: "INSERT"
  },

  // Insert each selected response attribute
  {
    target: ".ContextDataConfigs(1).ResponseAttributes",
    content: { Name: "contentID" },
    listIndex: 1,
    instruction: "INSERT"
  },
  {
    target: ".ContextDataConfigs(1).ResponseAttributes",
    content: { Name: "contentKey" },
    listIndex: 2,
    instruction: "INSERT"
  },
  {
    target: ".ContextDataConfigs(1).ResponseAttributes",
    content: { Name: "contentSourceSystem" },
    listIndex: 3,
    instruction: "INSERT"
  }
]
```

---

## Default Values Reference

| Setting | Default Value | Description |
|---------|---------------|-------------|
| MustReturnResults | true | Fail if no results found |
| MaxChunk | 5 | Maximum number of chunks |
| MaxChunkTotalSize | 5000 | Maximum total size of chunks |
| MinSimilarityScore | 80 | Minimum similarity threshold (0-100) |

---

## Output Variables

After this step, store:

| Variable | Type | Example |
|----------|------|---------|
| contextConfigs | array | Array with configured SEARCHRESULTS context |
| pageInstructions | array | Array of pageInstructions for context setup |

These will be combined with prompts and GenAI settings in final submission (see [08-submit-prompt.md](08-submit-prompt.md)).

---

## Next Step

Configure GenAI model settings. User will choose one of two paths:
- [07a-configure-genai-default.md](07a-configure-genai-default.md) - Use default GenAI model
- [07b-configure-genai-custom.md](07b-configure-genai-custom.md) - Select specific model
