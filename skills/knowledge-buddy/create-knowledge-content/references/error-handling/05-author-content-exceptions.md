# Step 5 Exceptions: Content Authoring Failures

This document covers error handling and troubleshooting for Step 5 (Author Content).

**Happy Path**: [05-author-content.md](../05-author-content.md)

---

## Content Format: Text Mode Errors

### 1. Empty Chunks

#### Problem

Chunks remain empty after AuthorContent submission (text mode).

#### Symptoms

- Chunks field is empty or undefined after Step 5
- Content not stored in case
- No chunks visible in case view
- Verification shows no content

#### Root Cause

Chunks is a **page list** (array), not a scalar field. Must be set using pageInstructions with APPEND, not via content parameter.

---

#### Solutions

##### ✅ Use pageInstructions with APPEND (not content property)

**Wrong Approach**:
```javascript
content: {
  Title: "...",
  Abstract: "...",
  Chunks: "some text"  // ❌ Won't work - Chunks is a page list
}
```

**Correct Approach**:
```javascript
content: {
  Title: "...",
  Abstract: "..."
},
pageInstructions: [
  {
    instruction: "APPEND",  // ✅ APPEND for page lists
    target: ".Chunks",
    content: {
      Content: "some text"  // ✅ Each chunk is an object
    }
  }
]
```

##### ✅ Target must be ".Chunks" (with dot prefix)

**Wrong**:
```javascript
{ target: "Chunks" }      // ❌ Missing dot
```

**Correct**:
```javascript
{ target: ".Chunks" }     // ✅ With dot
```

##### ✅ Each chunk object must have "Content" property (capital C)

**Wrong Property Names**:
```javascript
{ content: { text: "..." } }         // ❌ Wrong property
{ content: { chunk: "..." } }        // ❌ Wrong property
{ content: { content: "..." } }      // ❌ Wrong case (lowercase c)
```

**Correct Property Name**:
```javascript
{ content: { Content: "..." } }      // ✅ "Content" with capital C
```

##### ✅ Multiple chunks need multiple APPEND instructions

```javascript
pageInstructions: [
  {
    instruction: "APPEND",
    target: ".Chunks",
    content: { Content: "First chunk text..." }
  },
  {
    instruction: "APPEND",
    target: ".Chunks",
    content: { Content: "Second chunk text..." }
  },
  {
    instruction: "APPEND",
    target: ".Chunks",
    content: { Content: "Third chunk text..." }
  }
]
```

##### ✅ Setting Chunks as string in content parameter may not persist

Always use pageInstructions with APPEND for Chunks. The content parameter is only for scalar fields like Title and Abstract.

---

### 2. Wrong ArticleType

#### Problem

Content submitted with wrong ArticleType setting.

#### Symptoms

- Text content submitted but ArticleType is "file"
- File content attempted but ArticleType is "text"
- Ingestion fails or behaves unexpectedly

#### Solutions

##### For Text Content

```javascript
content: {
  Title: "...",
  Abstract: "...",
  ArticleType: "text"  // ✅ Explicitly set to "text"
}
```

##### For File Content

```javascript
content: {
  Title: "...",
  Abstract: "...",
  ArticleType: "file"  // ✅ Must be "file" for uploads
}
```

**Important**: ArticleType determines how Pega processes the content. Wrong type will cause ingestion failures.

---

## Content Format: File Mode Errors

### 3. File Upload Failures

#### Problem

File-based content fails to upload or attach.

#### Symptoms

- ContentAttachment is empty or undefined
- File not attached to case
- Ingestion fails for file content
- Filename shows as "undefined"
- Error about missing attachment

#### Root Cause

File upload requires a **3-step process** and specific pageInstructions. Skipping steps or incorrect configuration causes failures.

---

#### Required 3-Step Workflow

##### Step 5a: Upload File

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/document.pdf"
});

const tempAttachmentID = uploadResponse.data.ID;
```

**Common Issues**:
- File path doesn't exist
- File path is relative (must be absolute)
- No read permissions on file
- File is locked or in use
- File is too large

##### Step 5b: Refresh Assignment

```javascript
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "...",
  actionID: "AuthorContent",
  content: {
    Title: "...",
    Abstract: "...",
    ArticleType: "file"  // Critical
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }
  }]
});
```

**Why Refresh is Required**: Initializes the ContentAttachment embedded page in the case data.

##### Step 5c: Submit Action

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "...",
  actionID: "AuthorContent",
  content: {
    Title: "...",
    Abstract: "...",
    ArticleType: "file"  // Critical
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }  // MUST include in submit too!
  }]
});
```

---

#### Solutions

##### ✅ DO NOT skip the refresh step

**Wrong** (will fail):
```javascript
// Upload
const { data: { ID } } = await upload_attachment({...});

// Submit directly - SKIP
await perform_assignment_action({
  ...,
  pageInstructions: [{ ..., content: { ID } }]
});
```

**Correct** (will succeed):
```javascript
// Upload
const { data: { ID } } = await upload_attachment({...});

// Refresh first
await refresh_assignment_action({
  ...,
  pageInstructions: [{ ..., content: { ID } }]
});

// Then submit
await perform_assignment_action({
  ...,
  pageInstructions: [{ ..., content: { ID } }]
});
```

##### ✅ MUST include pageInstructions in BOTH refresh AND submit

The same pageInstructions must be provided to both `refresh_assignment_action` and `perform_assignment_action`.

**Common Mistake**:
```javascript
// Refresh with pageInstructions
await refresh_assignment_action({
  ...,
  pageInstructions: [...]
});

// Submit WITHOUT pageInstructions - WRONG!
await perform_assignment_action({
  ...,
  // Missing pageInstructions!
});
```

**Correct**:
```javascript
// Define once, use twice
const contentAttachmentPI = [{
  instruction: "REPLACE",
  target: ".ContentAttachment",
  content: { ID: tempAttachmentID }
}];

// Refresh with pageInstructions
await refresh_assignment_action({
  ...,
  pageInstructions: contentAttachmentPI
});

// Submit with SAME pageInstructions
await perform_assignment_action({
  ...,
  pageInstructions: contentAttachmentPI
});
```

##### ✅ Use REPLACE instruction (not UPDATE or APPEND)

```javascript
{
  instruction: "REPLACE",  // ✅ Correct for ContentAttachment
  target: ".ContentAttachment",
  content: { ID: tempAttachmentID }
}
```

**Why REPLACE**: ContentAttachment is an embedded page that needs to be completely replaced with the attachment reference.

##### ✅ Target must be ".ContentAttachment" (with dot)

```javascript
{ target: ".ContentAttachment" }  // ✅ Correct
{ target: "ContentAttachment" }   // ❌ Wrong
```

##### ✅ Content only needs {ID: tempAttachmentID}

```javascript
// Correct - minimal
{
  content: { ID: tempAttachmentID }
}

// Also works but unnecessary
{
  content: {
    ID: tempAttachmentID,
    pyAttachName: "file.pdf"  // Pega retrieves this automatically
  }
}
```

**Pega automatically retrieves**:
- Filename (pyAttachName)
- File size
- MIME type
- Other metadata

##### ✅ Set ArticleType to "file" (not "text")

```javascript
content: {
  Title: "...",
  Abstract: "...",
  ArticleType: "file"  // ✅ Critical for file mode
}
```

**If ArticleType is "text"**: Pega will look for Chunks instead of ContentAttachment, causing ingestion to fail.

##### ✅ DO NOT use add_case_attachments

**Wrong Tool**:
```javascript
// This creates LINK-ATTACHMENT, not ContentAttachment
await mcp__pega-dx-mcp__add_case_attachments({...});  // ❌ Wrong!
```

**Correct Tool**:
```javascript
// This creates the temp attachment for ContentAttachment
await mcp__pega-dx-mcp__upload_attachment({...});     // ✅ Correct
```

**Why**: `add_case_attachments` creates a different type of attachment (LINK-ATTACHMENT) that doesn't work with the ContentAttachment embedded page.

---

### 4. File Not Found or Access Errors

#### Problem

Cannot upload file due to file system issues.

#### Symptoms

- "File not found" error
- Permission denied errors
- Cannot read file

#### Solutions

##### ✅ Use absolute file paths

**Wrong**:
```javascript
filePath: "./document.pdf"              // ❌ Relative path
filePath: "document.pdf"                // ❌ Relative path
```

**Correct**:
```javascript
filePath: "/home/user/documents/document.pdf"        // ✅ Absolute path
filePath: "/Users/john/Downloads/policy.pdf"         // ✅ Absolute path
filePath: "C:\\Users\\John\\Documents\\file.pdf"     // ✅ Absolute path (Windows)
```

##### ✅ Verify file exists before upload

```javascript
const fs = require('fs');

if (!fs.existsSync(filePath)) {
  throw new Error(`File not found: ${filePath}`);
}

// Proceed with upload
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({ filePath });
```

##### ✅ Check file permissions

```javascript
const fs = require('fs');

try {
  fs.accessSync(filePath, fs.constants.R_OK);
  // File is readable
} catch (err) {
  throw new Error(`Cannot read file: ${filePath}. Check permissions.`);
}
```

##### ✅ Ensure file is not locked

- Close the file in other applications
- Check if file is open in editor, PDF viewer, etc.
- Verify no other process has exclusive lock

---

### 5. Unsupported File Formats

#### Problem

File format is not supported by the extraction method.

#### Symptoms

- Ingestion fails after upload
- Status: "Resolved-IngestionFailed"
- IngestionErrorMessage about extraction failure

#### Supported Formats

**Fully Supported**:
- ✅ PDF (.pdf)
- ✅ Word Documents (.docx)
- ✅ Text Files (.txt)

**May Have Issues**:
- ⚠️ Markdown (.md) - extraction may fail
- ⚠️ Other text formats - varies

#### Solutions

##### For Unsupported Formats

**Option 1**: Convert to supported format
- Convert MD to PDF or DOCX
- Convert other formats to TXT, PDF, or DOCX

**Option 2**: Use text mode instead
- Extract text manually
- Use text mode with APPEND Chunks
- Provides more control over content

---

### 6. ContentAttachment Shows "undefined" Filename

#### Problem

ContentAttachment is set but filename is undefined.

#### Symptoms

- ContentAttachment exists but pyAttachName is empty
- Verification shows incomplete attachment data

#### Solutions

##### ✅ Include fileName in upload

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "/path/to/document.pdf",
  fileName: "document.pdf"  // Explicitly provide filename
});
```

##### ✅ Verify upload response includes ID

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({...});

if (!uploadResponse.data || !uploadResponse.data.ID) {
  throw new Error("Upload failed - no attachment ID returned");
}

const tempAttachmentID = uploadResponse.data.ID;
```

---

## Assignment Action Errors

### 7. Assignment Not Found

#### Problem

AuthorContent assignment doesn't exist.

#### Symptoms

- "Assignment not found" error
- Cannot perform AuthorContent action

#### Solutions

##### ✅ Verify assignment ID from Step 4

After Step 4 (Configure), a new assignment is created:
```
ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW
```

Use this assignment ID, not the one from Step 3.

##### ✅ Check case progressed to Draft stage

```javascript
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });
console.log("Stage:", caseData.pyStatusWork);  // Should be in Draft stage
```

If still in Create stage, Step 4 may have failed or not completed.

---

### 8. Action ID Incorrect

#### Problem

Wrong actionID specified.

#### Solution

```javascript
actionID: "AuthorContent"  // ✅ Must be exactly "AuthorContent" (case-sensitive)
```

**Not**:
- "authorcontent"
- "Author Content"
- "AUTHORCONTENT"
- "authorContent"

---

## Troubleshooting Checklist

### For Text Mode

- [ ] Using **pageInstructions** with APPEND for Chunks
- [ ] Target is **".Chunks"** (with dot)
- [ ] Each chunk has **"Content"** property (capital C)
- [ ] ArticleType set to **"text"**
- [ ] Title and Abstract in content parameter
- [ ] Assignment ID is correct from Step 4
- [ ] Action ID is exactly "AuthorContent"

### For File Mode

- [ ] File path is **absolute** (not relative)
- [ ] File **exists** and is **readable**
- [ ] File format is **supported** (PDF, DOCX, TXT)
- [ ] **3-step workflow**: upload → refresh → submit
- [ ] **pageInstructions included** in BOTH refresh AND submit
- [ ] Using **REPLACE** instruction with ".ContentAttachment"
- [ ] ArticleType set to **"file"**
- [ ] Temporary attachment ID captured from upload
- [ ] **NOT using** add_case_attachments
- [ ] Title and Abstract in content parameter
- [ ] Assignment ID is correct from Step 4
- [ ] Action ID is exactly "AuthorContent"

---

## Verification After Author Content

After Step 5 completes, verify content was authored successfully:

### For Text Mode

```javascript
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"
});

console.log("Title:", authorView.Title);
console.log("Abstract:", authorView.Abstract);
console.log("Chunks:", authorView.Chunks);  // Should show [object Object],[object Object]...
```

### For File Mode

```javascript
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1026",
  viewID: "AuthorContent"
});

console.log("Title:", authorView.Title);
console.log("Abstract:", authorView.Abstract);
console.log("ContentAttachment:", authorView.ContentAttachment);  // Should show [object Object]

// Verify file attachment
const attachments = await mcp__pega-dx-mcp__get_case_attachments({
  caseID: "KB-1026"
});

console.log("Attachments:", attachments);  // Should show uploaded file
```

---

## Related Documentation

- **Happy Path**: [05-author-content.md](../05-author-content.md)
- **Configuration Errors**: [04-configure-case-exceptions.md](./04-configure-case-exceptions.md)
- **Next Step**: [06-verify-success.md](../06-verify-success.md)
- **Error Handling Index**: [index.md](./index.md)
