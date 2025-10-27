# Step 1: Query Available Data

## Purpose

Query Pega data pages to retrieve collections, data sources, access roles, and optionally existing content for user selection.

## Table of Contents

- [Query 1: Collections](#query-1-collections)
- [Query 2: Data Sources](#query-2-data-sources)
- [Query 3: Access Roles](#query-3-access-roles)
- [Query 4: Existing Content (Optional)](#query-4-existing-content-optional)

## MCP Tool

Use `mcp__pega-dx-mcp__get_list_data_view` for all queries. Query in parallel for better performance.

## Query 1: Collections

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: {
    select: [
      { field: "pyID" },
      { field: "CollectionName" }
    ]
  },
  paging: { pageSize: 50 }
})
```

**Extract**: CollectionName, pyID, compute pzInsKey as `"PEGAFW-QNA-WORK <pyID>"`

## Query 2: Data Sources

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_ContentAccessDataSourcesByCollection",
  dataViewParameters: {
    Available: "Yes",
    CollectionID: ""  // Leave empty for all, or filter by collection pyID
  }
})
```

**Extract**: Data source information, compute pzInsKey as `"PEGAFW-QNA-WORK <pyID>"`

## Query 3: Access Roles

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  dataViewParameters: {}
})
```

**Common Access Roles**:

| Access Role                      | Label                   | Purpose                    |
|----------------------------------|-------------------------|----------------------------|
| KnowledgeBuddy:Public            | Knowledge buddy public  | Public access              |
| KnowledgeBuddy:Internal          | Internal                | Internal users only        |
| KnowledgeBuddy:Author            | Knowledge buddy author  | Content authors            |
| KnowledgeBuddy:BuddyManager      | Knowledge buddy manager | Knowledge base managers    |
| KnowledgeBuddy:DataSourceManager | Data source manager     | Data source administrators |
| KnowledgeBuddy:Admin             | Buddy administrator     | Full administrative access |

**Default**: If no selection, use `"KnowledgeBuddy:Public"`. User can select multiple roles.

## Query 4: Existing Content (Optional)

Query to understand content structure or check for similar content:

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_ContentList",
  query: {
    select: [
      { field: "pyID" },
      { field: "Title" },
      { field: "pyStatusWork" },
      { field: "ArticleType" },
      { field: "CollectionName" },
      { field: "DataSourceName" }
    ]
  },
  paging: { pageSize: 10 }
})
```

## Usage

Store queried data for use in Step 2 (user input) and Steps 4-5 (configuration/authoring).

## Next Step

[02-user-input.md](02-user-input.md)

**Exceptions**: See [error-handling/01-query-data-exceptions.md](error-handling/01-query-data-exceptions.md) for troubleshooting.
