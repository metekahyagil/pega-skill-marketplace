# Complete Working Examples

This document provides complete end-to-end examples matching the new workflow.

## Example 1: Text Content with Simple Settings

Complete flow for creating text-based content with default chunking settings.

### Step 1: Create Case

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: { pyAddCaseContextPage: {} },
  processID: "pyStartCase"
})
// Response: caseID="KB-2020", assignmentID="ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!CREATEFORM_DEFAULT"
```

### Step 2: Query and Select Collection

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  dataViewParameters: {},
  paging: { pageNumber: 1, pageSize: 50 },
  query: {
    select: [{ field: "pyID" }, { field: "CollectionName" }],
    distinctResultsOnly: "true"
  }
})
// User selects: CollectionName="knowledge", pyID="DC-1"
```

### Step 3: Refresh Assignment

```javascript
mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!CREATEFORM_DEFAULT",
  actionID: "Create"
})
```

### Step 4: Query and Select Data Source

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_ContentAccessDataSourcesByCollection",
  dataViewParameters: { Available: "Yes", CollectionID: "DC-1" }
})
// User selects: Name="Knowledge_source", pyID="SRC-1"
```

### Step 5: Query and Select Access Roles

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  dataViewParameters: {}
})
// User selects (multi-select): ["KnowledgeBuddy:Admin", "KnowledgeBuddy:Author"]
```

### Step 6: Configure Chunking (Simple Mode)

User selects: "No" (use default settings)

### Step 7: Submit Configuration

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!CREATEFORM_DEFAULT",
  actionID: "Create",
  viewType: "page",
  content: {
    Collection: { pyID: "DC-1" },
    Datasource: { pyID: "SRC-1" },
    AdvancedSettings: false
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        pyID: "SRC-1",
        pzInsKey: "PEGAFW-QNA-WORK SRC-1"
      }
    },
    {
      target: ".ContentAccessConfigurations",
      content: {},
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      content: { AccessRoleName: "KnowledgeBuddy:Admin" },
      target: ".ContentAccessConfigurations",
      listIndex: 1,
      instruction: "UPDATE"
    },
    {
      target: ".ContentAccessConfigurations",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },
    {
      content: { AccessRoleName: "KnowledgeBuddy:Author" },
      target: ".ContentAccessConfigurations",
      listIndex: 2,
      instruction: "UPDATE"
    },
    {
      target: ".ContentAccessConfigurations",
      content: {},
      listIndex: 3,
      instruction: "INSERT"
    },
    {
      target: ".ContentAccessConfigurations",
      listIndex: 3,
      instruction: "DELETE"
    }
  ]
})
// Response: New assignmentID="ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!DRAFT_FLOW"
```

### Step 8: Refresh After Configuration

```javascript
mcp__pega-dx-mcp__refresh_case_view({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-2020",
  viewID: "pyDetailsTabContent"
})
```

### Step 9: Author Text Content

User selects: Format="Text", Title="API Guide", Abstract="Guide to Pega DX API", Chunks=2

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2020!DRAFT_FLOW",
  actionID: "AuthorContent",
  viewType: "form",
  content: {
    ArticleType: "text",
    Title: "API Guide",
    Abstract: "Guide to Pega DX API"
  },
  pageInstructions: [
    {
      target: ".Chunks",
      content: {},
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      content: { Content: "<p>This is first content</p>" },
      target: ".Chunks",
      listIndex: 1,
      instruction: "UPDATE"
    },
    {
      target: ".Chunks",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },
    {
      content: { Content: "<p>This is second <strong>content</strong>.</p>" },
      target: ".Chunks",
      listIndex: 2,
      instruction: "UPDATE"
    }
  ]
})
```

### Step 10: Verify and Display

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-2020",
  viewID: "pyCaseSummary"
})

mcp__pega-dx-mcp__get_case_view({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-2020",
  viewID: "pyDetailsTabContent"
})
```

Display summary to user with case ID, status, and configuration details.

---

## Example 2: File Content with Advanced Settings

Complete flow for uploading a file with custom chunking settings.

### Steps 1-5: Same as Example 1

(Create case, query collections, refresh, query data sources, query access roles)

### Step 6: Configure Chunking (Advanced Mode)

User selects: "Yes" → Method="SIZE", Size=1000, Overlap=200

### Step 7: Submit Configuration with Advanced Settings

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2021!CREATEFORM_DEFAULT",
  actionID: "Create",
  viewType: "page",
  content: {
    Collection: { pyID: "DC-1" },
    Datasource: { pyID: "SRC-2" },
    AdvancedSettings: true
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        pyID: "SRC-2",
        pzInsKey: "PEGAFW-QNA-WORK SRC-2"
      }
    },
    {
      instruction: "UPDATE",
      target: ".IndexParams",
      content: {
        ChunkingMethod: "SIZE",
        ChunkSize: 1000,
        ChunkOverlap: 200
      }
    },
    {
      target: ".ContentAccessConfigurations",
      content: {},
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      content: { AccessRoleName: "KnowledgeBuddy:Admin" },
      target: ".ContentAccessConfigurations",
      listIndex: 1,
      instruction: "UPDATE"
    },
    {
      target: ".ContentAccessConfigurations",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },
    {
      content: { AccessRoleName: "KnowledgeBuddy:BuddyManager" },
      target: ".ContentAccessConfigurations",
      listIndex: 2,
      instruction: "UPDATE"
    }
  ]
})
```

### Step 8: Refresh After Configuration

```javascript
mcp__pega-dx-mcp__refresh_case_view({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-2021",
  viewID: "pyDetailsTabContent"
})
```

### Step 9: Author File Content

User selects: Format="File", Title="Policy Document", Abstract="Company policies", FilePath="/path/to/policy.pdf"

#### Step 9a: Upload File

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/policy.pdf",
  appendUniqueIdToFileName: true
})
// Response: ID="8f93ae83-78a8-44b5-a603-b7bd12e087c5"
```

#### Step 9b: Refresh with Attachment

```javascript
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2021!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    ArticleType: "file",
    Title: "Policy Document",
    Abstract: "Company policies"
  },
  pageInstructions: [{
    target: ".ContentAttachment",
    content: { ID: "8f93ae83-78a8-44b5-a603-b7bd12e087c5" },
    instruction: "REPLACE"
  }]
})
```

#### Step 9c: Submit with Attachment

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-2021!DRAFT_FLOW",
  actionID: "AuthorContent",
  viewType: "form",
  content: {
    ArticleType: "file",
    Title: "Policy Document",
    Abstract: "Company policies"
  },
  pageInstructions: [{
    target: ".ContentAttachment",
    content: { ID: "8f93ae83-78a8-44b5-a603-b7bd12e087c5" },
    instruction: "REPLACE"
  }]
})
```

### Step 10: Verify and Display

Same as Example 1 - retrieve case summary and details, display to user.

---

## Key Patterns Summary

### Collection and Data Source References

Always include all three fields:
```javascript
{
  pyID: "DC-1",  // or data source ID
  pzInsKey: "PEGAFW-QNA-WORK DC-1"  // Class name + space + ID
}
```

### Access Roles (Page List)

Use INSERT+UPDATE+DELETE pattern:
```javascript
// For each role: INSERT empty, UPDATE with data
{ target: ".ContentAccessConfigurations", content: {}, listIndex: N, instruction: "INSERT" },
{ content: { AccessRoleName: "RoleName" }, target: ".ContentAccessConfigurations", listIndex: N, instruction: "UPDATE" }
// Final: INSERT empty, DELETE to clean up
{ target: ".ContentAccessConfigurations", content: {}, listIndex: N+1, instruction: "INSERT" },
{ target: ".ContentAccessConfigurations", listIndex: N+1, instruction: "DELETE" }
```

### Advanced Settings

Only include when `AdvancedSettings: true`:
```javascript
{
  instruction: "UPDATE",
  target: ".IndexParams",
  content: {
    ChunkingMethod: "SIZE",
    ChunkSize: 1000,
    ChunkOverlap: 200
  }
}
```

### File Upload

3-step process:
1. Upload file → get attachment ID
2. Refresh with pageInstructions
3. Submit with same pageInstructions

Both refresh and submit must include:
```javascript
pageInstructions: [{
  target: ".ContentAttachment",
  content: { ID: attachmentID },
  instruction: "REPLACE"
}]
```
