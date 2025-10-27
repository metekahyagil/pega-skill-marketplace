# Step 5 Exceptions: Content Authoring Failures

## Text Mode Errors

### Symptom
`perform_assignment_action` fails for text content.

### Common Issues

**Missing ArticleType**:
```javascript
// ❌ Wrong
content: {
  Title: "...",
  Abstract: "..."
  // Missing ArticleType!
}

// ✅ Correct
content: {
  Title: "...",
  Abstract: "...",
  ArticleType: "text"
}
```

**Chunks PageInstructions Issues**:
```javascript
// ❌ Wrong target or instruction
{
  instruction: "UPDATE",  // Should be APPEND
  target: "Chunks",       // Missing dot
  content: { Content: "..." }
}

// ✅ Correct
{
  instruction: "APPEND",
  target: ".Chunks",
  content: { Content: "..." }
}
```

**No Chunks Provided**:
- Must have at least 1 APPEND instruction to `.Chunks`
- Each chunk needs `Content` property (capital C)

### Fix
1. Set ArticleType: "text"
2. Use APPEND to `.Chunks` (with dot)
3. Each chunk: `{ Content: "text here" }`
4. Provide at least 1 chunk

## File Mode Errors (3-Step Process)

### Symptom
File upload fails or ContentAttachment not populated.

### Critical Rules for File Mode

**Step 5a: Upload File**
```javascript
// Upload and get temp ID
const uploadResponse = await upload_attachment({ filePath: "..." });
const tempAttachmentID = uploadResponse.data.ID;
```

**Step 5b: Refresh Assignment** (REQUIRED)
```javascript
// ❌ Wrong - skipping refresh
perform_assignment_action(...)

// ✅ Correct - refresh first
await refresh_assignment_action({
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

**Step 5c: Submit Action**
```javascript
// Must include same pageInstructions
await perform_assignment_action({
  assignmentID: "...",
  actionID: "AuthorContent",
  content: {
    Title: "...",
    Abstract: "...",
    ArticleType: "file"
  },
  pageInstructions: [{
    instruction: "REPLACE",
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID }
  }]
});
```

### Common File Mode Mistakes

**Using Wrong MCP Tool**:
- ❌ Don't use `add_case_attachments` (creates LINK-ATTACHMENT)
- ✅ Use `upload_attachment` then `refresh_assignment_action`

**Skipping Refresh Step**:
- ❌ Upload → Submit directly
- ✅ Upload → Refresh → Submit

**Missing PageInstructions in Submit**:
- ❌ pageInstructions only in refresh
- ✅ pageInstructions in BOTH refresh AND submit

**Wrong Instruction**:
- ❌ UPDATE or APPEND
- ✅ REPLACE for `.ContentAttachment`

**Missing ArticleType**:
- ArticleType: "file" required in content

### Fix
1. Use `upload_attachment` to get temp ID
2. Call `refresh_assignment_action` with pageInstructions
3. Call `perform_assignment_action` with SAME pageInstructions
4. Use REPLACE instruction for `.ContentAttachment`
5. Set ArticleType: "file"
6. Content: `{ ID: tempAttachmentID }`

## File Format Errors

### Symptom
Unsupported file format or extraction fails.

### Fix
- Supported: PDF, DOCX, TXT, MD
- Verify file readable and not corrupted
- Check file size limits
- MD files may have extraction issues

## Troubleshooting Checklist

**Text Mode**:
- [ ] ArticleType: "text"
- [ ] At least 1 APPEND to `.Chunks`
- [ ] Each chunk has `Content` property

**File Mode**:
- [ ] upload_attachment returns temp ID
- [ ] refresh_assignment_action called
- [ ] perform_assignment_action called
- [ ] pageInstructions in BOTH refresh and submit
- [ ] REPLACE instruction for `.ContentAttachment`
- [ ] ArticleType: "file"
- [ ] Supported file format

**Happy Path**: [05-author-content.md](../05-author-content.md)
