# Verification and Troubleshooting

After submitting content, verify that everything was configured and ingested correctly.

## Success Criteria Checklist

A successful content ingestion meets these criteria:

### For All Content (Text and File)
- [ ] Case exists with correct ID
- [ ] Case is in "Ingestion" stage
- [ ] Collection is set (not empty)
- [ ] Datasource is set (not empty)
- [ ] ContentAccessConfigurations has at least one entry
- [ ] Title is set
- [ ] Abstract is set
- [ ] Status eventually becomes "Resolved-Completed"

### For Text Content
- [ ] ArticleType is "text"
- [ ] Chunks field shows `[object Object],[object Object]...` (not empty)
- [ ] Each chunk has Content property

### For File Content
- [ ] ArticleType is "file"
- [ ] ContentAttachment shows `[object Object]` (not empty)
- [ ] ExtractionMethod is "Standard"
- [ ] Case has file attachment(s)
- [ ] Check IngestionErrorMessage is empty or null

## Verification Commands

### 1. View Case Details

Check the case configuration and content:

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "AuthorContent"
})
```

**What to inspect:**
- `Collection` - Should show collection reference object
- `Datasource` - Should show datasource reference object
- `ContentAccessConfigurations` - Should show array notation like `[object Object]`
- `Title` - Should contain the title
- `Abstract` - Should contain the abstract
- `ArticleType` - Should be "text" or "file"
- `Chunks` - (Text mode) Should show `[object Object]...` array
- `ContentAttachment` - (File mode) Should show `[object Object]`
- `ExtractionMethod` - (File mode) Should be "Standard"
- `IngestionErrorMessage` - Should be empty/null (if present, indicates error)

### 2. Check Case Status

Monitor ingestion progress:

```javascript
mcp__pega-dx-mcp__get_case({
  caseID: "<case-id>"
})
```

**Status progression:**
1. **New** - Just created
2. **Draft** - Configuration completed, ready for content
3. **Pending-Ingestion** - Content submitted, awaiting processing
4. **Resolved-Completed** - Successfully ingested

**Note:** Ingestion can take seconds to minutes depending on content size.

### 3. Verify File Attachments (File Mode Only)

Check that file was attached:

```javascript
mcp__pega-dx-mcp__get_case_attachments({
  caseID: "<case-id>"
})
```

**Should return:**
- At least one attachment
- Type: "File"
- Category: "File"
- File name and size information

## Common Issues and Solutions

### Issue: Collection or Datasource Fields Empty

**Symptom:** Collection or Datasource shows as empty object or missing fields

**Causes:**
- pageInstructions not used during Create action
- Target path missing dot prefix (`.Collection` vs `Collection`)
- Wrong instruction type (REPLACE instead of UPDATE)
- Missing pzInsKey or incorrect format

**Solution:**
```javascript
// Correct format in Create action:
pageInstructions: [
  {
    instruction: "UPDATE",  // Not REPLACE
    target: ".Collection",  // With dot
    content: {
      CollectionName: "knowledge",
      pyID: "DC-1",
      pzInsKey: "PEGAFW-QNA-WORK DC-1"  // Correct format with space
    }
  }
]
```

### Issue: ContentAccessConfigurations Empty

**Symptom:** ContentAccessConfigurations field is empty or shows `[]`

**Causes:**
- APPEND instruction not used
- Target path incorrect
- Content missing AccessRoleName property

**Solution:**
```javascript
// Correct format in Create action:
pageInstructions: [
  {
    instruction: "APPEND",  // Not UPDATE
    target: ".ContentAccessConfigurations",  // With dot
    content: {
      AccessRoleName: "Knowledge Buddy Public"  // Exact property name
    }
  }
]
```

### Issue: Chunks Empty (Text Mode)

**Symptom:** Chunks field is empty, null, or shows `[]`

**Causes:**
- pageInstructions not used for Chunks
- Chunks set in content parameter (wrong approach)
- Target path missing dot
- Content property missing or wrong case

**Solution:**
```javascript
// Correct format in AuthorContent action:
pageInstructions: [
  {
    instruction: "APPEND",  // Add each chunk
    target: ".Chunks",      // With dot
    content: {
      Content: "chunk text here..."  // Capital C
    }
  }
  // Repeat for additional chunks
]
```

### Issue: ContentAttachment Empty (File Mode)

**Symptom:** ContentAttachment is empty or null in file mode

**Causes:**
- File not attached before AuthorContent action
- ArticleType not set to "file"
- Trying to set ContentAttachment manually via pageInstructions

**Solution:**
```javascript
// Correct sequence:
// 1. Upload file
const upload = await upload_attachment({...});

// 2. Attach to case FIRST
await add_case_attachments({
  caseID: "...",
  attachments: [{ type: "File", category: "File", ID: upload.ID }]
});

// 3. Then submit with file mode
await perform_assignment_action({
  assignmentID: "...",
  actionID: "AuthorContent",
  content: {
    ArticleType: "file"  // Critical
  }
  // DO NOT use pageInstructions for ContentAttachment
});
```

### Issue: Ingestion Fails but Workflow Succeeds

**Symptom:** Case reaches Resolved-Completed but IngestionErrorMessage contains an error

**Common causes:**
- Unsupported file format (Markdown, etc.)
- File encoding issues (non-UTF-8 text files)
- Corrupted files
- Empty or invalid file content
- Extraction service unavailable

**Solution:**
- Check `IngestionErrorMessage` field for specific error details
- For text files: ensure UTF-8 encoding
- For binary files: verify file is not corrupted
- Try different file format (PDF or DOCX more reliable than TXT/MD)
- Consider using text mode instead if file extraction keeps failing

### Issue: BAD_REQUEST Errors

**Symptom:** API returns 400 BAD_REQUEST error

**Common causes:**
- Missing required fields (pyID, pzInsKey)
- Incorrect pzInsKey format (missing space between class and ID)
- Invalid instruction type
- Malformed JSON in pageInstructions

**Solution:**
- Verify pzInsKey format: `"PEGAFW-QNA-WORK <ID>"` (with space)
- Check all required fields are present
- Validate JSON structure
- Use UPDATE for single pages, APPEND for page lists

## Verification Example

Complete verification workflow:

```javascript
// 1. Check case view
const view = await mcp__pega-dx-mcp__get_case_view({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-1023",
  viewID: "AuthorContent"
});

console.log("Collection:", view.content.Collection);
console.log("Datasource:", view.content.Datasource);
console.log("Access configs:", view.content.ContentAccessConfigurations);
console.log("Title:", view.content.Title);
console.log("Type:", view.content.ArticleType);

// For text mode:
console.log("Chunks:", view.content.Chunks);

// For file mode:
console.log("Attachment:", view.content.ContentAttachment);
console.log("Extraction:", view.content.ExtractionMethod);
console.log("Errors:", view.content.IngestionErrorMessage);

// 2. Check status
const caseData = await mcp__pega-dx-mcp__get_case({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-1023"
});

console.log("Status:", caseData.status);
console.log("Stage:", caseData.stage);

// 3. For file mode, check attachments
const attachments = await mcp__pega-dx-mcp__get_case_attachments({
  caseID: "PEGAFW-KB-WORK-ARTICLE KB-1023"
});

console.log("Attachment count:", attachments.length);
```

## What Success Looks Like

### Text Mode Success
```
Collection: { CollectionName: "knowledge", pyID: "DC-1", ... }
Datasource: { Name: "Knowledge_ingestion_test", pyID: "SRC-1001", ... }
Access configs: [object Object]
Title: "My Article Title"
Type: text
Chunks: [object Object],[object Object],[object Object]
Status: Resolved-Completed
Stage: Ingestion
```

### File Mode Success
```
Collection: { CollectionName: "knowledge", pyID: "DC-1", ... }
Datasource: { Name: "Knowledge_ingestion_test", pyID: "SRC-1001", ... }
Access configs: [object Object]
Title: "My Document"
Type: file
Attachment: [object Object]
Extraction: Standard
Errors: null
Status: Resolved-Completed
Stage: Ingestion
Attachment count: 1
```

## Next Steps

- **See complete examples:** **[examples.md](./examples.md)**
- **Technical details:** **[technical-reference.md](./technical-reference.md)**
- **Create new content:** Return to main workflow in **[SKILL.md](../SKILL.md)**
