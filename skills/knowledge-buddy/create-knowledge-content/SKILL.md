---
name: create-knowledge-content
description: Simplified workflow for creating knowledge base content articles in Pega Knowledge Buddy applications. Use this skill when the user wants to create, author, and ingest content into a knowledge base with collection, data source, access configuration, and chunking settings. Handles the complete end-to-end workflow from case creation through content ingestion with support for multiple content chunks and access roles.
---

# Create Knowledge Content

## Overview

This skill provides a streamlined workflow for creating knowledge base content articles in Pega Knowledge Buddy applications. It automates the multi-step process of creating a Content case, configuring collection and data source associations, setting access roles, configuring chunking parameters, authoring multiple content chunks, and submitting for ingestion into the knowledge base vector store.

**When to use this skill:**
- Creating new knowledge base articles or content
- Adding content to existing collections and data sources
- Ingesting documentation, policies, or reference materials into a knowledge base
- Uploading documents (PDF, DOCX, TXT) for automatic text extraction and indexing
- User requests like "Create a new article about X", "Add content to the knowledge base", or "Upload this document to the knowledge base"

## Workflow

### Step 0: Verify Pega Connection (Prerequisite)

Before starting the content creation workflow, verify that the MCP can connect to Pega:

```javascript
// Test connectivity and authentication
mcp__pega-dx-mcp__ping_pega_service({})
```

This validates:
- OAuth2 authentication is working
- Pega server is accessible
- MCP configuration is correct

**If this fails**, check:
- `PEGA_BASE_URL` environment variable
- `PEGA_CLIENT_ID` and `PEGA_CLIENT_SECRET` credentials
- Network connectivity to Pega instance
- Pega DX API is enabled

**Only proceed with content creation if ping succeeds.**

### Step 1: Query Available Collections

Before creating content, discover what collections exist in the system:

```javascript
// Use get_list_data_view to query collections
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

Store the collection information for user selection.

### Step 2: Query Available Data Sources (Content Types)

Discover what data sources (content types) exist:

```javascript
// Use get_list_data_view to query data sources
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

Store the data source information for user selection.

### Step 3: Gather User Input

Use the AskUserQuestion tool to collect:

1. **Collection Selection**: Present available collections from Step 1
2. **Data Source Selection**: Present available data sources from Step 2 (optionally filtered by selected collection)
3. **Content Details**: Ask for:
   - Title (article title)
   - Abstract (summary/description)
   - Content (main article text/chunks)

**Important**: For better UX, can ask all three questions in a single AskUserQuestion call (up to 4 questions supported).

### Step 4: Create Content Case

Create a new Content case using the case type `PegaFW-KB-Work-Article`:

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
})
```

This creates the case and returns an assignment ID for the Create action.

### Step 5: Configure Collection, Data Source, and Access

Perform the Create action with pageInstructions to set the Collection, Datasource, IndexParams, and ContentAccessConfigurations:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id-from-step-4>",
  actionID: "Create",
  content: {
    AdvancedSettings: true  // Optional: enable advanced settings if needed
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "<selected-collection-name>",
        pyID: "<selected-collection-id>",
        pzInsKey: "<selected-collection-inskey>"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "<selected-datasource-name>",
        pyID: "<selected-datasource-id>",
        pzInsKey: "<selected-datasource-inskey>"
      }
    },
    {
      instruction: "UPDATE",
      target: ".IndexParams",
      content: {
        ChunkingMethod: "TITLE",  // Options: "TITLE", "SIZE", "NONE", "ABSTRACT" - optional, can omit for defaults
        ChunkSize: 1000,           // optional, can omit for defaults
        ChunkOverlap: 200          // optional, can omit for defaults
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "Knowledge Buddy Public"  // or other access role names
      }
    }
    // Add more access roles with additional APPEND instructions if needed
  ]
})
```

**Critical Points**:
- Collection and Datasource MUST be set using pageInstructions with the UPDATE instruction
- Use UPDATE instruction (not REPLACE) to preserve existing page structure
- Target paths MUST start with a dot (e.g., ".Collection", not "Collection")
- Both Collection AND Datasource require complete references with pyID and pzInsKey
- pzInsKey format is: `<CLASS-NAME> <ID>` (e.g., "PEGAFW-QNA-WORK DC-1" for collections, "PEGAFW-QNA-WORK SRC-1001" for data sources)
- IndexParams can be set optionally for custom chunking configuration
- ContentAccessConfigurations is a page list - use APPEND instruction to add access roles
- Multiple access roles can be added with multiple APPEND instructions

This progresses to the Draft stage with an "Author content" assignment.

### Step 6: Author Content

Perform the AuthorContent action with the user-provided content. The workflow differs based on Content Format type.

#### Option A: Text Content (Manual Entry)

For text-based content, use pageInstructions with APPEND to add content chunks:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<author-content-assignment-id>",
  actionID: "AuthorContent",
  content: {
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>",
    ArticleType: "text"  // Explicitly set to text mode
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "<first-chunk-text>"
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "<second-chunk-text>"
      }
    }
    // Add more chunks as needed
  ]
})
```

**Key Points**:
- **ArticleType**: Set to "text" for manual entry mode
- **Title and Abstract**: Set via content parameter as scalar fields
- **Chunks**: A page list that requires APPEND pageInstructions for each chunk
- **Target**: Must be ".Chunks" (with dot prefix)
- **Content property**: Each chunk object has a "Content" property containing the text
- **Multiple chunks**: Add one APPEND instruction per chunk

#### Option B: File Content (Upload Document)

For file-based content (PDF, DOCX, TXT, etc.), use a 3-step process with upload, refresh, and submit:

**Step 6a: Upload File**
```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/document.pdf",
  fileName: "document.pdf"
});
// Returns: Temporary attachment ID (valid for 2 hours)
const tempAttachmentID = uploadResponse.data.ID;
```

**Step 6b: Refresh to Populate ContentAttachment Field**
```javascript
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "<author-content-assignment-id>",
  actionID: "AuthorContent",
  content: {
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>",
    ArticleType: "file"  // Critical: Sets to file mode
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }
  }]
})
```

**Step 6c: Submit AuthorContent with PageInstructions**
```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<author-content-assignment-id>",
  actionID: "AuthorContent",
  content: {
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>",
    ArticleType: "file"  // Critical: Sets to file mode
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }  // MUST include pageInstructions in submit too!
  }]
})
```

**Key Points**:
- **Upload first**: Use upload_attachment to get temporary attachment ID
- **Refresh required**: Call refresh_assignment_action with pageInstructions to populate ContentAttachment
- **PageInstructions in both**: MUST include same pageInstructions in BOTH refresh and submit calls
- **Target**: Must be ".ContentAttachment" (with dot prefix)
- **Instruction**: REPLACE (not UPDATE or APPEND)
- **Content**: Only needs {ID: tempAttachmentID} - Pega retrieves file metadata automatically
- **ArticleType**: Must be "file" in content parameter
- **ExtractionMethod**: Automatically set to "Standard" by the system
- **DO NOT use add_case_attachments**: That creates LINK-ATTACHMENT (different from ContentAttachment embedded page)
- **DO NOT skip refresh**: The refresh step is required to initialize the ContentAttachment embedded page

This submits the content for ingestion and moves the case to the Ingestion stage.

### Step 7: Verify Success

Retrieve the final case status to confirm successful ingestion:

```javascript
mcp__pega-dx-mcp__get_case({
  caseID: "<case-id>"
})
```

Verify:
- Status: "Resolved-Published" (or "Pending-Ingestion" if processing)
- CollectionName is set correctly
- DataSourceName is set correctly with pyID and pzInsKey
- VectorStoreCollectionName matches the collection
- IndexStatus shows completion status
- Title and Abstract are stored
- Chunks contains objects (page list) with Content property

To view the detailed content including chunks, use:

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "AuthorContent"
})
```

This will show Title, Abstract, and Chunks as structured data.

Report success to the user with the case ID and content URL.

## Key Technical Details

### Case Types Used

1. **PegaFW-KB-Work-Article** - Content case type
2. **PegaFW-QnA-Work-Index** - Collection (Data Collection case type)
3. **PegaFW-QnA-Work-DataSource** - Data Source case type

### Assignment Actions

1. **Create** - First assignment action in CREATEFORM_DEFAULT process
   - Purpose: Configure collection and data source associations
   - Uses pageInstructions for embedded page references

2. **AuthorContent** - Second assignment action in DRAFT_FLOW process
   - Purpose: Provide article title, abstract, and content (text chunks or file attachment)
   - Uses content parameter for Title, Abstract, and ArticleType (scalar fields)
   - For text mode: Uses pageInstructions with APPEND for Chunks (page list)
   - For file mode: Uses case attachments (uploaded separately)
   - Supports two Content Format options:
     - **Text**: ArticleType="text", for manually entered text content with multiple chunks
     - **File**: ArticleType="file", for uploaded documents (PDF, DOCX, etc.)

### Page Instructions for Embedded Pages

**Critical Pattern**: Embedded page references (like Collection and Datasource) cannot be set via content properties. They require pageInstructions with the UPDATE instruction:

```json
{
  "instruction": "UPDATE",
  "target": ".EmbeddedPageName",
  "content": {
    "Property": "value",
    "pyID": "ID",
    "pzInsKey": "CLASS-NAME ID"
  }
}
```

**Important Rules**:
- **Target path**: MUST start with a dot (e.g., ".Collection", ".Datasource", ".IndexParams")
- **Instruction**: Use UPDATE to add/update fields while preserving page structure
- **Complete references**: Always include pyID and pzInsKey for object references
- **pzInsKey format**: "CLASS-NAME ID" (with space, e.g., "PEGAFW-QNA-WORK DC-1")

**Available Instructions**:
- `UPDATE` - Add/update fields in existing embedded page (recommended for Collection/Datasource/IndexParams)
- `REPLACE` - Replace entire embedded page (use with caution, removes all existing fields)
- `DELETE` - Remove embedded page
- `APPEND` - Add items to page lists (required for Chunks page list)

### Content Format Options

The AuthorContent form supports two content format types controlled by the **ArticleType** field:

1. **Text** (ArticleType="text") - Manual text entry
   - Use for typing or pasting text content directly
   - Content stored in Chunks page list
   - Each chunk is an object with a "Content" property
   - Supports multiple content chunks

2. **File** (ArticleType="file") - File upload
   - Use for uploading documents (PDF, DOCX, TXT, etc.)
   - File must be uploaded and attached to case first
   - Content stored in ContentAttachment embedded page (auto-linked)
   - ExtractionMethod automatically set to "Standard"
   - System extracts text from document for ingestion

**This skill covers both Text and File formats** with complete workflows for each.

### Content Chunks Structure

**Chunks** is a page list (array) that stores multiple content pieces:

```javascript
// Each chunk in the Chunks page list has this structure:
{
  Content: "text content for this chunk..."
}
```

**To add chunks**: Use APPEND pageInstruction for each chunk:
- **Target**: ".Chunks" (with dot prefix)
- **Content object**: Must contain "Content" property with the text
- **Multiple chunks**: Add separate APPEND instruction for each chunk
- **Single chunk**: Can add just one APPEND instruction for simple content
- **Content Format**: "Text" (default) - no need to explicitly set

**Example**: Adding 3 chunks creates a page list with 3 objects, each containing a Content property

**Storage Verification**: After submission, use `get_case_view` with viewID "AuthorContent" to see chunks as `[object Object],[object Object]...`

### Access Configurations Structure

**ContentAccessConfigurations** is a page list (array) that stores access role permissions:

```javascript
// Each access configuration in the page list has this structure:
{
  AccessRoleName: "Knowledge Buddy Public"  // or other role names
}
```

**To add access configurations**: Use APPEND pageInstruction for each role:
- **Target**: ".ContentAccessConfigurations" (with dot prefix)
- **Content object**: Must contain "AccessRoleName" property
- **Multiple roles**: Add separate APPEND instruction for each role
- **Common roles**: "Knowledge Buddy Public", or other custom access roles

**Example**: Adding access role creates a page list with objects containing AccessRoleName property

**Storage Verification**: Use `get_case_view` with viewID "Create" to see ContentAccessConfigurations as `[object Object]`

### Stage Progression

The Content case progresses through these stages:
1. **Create** (PRIM0) - Initial case creation and configuration
2. **Draft** (PRIM2) - Content authoring
3. **Ingestion** (ALT2) - Content processing and vector store indexing
4. **Resolve** (PRIM3) - Final published state

### Default Settings

This skill can configure the following settings:
- **AdvancedSettings**: Can be enabled via content parameter (set to true if needed)
- **ArticleType**: text (default, automatically set)
- **Chunking**: Can be customized via IndexParams pageInstruction:
  - **ChunkingMethod**: "TITLE", "SIZE", "NONE", or "ABSTRACT"
    - **TITLE**: Chunks content by document title/sections
    - **SIZE**: Chunks content by fixed character/token size
    - **NONE**: No chunking, treats content as single unit
    - **ABSTRACT**: Chunks based on abstract/summary structure
  - **ChunkSize**: Integer value (e.g., 5000, 888) - applicable for SIZE method
  - **ChunkOverlap**: Integer value (e.g., 250, 222) - overlap between chunks for context preservation
- **Access Roles**: Can be configured via ContentAccessConfigurations pageInstruction:
  - Use APPEND instruction to add access roles
  - Each role is an object with "AccessRoleName" property
  - Common role: "Knowledge Buddy Public"
  - Multiple roles can be added with multiple APPEND instructions

For simple content creation, omit IndexParams and ContentAccessConfigurations to use system defaults. For advanced use cases, include these pageInstructions as shown in Step 5.

## Error Handling

Common issues and solutions:

1. **Empty Collection/Datasource**: If Collection or Datasource fields remain empty after Create action, verify:
   - pageInstructions are used (not content properties)
   - Target paths start with a dot (e.g., ".Collection" not "Collection")
   - UPDATE instruction is used (not REPLACE)
   - pzInsKey format is correct: "CLASS-NAME ID" with a space
   - All required fields are provided for BOTH Collection and Datasource:
     - Collection: CollectionName, pyID, pzInsKey
     - Datasource: Name, pyID, pzInsKey

2. **BAD_REQUEST Error (400)**: Usually caused by:
   - Missing dot prefix in target path
   - Trying to set embedded pages via content instead of pageInstructions
   - Incorrect pzInsKey format
   - Missing required fields (pyID or pzInsKey)

3. **Case Creation Fails**: The Content case type accepts empty content, so creation should always succeed. If it fails, check authentication and permissions.

4. **Ingestion Pending**: After AuthorContent, the case status will be "Pending-Ingestion" while the system processes the content. This is normal and expected.

5. **Empty Chunks**: If Chunks remain empty after AuthorContent submission:
   - Verify you used pageInstructions with APPEND (not content property)
   - Target must be ".Chunks" (with dot prefix)
   - Each chunk object must have "Content" property (capital C)
   - Setting Chunks as a string in content parameter may not persist

6. **Empty ContentAccessConfigurations**: If access roles are not set after Create action:
   - Verify you used pageInstructions with APPEND (not content property)
   - Target must be ".ContentAccessConfigurations" (with dot prefix)
   - Each access config object must have "AccessRoleName" property
   - Setting ContentAccessConfigurations as a string in content parameter will not work

7. **File Upload Issues**: If file-based content fails:
   - MUST use upload_attachment → refresh → submit workflow (3 steps)
   - DO NOT use add_case_attachments (creates LINK-ATTACHMENT, not ContentAttachment)
   - MUST call refresh_assignment_action with pageInstructions BEFORE submit
   - MUST include pageInstructions in BOTH refresh AND submit calls
   - Use REPLACE instruction with target ".ContentAttachment"
   - Content only needs {ID: tempAttachmentID} - filename retrieved automatically
   - Set ArticleType to "file" (not "text") in content parameter
   - If ContentAttachment shows "undefined" filename, check that upload included fileName and mimeType

8. **Ingestion Failures**: If status shows "Resolved-IngestionFailed":
   - Check IngestionErrorMessage field for specific error details (e.g., "Implementation resulted in an exception")
   - Verify file format is supported by extraction method:
     - **Supported**: PDF, DOCX, TXT
     - **May have issues**: MD (Markdown), other text formats
   - For text files, ensure proper encoding (UTF-8)
   - For binary files (PDF, DOCX), ensure files are not corrupted
   - Use "Re-ingest content" action to retry ingestion
   - **Note**: The workflow can complete successfully (case reaches Ingestion stage) even if extraction fails. This indicates the file attachment process worked correctly.

## Success Criteria

A successful content creation includes:
- ✅ Case created with valid ID
- ✅ Collection associated (CollectionName and VectorStoreCollectionName populated)
- ✅ Data Source associated (DataSourceName populated with pyID and pzInsKey)
- ✅ Access configurations set (ContentAccessConfigurations page list populated)
- ✅ Chunking parameters configured (if advanced settings enabled)
- ✅ Content authored:
  - **For Text mode**:
    - Title present (stored in pyLabel)
    - Abstract present
    - Chunks page list populated with Content objects
    - ArticleType: "text"
  - **For File mode**:
    - Title present (stored in pyLabel)
    - Abstract present
    - ContentAttachment embedded page populated (auto-linked from case attachment)
    - ArticleType: "file"
    - ExtractionMethod: "Standard"
    - Case has file attachment(s)
- ✅ Status: "Resolved-Published" (or "Pending-Ingestion" if still processing)
- ✅ Content URL available (generated after publishing)
- ✅ IndexStatus: "Completed"

**Verification Tips**:
- Use `get_case_view` with viewID "Create" to see Collection, Datasource, IndexParams, and ContentAccessConfigurations
- Use `get_case_view` with viewID "AuthorContent" to see Title, Abstract, and content structure
- **Text mode**: Chunks will show as `[object Object],[object Object]...` in the view
- **File mode**: ContentAttachment will show as `[object Object]`
- Use `get_case_attachments` for file mode to see attached documents
- ContentAccessConfigurations will show as `[object Object]` for each access role added
- Each chunk object contains a "Content" property with the text
- Each access config object contains an "AccessRoleName" property

Report the case ID and content URL to the user upon completion.

## Complete Working Example

Here's a complete example with actual values:

```javascript
// Step 1: Create case
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article"
});
// Returns: KB-1023 with assignment ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT

// Step 2: Configure with collection, data source, chunking, and access
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    AdvancedSettings: true
  },
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
      instruction: "UPDATE",
      target: ".IndexParams",
      content: {
        ChunkingMethod: "SIZE",  // Options: "TITLE", "SIZE", "NONE", "ABSTRACT"
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
// Returns: Progresses to Draft stage with assignment ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW

// Step 3: Author content with multiple chunks
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Understanding Pega DX API",
    Abstract: "A comprehensive guide to using the Pega DX API"
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Introduction: This article covers the fundamentals of the Pega DX API..."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Authentication: The DX API uses OAuth 2.1 with PKCE for secure access..."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Best Practices: Follow these guidelines for optimal API integration..."
      }
    }
  ]
});

// Verify final case state shows:
// - CollectionName: "knowledge"
// - DataSourceName: "Knowledge_ingestion_test"
// - Datasource.pyID: "SRC-1001"
// - Datasource.pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
// - VectorStoreCollectionName: "knowledge"
// - ContentAccessConfigurations: [object Object] (access roles configured)
// - Status: "Resolved-Published"
// - Title and Abstract are stored as scalar fields
// - Chunks contains 3 objects in page list format
```

## Complete Working Example - File Upload

Here's a complete example for file-based content:

```javascript
// Step 1: Create case
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article"
});
// Returns: KB-1026 with assignment ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!CREATEFORM_DEFAULT

// Step 2: Configure with collection, data source, and access
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    AdvancedSettings: true
  },
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
  ]
});

// Step 3a: Upload file
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/policy-document.pdf",
  fileName: "policy-document.pdf"
});
const tempAttachmentID = uploadResponse.data.ID;

// Step 3b: Refresh to populate ContentAttachment
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Company Policy Document",
    Abstract: "Internal policies for employee handbook",
    ArticleType: "file"
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }
  }]
});

// Step 3c: Submit with pageInstructions (CRITICAL!)
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Company Policy Document",
    Abstract: "Internal policies for employee handbook",
    ArticleType: "file"
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }  // Must include in submit too!
  }]
});

// Verify final case state shows:
// - ArticleType: "file"
// - ContentAttachment: embedded page with pyAttachName="policy-document.pdf"
// - ExtractionMethod: "Standard"
// - Status: "Pending-Ingestion" or "Resolved-Published"
// - IndexStatus.status: "Completed" (when successful)
// - Has 1 case attachment (DATA-WORKATTACH-FILE, not LINK-ATTACHMENT)
// - NOTE: ContentAttachment is an embedded page, separate from case attachments list
```
