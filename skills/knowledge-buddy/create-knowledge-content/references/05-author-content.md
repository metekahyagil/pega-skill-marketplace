# Step 5: Author Content

## Purpose

Author article content based on format selected in Step 2 (Text or File).

## Content Format Options

- **Text**: Manual text entry with chunks (ArticleType="text")
- **File**: Document upload with automatic extraction (ArticleType="file")

## Option A: Text Content

Submit content with Title, Abstract, and chunks via pageInstructions.

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Understanding Pega DX API",
    Abstract: "A comprehensive guide to using the Pega DX API",
    ArticleType: "text"
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Introduction: This article covers the fundamentals..."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Authentication: The DX API uses OAuth 2.1..."
      }
    }
    // Add one APPEND per chunk
  ]
})
```

**Rules**:
- Title/Abstract in content parameter
- ArticleType: "text"
- Chunks: APPEND to `.Chunks` page list
- Each chunk has `Content` property

## Option B: File Content (3-step process)

### Step 5a: Upload File

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/document.pdf"
});
const tempAttachmentID = uploadResponse.data.ID;
```

### Step 5b: Refresh Assignment

```javascript
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
```

### Step 5c: Submit Action

```javascript
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

**Rules**:
- Upload first to get temp attachment ID
- Refresh required before submit
- PageInstructions MUST be in BOTH refresh and submit
- ArticleType: "file"
- Use REPLACE instruction for `.ContentAttachment`
- Supported formats: PDF, DOCX, TXT, MD

## Response

Case transitions: Draft (PRIM2) → Ingestion (ALT2)

Status: "Pending-Ingestion" (processing) → "Resolved-Published" (complete)

## Next Step

[06-verify-success.md](06-verify-success.md)

**Exceptions**: See [error-handling/05-author-content-exceptions.md](error-handling/05-author-content-exceptions.md) for troubleshooting.
