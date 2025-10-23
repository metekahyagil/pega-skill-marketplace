# Author File Content

This guide covers authoring content by uploading documents (PDF, DOCX, TXT, etc.) with automatic text extraction.

## When to Use File Mode

Use file mode when:
- Uploading documents (PDF, DOCX, TXT, etc.)
- Content is already in a document format
- You want automatic text extraction
- Content should be indexed from a file

## Three-Step File Upload Workflow

File mode requires three distinct steps:
1. Upload file to temporary storage
2. Attach file to the case
3. Submit AuthorContent action with file mode

### Step 1: Upload File

Upload the file to get a temporary attachment ID:

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/document.pdf",
  fileName: "document.pdf"
});
```

**Returns:**
- Temporary attachment ID (e.g., "dda135c9-8a09-4d35-a2c8-a4a22d7e51ae")
- **Valid for:** 2 hours
- **Usage:** One-time use only (upload new file for each case)

### Step 2: Attach File to Case

Link the temporary attachment to your case:

```javascript
await mcp__pega-dx-mcp__add_case_attachments({
  caseID: "<case-id>",
  attachments: [{
    type: "File",
    category: "File",
    ID: "<temporary-attachment-id-from-step-1>"
  }]
});
```

**Result:** File is permanently attached to the case.

### Step 3: Submit AuthorContent with File Mode

Submit the AuthorContent action with `ArticleType: "file"`:

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<author-content-assignment-id>",
  actionID: "AuthorContent",
  content: {
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>",
    ArticleType: "file"  // Critical: Sets to file mode
  }
  // DO NOT use pageInstructions for ContentAttachment
  // DO NOT use attachments parameter here
});
```

**Returns:** Submits case for ingestion and moves to Ingestion stage.

## Key Requirements

### Content Fields
- **Title:** Article title (stored in `pyLabel`)
- **Abstract:** Summary/description
- **ArticleType:** Must be `"file"` to enable file mode

### Auto-Populated Fields (by system)
- **ContentAttachment:** Automatically linked from case attachment
- **ExtractionMethod:** Automatically set to "Standard"

### Critical Rules
- **File must be attached FIRST** using `add_case_attachments`
- **DO NOT use pageInstructions** for ContentAttachment (system auto-links)
- **DO NOT use attachments parameter** on `perform_assignment_action`
- **Upload → Attach → Submit** sequence is mandatory

## Supported File Formats

**Fully supported:**
- PDF (.pdf)
- Word (.docx)
- Text (.txt)

**May have issues:**
- Markdown (.md) - extraction may fail
- Other formats - test before production use

## Complete Example

```javascript
// Step 1: Upload file
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/home/user/policy-document.pdf",
  fileName: "policy-document.pdf"
});
// Returns: "dda135c9-8a09-4d35-a2c8-a4a22d7e51ae"

// Step 2: Attach to case
await mcp__pega-dx-mcp__add_case_attachments({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-1026",
  attachments: [{
    type: "File",
    category: "File",
    ID: "dda135c9-8a09-4d35-a2c8-a4a22d7e51ae"
  }]
});

// Step 3: Submit with file mode
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1026!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Company Policy Document",
    Abstract: "Internal policies for employee handbook",
    ArticleType: "file"
  }
});
```

**Result:**
- Case moves to Ingestion stage
- Status becomes "Pending-Ingestion"
- System extracts text from PDF
- Content indexed into knowledge base

## Common Issues

1. **File upload fails:**
   - Check file path is correct and accessible
   - Verify file format is supported
   - Ensure file is not corrupted

2. **Attachment not linked:**
   - Verify you called `add_case_attachments` before AuthorContent
   - Check case ID format is correct
   - Confirm temporary attachment ID hasn't expired (2 hour limit)

3. **ContentAttachment empty:**
   - Ensure `ArticleType: "file"` is set
   - Verify file was attached to case first
   - Don't try to set ContentAttachment manually via pageInstructions

4. **Ingestion failures:**
   - Check `IngestionErrorMessage` field for details
   - Verify file format is supported (PDF, DOCX, TXT)
   - For text files, ensure proper encoding (UTF-8)
   - For binary files, ensure not corrupted
   - **Note:** Workflow can complete successfully even if extraction fails

5. **Reusing temporary IDs:**
   - Each temporary attachment ID can only be used once
   - Upload a new file for each case
   - Expired IDs (>2 hours) cannot be reused

## Verification

After submission, verify the file was attached and configured:

```javascript
// View case details
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "AuthorContent"
})

// Check attachments
mcp__pega-dx-mcp__get_case_attachments({
  caseID: "<case-id>"
})
```

**What to check:**
- `ArticleType` - Should be "file"
- `ContentAttachment` - Should show as `[object Object]`
- `ExtractionMethod` - Should be "Standard"
- Case has 1+ file attachment(s)

For more verification details, see **[05-verification.md](./05-verification.md)**.

## Next Steps

- **Verify success:** See **[05-verification.md](./05-verification.md)**
- **For text entry instead:** See **[03-author-text.md](./03-author-text.md)**
- **Technical details:** See **[technical-reference.md](./technical-reference.md)**
