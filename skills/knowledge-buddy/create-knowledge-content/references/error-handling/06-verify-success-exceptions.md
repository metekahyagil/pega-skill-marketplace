# Step 6 Exceptions: Verification and Ingestion Failures

This document covers error handling and troubleshooting for Step 6 (Verify Success).

**Happy Path**: [06-verify-success.md](../06-verify-success.md)

---

## 1. Ingestion Pending (Extended Time)

### Problem

Case remains in "Pending-Ingestion" status for extended time.

### Symptoms

- Status: "Pending-Ingestion" for more than 10-15 minutes
- IndexStatus not showing "Completed"
- Content not appearing in knowledge base search

### Understanding Ingestion

**Normal Behavior**:
- Content is being processed and indexed
- Vector embeddings are being generated
- May take several minutes for large content or many chunks
- **This is expected immediately after Step 5**

**When to Investigate**:
- Still pending after 10-15 minutes
- IngestionErrorMessage field appears
- IndexStatus shows error state

---

### Solutions

#### ✅ Check system logs for ingestion errors

**In Pega**:
1. Dev Studio → System → Logs
2. Filter for errors related to ingestion
3. Look for exceptions in vector store operations
4. Check for memory or resource issues

#### ✅ Check IngestionErrorMessage field

```javascript
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });

if (caseData.IngestionErrorMessage) {
  console.log("Ingestion Error:", caseData.IngestionErrorMessage);
}
```

**Common Error Messages**:
- "Implementation resulted in an exception" - System error during ingestion
- "Extraction failed" - File content extraction failed
- "Vector store unavailable" - Cannot connect to vector database
- "Embedding generation failed" - AI service error

#### ✅ Verify content format is supported

**Supported Formats**:
- ✅ PDF (.pdf) - fully supported
- ✅ DOCX (.docx) - fully supported
- ✅ TXT (.txt) - fully supported

**Problematic Formats**:
- ⚠️ MD (.md) - Markdown may have extraction issues
- ⚠️ Other text formats - varies by format

**For File Mode**:
- Check file is not corrupted
- Verify file opens correctly outside Pega
- Ensure proper encoding (UTF-8 for text files)
- Try with a smaller or simpler file

**For Text Mode**:
- Check chunk content for special characters
- Verify total content size is reasonable
- Ensure chunks are properly formatted

#### ✅ Wait and check again

Ingestion may still be processing:
- Large files take longer
- Multiple chunks take longer
- System load affects processing time
- Network latency to vector store

**Retry Check**:
```javascript
// Wait 2 minutes and check again
await new Promise(resolve => setTimeout(resolve, 120000));

const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });
console.log("Status:", caseData.pyStatusWork);
console.log("IndexStatus:", caseData.IndexStatus);
```

---

## 2. Ingestion Failed

### Problem

Status shows "Resolved-IngestionFailed".

### Symptoms

- Status: "Resolved-IngestionFailed"
- IngestionErrorMessage contains error details
- Content not indexed in vector store
- Content not searchable in knowledge base

### Solutions

#### ✅ Check IngestionErrorMessage for specific error

```javascript
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });

console.log("Status:", caseData.pyStatusWork);
console.log("Error:", caseData.IngestionErrorMessage);
console.log("IndexStatus:", caseData.IndexStatus);
```

Example errors and solutions:

**"Implementation resulted in an exception"**:
- System error during ingestion process
- Check Pega logs for stack trace
- May be temporary - try re-ingestion
- Contact Pega administrator if persistent

**"Extraction failed" or "Cannot extract content"**:
- File format issue (see file format solutions below)
- Corrupted file
- Unsupported file encoding
- File too large or complex

**"Vector store connection failed"**:
- Vector database unavailable
- Network connectivity issue
- Configuration problem with vector store
- Check Pega administrator settings

**"Embedding generation failed"**:
- AI service unavailable
- API key or authentication issue
- Rate limiting on embedding service
- Model configuration problem

#### ✅ Verify file format and integrity (File Mode)

**Check File**:
1. Open file manually to verify not corrupted
2. Check file size is reasonable (not too large)
3. Verify file format matches extension
4. Try re-saving file in supported format

**File-Specific Issues**:

**PDF**:
- Ensure not password protected
- Check not scanned images (OCR may be needed)
- Verify text layer exists in PDF
- Try re-exporting PDF with text layer

**DOCX**:
- Ensure proper Word format (not older .doc)
- Check for embedded objects that may cause issues
- Verify no macro security blocks
- Try saving as new DOCX

**TXT**:
- Verify UTF-8 encoding
- Check for invalid characters
- Ensure proper line endings
- Remove any binary data

**Markdown (.md)**:
- Known to have extraction issues
- Consider converting to PDF or DOCX
- Or use text mode instead of file mode

#### ✅ Use "Re-ingest content" action

If ingestion failed, Pega may provide a "Re-ingest content" action:

```javascript
// Check for available actions on the case
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });

// If re-ingest action available, execute it
// (Action name may vary by Pega version)
```

**When to Re-ingest**:
- After fixing file issues
- After temporary system errors
- After vector store becomes available
- After configuration changes

#### ✅ Recreate content with different format

If re-ingestion fails repeatedly:

**Option 1**: Use text mode instead of file mode
- Manually extract text from file
- Create new case using text mode
- More control over content format

**Option 2**: Convert file to different format
- PDF → DOCX or TXT
- DOCX → PDF or TXT
- MD → PDF or DOCX

---

## 3. Partial Ingestion Success

### Problem

Workflow completes (reaches Ingestion stage) but extraction/indexing fails.

### Symptoms

- Case reaches Ingestion stage successfully
- File attachment process worked correctly
- But IngestionErrorMessage shows extraction failure
- Or IndexStatus shows incomplete/error

### Understanding

This indicates:
- ✅ Steps 1-5 completed correctly
- ✅ File was uploaded and attached successfully
- ✅ Case progressed through workflow stages
- ❌ Content extraction or indexing failed

**This is NOT a workflow failure** - it's an extraction/indexing issue.

### Solutions

Follow solutions in "Ingestion Failed" section above. The workflow itself succeeded; focus on file format and extraction issues.

---

## 4. Empty or Missing Fields

### Problem

Required fields are empty after all steps complete.

### Symptoms

- CollectionName is empty
- DataSourceName is empty
- Chunks is empty (text mode)
- ContentAttachment is empty (file mode)
- Title or Abstract missing

### Solutions

#### Empty Collection/Datasource

This means Step 4 failed. See:
- **[04-configure-case-exceptions.md](./04-configure-case-exceptions.md)** - Configuration errors

#### Empty Chunks (Text Mode)

This means Step 5 failed for text content. See:
- **[05-author-content-exceptions.md](./05-author-content-exceptions.md)** - Section "Empty Chunks"

#### Empty ContentAttachment (File Mode)

This means Step 5 failed for file content. See:
- **[05-author-content-exceptions.md](./05-author-content-exceptions.md)** - Section "File Upload Failures"

#### Missing Title or Abstract

These are scalar fields set in Step 5 content parameter:

```javascript
content: {
  Title: "Article Title",       // Required
  Abstract: "Article summary"    // Required
}
```

If missing, Step 5 may have failed or been incomplete.

---

## 5. Case Not Found

### Problem

Cannot retrieve case data in Step 6.

### Symptoms

- "Case not found" error
- Invalid case ID
- Access denied to case

### Solutions

#### ✅ Verify case ID is correct

```javascript
// Case ID from Step 3
const caseID = createResponse.caseID;  // e.g., "KB-1023"

// Use exact case ID
const caseData = await mcp__pega-dx-mcp__get_case({ caseID });
```

**Common Issues**:
- Typo in case ID
- Using assignment ID instead of case ID
- Case ID from different case/session

#### ✅ Check case permissions

- User may not have access to view the case
- Access group restrictions
- Case may be in another access group

#### ✅ Verify case exists in Pega

Log into Pega and search for the case ID manually to confirm it exists.

---

## 6. Verification View Failures

### Problem

Cannot retrieve case views (Create, AuthorContent) for detailed verification.

### Symptoms

- `get_case_view` fails
- View not found error
- View returns empty data

### Solutions

#### ✅ Use correct view IDs

```javascript
// For configuration details
await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "Create"  // Exact case-sensitive view ID
});

// For content details
await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"  // Exact case-sensitive view ID
});
```

#### ✅ Use get_case instead

If views fail, use basic `get_case` which returns high-level data:

```javascript
const caseData = await mcp__pega-dx-mcp__get_case({
  caseID: "KB-1023"
});

// Check key fields
console.log("Status:", caseData.pyStatusWork);
console.log("Collection:", caseData.CollectionName);
console.log("Data Source:", caseData.DataSourceName);
```

---

## 7. Content Not Searchable

### Problem

Ingestion succeeded but content doesn't appear in knowledge base search.

### Symptoms

- Status: "Resolved-Published"
- IndexStatus: "Completed"
- But searching in knowledge base returns no results

### Solutions

#### ✅ Wait for search index refresh

Vector store and search indices may need time to refresh:
- Wait 5-10 minutes after ingestion completes
- Search indices updated periodically
- Try search again after waiting

#### ✅ Verify search query

- Search terms must match content
- Try searching for unique terms from Title or Abstract
- Check search is looking in correct collection

#### ✅ Check access permissions

- Content has access role restrictions
- User performing search must have required role
- Verify ContentAccessConfigurations includes appropriate roles

#### ✅ Verify collection configuration

- Collection must be properly configured in Pega
- Vector store connection must be active
- Search must be enabled for the collection

---

## Troubleshooting Checklist

When verification or ingestion fails:

### Basic Verification
- [ ] Case ID is correct and case exists
- [ ] Using `get_case` to retrieve basic status
- [ ] Status is "Resolved-Published" or "Pending-Ingestion"
- [ ] No IngestionErrorMessage present

### Configuration Verification
- [ ] CollectionName populated (not empty)
- [ ] DataSourceName populated (not empty)
- [ ] VectorStoreCollectionName matches collection
- [ ] ContentAccessConfigurations populated

### Content Verification (Text Mode)
- [ ] Title present
- [ ] Abstract present
- [ ] Chunks populated (shows as [object Object]...)
- [ ] ArticleType is "text"

### Content Verification (File Mode)
- [ ] Title present
- [ ] Abstract present
- [ ] ContentAttachment populated (shows as [object Object])
- [ ] ArticleType is "file"
- [ ] File attachment exists in attachments list
- [ ] ExtractionMethod is "Standard"

### Ingestion Status
- [ ] Not stuck in "Pending-Ingestion" for extended time
- [ ] No IngestionErrorMessage
- [ ] IndexStatus shows "Completed"
- [ ] If failed, checked file format and integrity

---

## What to Do If All Checks Pass

If verification shows all data is correct but content isn't working:

1. **Wait and Retry**:
   - Ingestion may still be processing
   - Search indices may be updating
   - Wait 10-15 minutes and check again

2. **Check Pega System Health**:
   - Vector store connectivity
   - AI service availability
   - System resource usage
   - Network connectivity

3. **Review Pega Logs**:
   - Application logs for exceptions
   - Tracer for workflow issues
   - Vector store logs for indexing errors

4. **Test Search Manually**:
   - Log into knowledge base portal
   - Search for content manually
   - Verify content appears in UI

5. **Contact Administrator**:
   - Provide case ID
   - Share IngestionErrorMessage if present
   - Include steps already tried
   - Request review of vector store configuration

---

## Success Criteria Reference

For complete verification checklist, see:
- **[success-criteria.md](../success-criteria.md)** - Complete success criteria and verification methods

---

## Related Documentation

- **Happy Path**: [06-verify-success.md](../06-verify-success.md)
- **Configuration Issues**: [04-configure-case-exceptions.md](./04-configure-case-exceptions.md)
- **Content Issues**: [05-author-content-exceptions.md](./05-author-content-exceptions.md)
- **Success Criteria**: [success-criteria.md](../success-criteria.md)
- **Error Handling Index**: [index.md](./index.md)
