# Query Collections and Data Sources

Before creating content, you need to discover what collections and data sources exist in the Pega Knowledge Buddy system.

## Step 1: Query Available Collections

Collections are the top-level organizational units in the knowledge base (vector stores).

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: {
    select: [
      { field: "CollectionName" },
      { field: "pyID" },
      { field: "pyLabel" },
      { field: "pyStatusWork" }
    ]
  },
  paging: { pageSize: 100 }
})
```

**Returns:** List of available collections with:
- `CollectionName` - The collection identifier (e.g., "knowledge")
- `pyID` - The collection ID (e.g., "DC-1")
- `pyLabel` - Display name
- `pyStatusWork` - Status (e.g., "Open")

**What to capture:**
- Store `CollectionName`, `pyID`, and compute `pzInsKey` for the selected collection
- `pzInsKey` format: `"PEGAFW-QNA-WORK <pyID>"` (e.g., "PEGAFW-QNA-WORK DC-1")

## Step 2: Query Available Data Sources

Data sources (content types) define how content is organized within collections.

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_DataSourceList",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" },
      { field: "pyLabel" },
      { field: "CollectionName" },
      { field: "pyStatusWork" }
    ]
  },
  paging: { pageSize: 100 }
})
```

**Returns:** List of available data sources with:
- `Name` - The data source identifier (e.g., "Knowledge_ingestion_test")
- `pyID` - The data source ID (e.g., "SRC-1001")
- `pyLabel` - Display name
- `CollectionName` - Parent collection
- `pyStatusWork` - Status

**What to capture:**
- Store `Name`, `pyID`, and compute `pzInsKey` for the selected data source
- `pzInsKey` format: `"PEGAFW-QNA-WORK <pyID>"` (e.g., "PEGAFW-QNA-WORK SRC-1001")

## Step 3: Present Options to User

Use the `AskUserQuestion` tool to let the user select:
1. Which collection to use (from Step 1 results)
2. Which data source to use (from Step 2 results, optionally filtered by selected collection)
3. Content details (title, abstract, content text or file path)

**Tip:** You can ask all questions in a single `AskUserQuestion` call (supports up to 4 questions).

## Next Step

After gathering user selections, proceed to **[02-create-and-configure.md](./02-create-and-configure.md)** to create the case and configure it.
