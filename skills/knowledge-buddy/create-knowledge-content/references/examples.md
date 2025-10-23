# Complete Working Examples

This document provides complete, working examples with actual values for creating knowledge base content.

---

## Example 1: Text Content with Advanced Settings

This example demonstrates the complete workflow for creating text-based content with custom chunking parameters.

### Scenario

- **Collection**: "knowledge" (DC-1)
- **Data Source**: "Knowledge_ingestion_test" (SRC-1001)
- **Settings**: Advanced (custom chunking)
- **Content Format**: Text with 3 chunks
- **Title**: "Understanding Pega DX API"
- **Abstract**: "A comprehensive guide to using the Pega DX API"

### Step 0: Authenticate

```javascript
await mcp__pega-dx-mcp__authenticate_pega({});
// Verify connection succeeds before proceeding
```

### Step 1: Query Collections and Data Sources

```javascript
// Query collections
const collections = await mcp__pega-dx-mcp__get_list_data_view({
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
});

// Query data sources
const dataSources = await mcp__pega-dx-mcp__get_list_data_view({
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
});
```

### Step 2: User selects options

(User selects collection "knowledge", data source "Knowledge_ingestion_test", advanced settings with SIZE method, 1000 chunk size, 200 overlap, text format)

### Step 3: Create Case

```javascript
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
});

// Returns:
// {
//   caseID: "KB-1023",
//   assignments: [{
//     ID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
//     ...
//   }]
// }

const caseID = createResponse.caseID;
const createAssignmentID = createResponse.assignments[0].ID;
```

### Step 4: Configure Collection, Data Source, and Chunking

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    AdvancedSettings: true  // Enable advanced settings
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "knowledge",
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"  // Note the space!
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "Knowledge_ingestion_test",
        pyID: "SRC-1001",
        pzInsKey: "PEGAFW-QNA-WORK SRC-1001"  // Note the space!
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
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "Knowledge Buddy Public"
      }
    }
  ]
});

// Returns: Progresses to Draft stage
// New assignment: ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW
```

### Step 5: Author Content

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Understanding Pega DX API",
    Abstract: "A comprehensive guide to using the Pega DX API"
    // ArticleType defaults to "text"
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Introduction: This article covers the fundamentals of the Pega DX API. The DX API provides a modern, RESTful interface for interacting with Pega applications. It supports OAuth 2.1 authentication with PKCE for secure access and offers comprehensive endpoints for case management, data operations, and system configuration."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Authentication: The DX API uses OAuth 2.1 with PKCE for secure access. To authenticate, first register your application with Pega to obtain client credentials. Then use the authorization code flow with PKCE to obtain access tokens. Tokens should be refreshed before expiration to maintain continuous access."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Best Practices: Follow these guidelines for optimal API integration. Always use HTTPS for API calls. Implement proper error handling and retry logic. Cache access tokens and refresh before expiration. Use appropriate page sizes for list operations. Follow rate limiting guidelines to avoid throttling."
      }
    }
  ]
});

// Case progresses to Ingestion stage, then Resolve stage
```

### Step 6: Verify Success

```javascript
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: "KB-1023"
});

// Verify final case state shows:
console.log("Case ID:", finalCase.pyID);                    // "KB-1023"
console.log("Status:", finalCase.pyStatusWork);             // "Resolved-Published"
console.log("Collection:", finalCase.CollectionName);       // "knowledge"
console.log("Data Source:", finalCase.DataSourceName);      // "Knowledge_ingestion_test"
console.log("Title:", finalCase.pyLabel);                   // "Understanding Pega DX API"
console.log("Abstract:", finalCase.Abstract);               // "A comprehensive guide..."
console.log("Vector Store:", finalCase.VectorStoreCollectionName); // "knowledge"

// Get detailed content view
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"
});

console.log("Chunks:", authorView.Chunks);  // [object Object],[object Object],[object Object]
// Shows 3 chunks (each chunk is an object with Content property)
```

**Result**: Content successfully created with 3 text chunks, custom chunking parameters, and published to knowledge base.

---

## Example 2: File Content with Simple Settings

This example demonstrates the complete workflow for uploading a file with default chunking parameters.

### Scenario

- **Collection**: "knowledge" (DC-1)
- **Data Source**: "Knowledge_ingestion_test" (SRC-1001)
- **Settings**: Simple (default chunking)
- **Content Format**: File (PDF document)
- **File Path**: "/path/to/policy-document.pdf"
- **Title**: "Company Policy Document"
- **Abstract**: "Internal policies for employee handbook"

### Step 0: Authenticate

```javascript
await mcp__pega-dx-mcp__authenticate_pega({});
```

### Steps 1-2: Query and User Selection

(Same as Example 1, but user selects simple settings and file format)

### Step 3: Create Case

```javascript
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
});

const caseID = createResponse.caseID;  // "KB-1026"
const createAssignmentID = createResponse.assignments[0].ID;
```

### Step 4: Configure (Simple Settings)

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {},  // No AdvancedSettings
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "knowledge",
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "Knowledge_ingestion_test",
        pyID: "SRC-1001",
        pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "Knowledge Buddy Public"
      }
    }
    // No IndexParams - uses defaults (SIZE, 1000, 200)
  ]
});

// New assignment: ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW
```

### Step 5a: Upload File

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/policy-document.pdf",
  fileName: "policy-document.pdf"
});

const tempAttachmentID = uploadResponse.data.ID;  // e.g., "ATTACH-12345"
console.log("Uploaded file, temporary ID:", tempAttachmentID);
```

### Step 5b: Refresh Assignment

```javascript
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Company Policy Document",
    Abstract: "Internal policies for employee handbook",
    ArticleType: "file"  // Critical: Sets to file mode
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }
  }]
});

console.log("Refreshed assignment with ContentAttachment");
```

### Step 5c: Submit Action

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Company Policy Document",
    Abstract: "Internal policies for employee handbook",
    ArticleType: "file"  // Critical: Sets to file mode
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }  // MUST include in submit too!
  }]
});

console.log("Submitted content for ingestion");
// Case progresses to Ingestion stage, then Resolve stage
```

### Step 6: Verify Success

```javascript
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: "KB-1026"
});

// Verify final case state
console.log("Case ID:", finalCase.pyID);                    // "KB-1026"
console.log("Status:", finalCase.pyStatusWork);             // "Resolved-Published"
console.log("Article Type:", finalCase.ArticleType);        // "file"
console.log("Collection:", finalCase.CollectionName);       // "knowledge"
console.log("Data Source:", finalCase.DataSourceName);      // "Knowledge_ingestion_test"
console.log("Title:", finalCase.pyLabel);                   // "Company Policy Document"
console.log("Extraction Method:", finalCase.ExtractionMethod); // "Standard"

// Get content view to see ContentAttachment
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1026",
  viewID: "AuthorContent"
});

console.log("ContentAttachment:", authorView.ContentAttachment);  // [object Object]

// Get case attachments to see uploaded file
const attachments = await mcp__pega-dx-mcp__get_case_attachments({
  caseID: "KB-1026"
});

console.log("Attachments:", attachments);
// Shows:
// - pyAttachName: "policy-document.pdf"
// - pxObjClass: "Data-WorkAttach-File"
```

**Result**: File successfully uploaded, text extracted automatically, and content published to knowledge base with default chunking.

---

## Example 3: Simple Settings with Single Text Chunk

This example shows the simplest possible workflow: single chunk of text with all defaults.

### Complete Workflow

```javascript
// Step 0: Authenticate
await mcp__pega-dx-mcp__authenticate_pega({});

// Step 1-2: Query and User Selection (simplified)
// User selects: collection "knowledge", datasource "Knowledge_ingestion_test", simple settings, text format, 1 chunk

// Step 3: Create Case
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
});

// Step 4: Configure (Simple Settings)
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: createResponse.assignments[0].ID,
  actionID: "Create",
  content: {},
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "knowledge",
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "Knowledge_ingestion_test",
        pyID: "SRC-1001",
        pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: { AccessRoleName: "Knowledge Buddy Public" }
    }
  ]
});

// Step 5: Author Single Chunk
const authorResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE ${createResponse.caseID}!DRAFT_FLOW`,
  actionID: "AuthorContent",
  content: {
    Title: "Quick Start Guide",
    Abstract: "Getting started with the platform"
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Welcome to the platform! This quick start guide will help you get up and running in minutes. First, sign in with your credentials. Then, explore the dashboard to familiarize yourself with the interface. Finally, create your first project to begin working."
      }
    }
  ]
});

// Step 6: Verify
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: createResponse.caseID
});

console.log("✅ Created:", finalCase.pyLabel, "-", finalCase.pyStatusWork);
```

**Result**: Simple content article with single chunk, all default settings, published successfully.

---

## Key Patterns from Examples

### Pattern 1: pzInsKey Format

Always include space between class name and ID:
```javascript
pzInsKey: "PEGAFW-QNA-WORK DC-1"      // ✅ Correct
pzInsKey: "PEGAFW-QNA-WORKDC-1"       // ❌ Wrong
pzInsKey: "PEGAFW-QNA-WORK  DC-1"     // ❌ Wrong (double space)
```

### Pattern 2: PageInstructions for Embedded Pages

Always use UPDATE with dot prefix:
```javascript
{
  instruction: "UPDATE",      // Not REPLACE
  target: ".Collection",      // With dot prefix
  content: {
    CollectionName: "...",
    pyID: "...",
    pzInsKey: "..."          // All three required
  }
}
```

### Pattern 3: PageInstructions for Page Lists

Always use APPEND with dot prefix:
```javascript
{
  instruction: "APPEND",      // Not UPDATE or REPLACE
  target: ".Chunks",          // With dot prefix
  content: {
    Content: "..."           // Property name is "Content" (capital C)
  }
}
```

### Pattern 4: File Upload Sequence

Always follow 3-step process:
```javascript
// 1. Upload
const { data: { ID } } = await upload_attachment({ filePath: "..." });

// 2. Refresh
await refresh_assignment_action({
  ...,
  pageInstructions: [{ instruction: "REPLACE", target: ".ContentAttachment", content: { ID } }]
});

// 3. Submit
await perform_assignment_action({
  ...,
  pageInstructions: [{ instruction: "REPLACE", target: ".ContentAttachment", content: { ID } }]  // Same pageInstructions!
});
```

### Pattern 5: Simple vs Advanced Settings

```javascript
// Simple (no IndexParams)
content: {},
pageInstructions: [
  { Collection }, { Datasource }, { ContentAccessConfigurations }
]

// Advanced (with IndexParams)
content: { AdvancedSettings: true },
pageInstructions: [
  { Collection }, { Datasource }, { IndexParams }, { ContentAccessConfigurations }
]
```

---

## Testing Tips

### Verify Each Step

After each step, verify the result:
- Step 3: Check caseID and assignmentID are returned
- Step 4: Check case progresses to Draft stage
- Step 5: Check case progresses to Ingestion/Resolve stage
- Step 6: Check final status is "Resolved-Published"

### Use get_case_view for Debugging

```javascript
// View configuration
const createView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "Create"
});
console.log("Collection:", createView.Collection);
console.log("Datasource:", createView.Datasource);
console.log("IndexParams:", createView.IndexParams);

// View content
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"
});
console.log("Title:", authorView.Title);
console.log("Chunks:", authorView.Chunks);
```

### Common Values for Testing

```javascript
// Collection
CollectionName: "knowledge"
pyID: "DC-1"
pzInsKey: "PEGAFW-QNA-WORK DC-1"

// Data Source
Name: "Knowledge_ingestion_test"
pyID: "SRC-1001"
pzInsKey: "PEGAFW-QNA-WORK SRC-1001"

// Chunking (defaults)
ChunkingMethod: "SIZE"
ChunkSize: 1000
ChunkOverlap: 200

// Access Role
AccessRoleName: "Knowledge Buddy Public"
```

---

## Related References

- **[technical-details.md](./technical-details.md)** - Technical patterns and structures
- **[default-settings.md](./default-settings.md)** - Simple vs advanced configuration
- **[error-handling.md](./error-handling.md)** - Troubleshooting common issues
- **[success-criteria.md](./success-criteria.md)** - Verification checklist
