# Configure Context Data - Add Custom Contexts

## Overview

Add additional contexts beyond the default SEARCHRESULTS context. Supports multiple custom contexts with individual configurations.

---

## Query Flow

Queries are executed on-demand for each additional context:
1. Query D_IndexList → Present collections
2. User selects collection for each context → Query D_DataSourceListByCollection and D_AttributeList
3. Present data sources and attributes for user selection

**CRITICAL**: When querying D_DataSourceListByCollection, the parameter name MUST be `pyID`, NOT `CollectionID`. See inline query examples below.

---

## Step 1: Ask Number of Additional Contexts

**Atomic question**: "How many additional contexts do you want to add?"

```javascript
AskUserQuestion({
  questions: [{
    question: "How many additional contexts do you want to add? (beyond SEARCHRESULTS)",
    header: "Context count",
    multiSelect: false,
    options: [
      { label: "1", description: "One additional context" },
      { label: "2", description: "Two additional contexts" },
      { label: "3", description: "Three additional contexts" }
    ]
  }]
})
```

**Store**: Number of additional contexts (e.g., 2)

---

## Step 2: Configure SEARCHRESULTS

**Atomic question**: "What do you want to do with the default SEARCHRESULTS context?"

```javascript
AskUserQuestion({
  questions: [{
    question: "What do you want to do with the default SEARCHRESULTS context?",
    header: "SEARCHRESULTS",
    multiSelect: false,
    options: [
      { label: "Keep defaults", description: "Keep SEARCHRESULTS with default values" },
      { label: "Edit it", description: "Customize SEARCHRESULTS before adding other contexts" }
    ]
  }]
})
```

**If "Keep defaults"**: Keep SEARCHRESULTS with defaults → Proceed to Step 3

**If "Edit it"**: Follow [06b-configure-context-edit-default.md](06b-configure-context-edit-default.md) steps for SEARCHRESULTS → Then proceed to Step 3

---

## Step 3: For Each Additional Context

**For each additional context** (repeat N times based on Step 1):

### Question 3.1: Context Name

```javascript
AskUserQuestion({
  questions: [{
    question: "Enter name for additional context {index}:",
    header: "Context name",
    multiSelect: false,
    options: [
      { label: "ADDITIONAL_CONTEXT", description: "Additional context data" },
      { label: "EXTRA_DOCS", description: "Extra documentation" },
      { label: "Enter custom", description: "Provide custom name" }
    ]
  }]
})
```

**Store**: Context name (e.g., "ADDITIONAL_CONTEXT")

---

### Question 3.2: Select Collection

**Same as [06b-configure-context-edit-default.md](06b-configure-context-edit-default.md) Step 1**

```javascript
AskUserQuestion({
  questions: [{
    question: "Select collection for {contextName} context:",
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

**Store**: Selected collection's `pyID` (e.g., "DC-2")

---

## Query Data Sources and Attributes for This Context

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
    pyID: "DC-2",  // CRITICAL: Parameter name is "pyID", NOT "CollectionID"
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
    indexName: "docs"  // Use CollectionName (not pyID) here
  }
})
```

---

### Question 3.3: Select Data Sources

Options: Select all / Select none / Enter specific data sources

---

### Question 3.4: Select Response Attributes

**Same as [06b-configure-context-edit-default.md](06b-configure-context-edit-default.md) Step 3**

Options: Select all / Select none / Enter specific attributes

---

### Question 3.5: Advanced Settings

**Same as [06b-configure-context-edit-default.md](06b-configure-context-edit-default.md) Step 4**

If yes, ask:
- Search must return results? (true/false)
- Maximum number of chunks? (default: 5)
- Maximum total size of chunks? (default: 5000)
- Minimum similarity score? (default: 80)

---

## Step 4: Build pageInstructions for Multiple Contexts

Build pageInstructions with SEARCHRESULTS (index 1) + additional contexts (index 2, 3, ...):

```javascript
pageInstructions: [
  // ============================================
  // Context 1: SEARCHRESULTS (keep defaults or edit)
  // ============================================
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: { pyID: "" },  // Or selected collection if edited
      MustReturnResults: true,
      MaxChunk: 5,
      MaxChunkTotalSize: 5000,
      DataSourcesCSV: "",
      ResponseAttributesCSV: "",
      MinSimilarityScore: 80
    }
  },

  // If SEARCHRESULTS was edited, configure its data sources
  { instruction: "DELETE ALL", target: ".ContextDataConfigs(1).DataSources" },
  { target: ".ContextDataConfigs(1).DataSources", content: { pyID: "SRC-1" }, listIndex: 1, instruction: "INSERT" },

  // ============================================
  // Context 2: ADDITIONAL_CONTEXT
  // ============================================
  {
    target: ".ContextDataConfigs",
    content: {},
    listIndex: 2,
    instruction: "INSERT"
  },

  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 2,
    content: {
      Name: "ADDITIONAL_CONTEXT",
      Collection: { pyID: "DC-2" },  // User-selected
      MustReturnResults: false,
      MaxChunk: 3,
      MaxChunkTotalSize: 3000,
      DataSourcesCSV: "",
      ResponseAttributesCSV: "",
      MinSimilarityScore: 70
    }
  },

  // Configure data sources for Context 2
  { instruction: "DELETE ALL", target: ".ContextDataConfigs(2).DataSources" },
  { target: ".ContextDataConfigs(2).DataSources", content: { pyID: "SRC-3" }, listIndex: 1, instruction: "INSERT" },

  // Insert response attributes for Context 2
  { target: ".ContextDataConfigs(2).ResponseAttributes", content: { Name: "contentURL" }, listIndex: 1, instruction: "INSERT" },

  // ============================================
  // Context 3: EXTRA_DOCS (if 3 contexts selected)
  // ============================================
  {
    target: ".ContextDataConfigs",
    content: {},
    listIndex: 3,
    instruction: "INSERT"
  },

  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 3,
    content: {
      Name: "EXTRA_DOCS",
      Collection: { pyID: "DC-3" },
      MustReturnResults: true,
      MaxChunk: 2,
      MaxChunkTotalSize: 2000,
      DataSourcesCSV: "",
      ResponseAttributesCSV: "",
      MinSimilarityScore: 75
    }
  },

  // Configure data sources for Context 3
  { instruction: "DELETE ALL", target: ".ContextDataConfigs(3).DataSources" },
  { target: ".ContextDataConfigs(3).DataSources", content: { pyID: "SRC-4" }, listIndex: 1, instruction: "INSERT" }
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
| contextConfigs | array | Array with SEARCHRESULTS + additional contexts |
| pageInstructions | array | Array of pageInstructions for all contexts |

These will be combined with prompts and GenAI settings in final submission (see [08-submit-prompt.md](08-submit-prompt.md)).

---

## Next Step

Configure GenAI model settings. User will choose one of two paths:
- [07a-configure-genai-default.md](07a-configure-genai-default.md) - Use default GenAI model
- [07b-configure-genai-custom.md](07b-configure-genai-custom.md) - Select specific model
