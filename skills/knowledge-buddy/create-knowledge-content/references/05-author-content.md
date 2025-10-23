# Step 5: Author Content

After configuring the case, author the article content. The workflow differs based on the **Content Format** selected by the user in Step 2.

## Content Format Options

The AuthorContent form supports two modes:
- **Text** (ArticleType="text"): Manual text entry with multiple chunks
- **File** (ArticleType="file"): File upload with automatic extraction

Choose the appropriate workflow based on user's selection in Step 2a.

---

## Option A: Text Content (Manual Entry)

Use this workflow when user selected "Text" content format in Step 2.

### Structure

- **Title and Abstract**: Set via `content` parameter (scalar fields)
- **Chunks**: Set via `pageInstructions` with APPEND (page list)
- **ArticleType**: Set to "text"

### Implementation

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

### Key Points

- **ArticleType**: Must be "text" for manual entry
- **Title/Abstract**: Scalar fields set in content parameter
- **Chunks**: Page list (array) requiring APPEND instructions
- **Target**: Must be ".Chunks" (with dot prefix)
- **Content property**: Each chunk object has a "Content" property (capital C)
- **Multiple chunks**: Add one APPEND instruction per chunk

### Example with 3 Chunks

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
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
})
```

---

## Option B: File Content (Upload Document)

Use this workflow when user selected "File" content format in Step 2.

### Overview

File upload requires a **3-step process**:
1. Upload file to get temporary attachment ID
2. Refresh assignment with pageInstructions
3. Submit action with same pageInstructions

### Step 5a: Upload File

Upload the file and get a temporary attachment ID (valid for 2 hours):

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "<user-provided-file-path>"  // From Step 2c
});

const tempAttachmentID = uploadResponse.data.ID;
```

**Returns**: Temporary attachment ID (e.g., "ATTACH-12345")

### Step 5b: Refresh Assignment

Refresh the assignment to populate the ContentAttachment field:

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

### Step 5c: Submit Action

Submit the AuthorContent action with the same pageInstructions:

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
    content: { ID: tempAttachmentID }  // MUST include in submit too!
  }]
})
```

### Key Points for File Upload

- **Upload first**: Use `upload_attachment` to get temporary attachment ID
- **Refresh required**: Call `refresh_assignment_action` before submit
- **PageInstructions in both**: Include in BOTH refresh AND submit calls
- **Target**: Must be ".ContentAttachment" (with dot prefix)
- **Instruction**: Use REPLACE (not UPDATE or APPEND)
- **Content**: Only needs `{ID: tempAttachmentID}` - Pega retrieves metadata automatically
- **ArticleType**: Must be "file" in content parameter
- **ExtractionMethod**: Automatically set to "Standard" by system

### What NOT to Do

❌ **DO NOT use `add_case_attachments`**: This creates LINK-ATTACHMENT, not ContentAttachment
❌ **DO NOT skip refresh**: The refresh step initializes the ContentAttachment embedded page

### Complete File Upload Example

```javascript
// Step 5a: Upload
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/policy-document.pdf",
  fileName: "policy-document.pdf"
});
const tempAttachmentID = uploadResponse.data.ID;

// Step 5b: Refresh
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

// Step 5c: Submit
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
    content: { ID: tempAttachmentID }
  }]
});
```

### Supported File Formats

- **PDF** (.pdf)
- **Word Documents** (.docx)
- **Text Files** (.txt)
- **Markdown** (.md) - may have issues with extraction

---

## What Happens After Submission

After successful submission:
- Case progresses to **Ingestion** stage (ALT2)
- Status: "Pending-Ingestion" (while processing)
- System processes content and adds to vector store
- Status changes to "Resolved-Published" when complete

## Next Step

After authoring content, proceed to **[06-verify-success.md](./06-verify-success.md)** to verify successful ingestion.

---

**Exceptions**: See **[error-handling/05-author-content-exceptions.md](error-handling/05-author-content-exceptions.md)** for content authoring failures and troubleshooting.
