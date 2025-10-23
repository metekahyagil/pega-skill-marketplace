# Success Criteria

This document provides a comprehensive checklist to verify successful knowledge base content creation.

## Complete Success Checklist

A successful content creation includes all of the following:

### ✅ Basic Case Creation
- [ ] Case created with valid ID (e.g., KB-1023)
- [ ] Case type: PegaFW-KB-Work-Article
- [ ] Initial assignment created

### ✅ Collection Association
- [ ] CollectionName populated (e.g., "knowledge")
- [ ] VectorStoreCollectionName populated (matches CollectionName)
- [ ] Collection embedded page contains pyID and pzInsKey

### ✅ Data Source Association
- [ ] DataSourceName populated (e.g., "Knowledge_ingestion_test")
- [ ] Datasource.pyID populated (e.g., "SRC-1001")
- [ ] Datasource.pzInsKey populated (e.g., "PEGAFW-QNA-WORK SRC-1001")

### ✅ Access Configuration
- [ ] ContentAccessConfigurations page list populated
- [ ] At least one access role configured (e.g., "Knowledge Buddy Public")
- [ ] Access roles visible as `[object Object]` in case view

### ✅ Chunking Parameters (If Advanced Settings Enabled)
- [ ] IndexParams embedded page populated
- [ ] ChunkingMethod set (TITLE, SIZE, NONE, or ABSTRACT)
- [ ] ChunkSize configured (integer value)
- [ ] ChunkOverlap configured (integer value)

### ✅ Content Authored - Text Mode

**If ArticleType = "text"**:
- [ ] Title present (stored in pyLabel)
- [ ] Abstract present
- [ ] ArticleType field set to "text"
- [ ] Chunks page list populated with Content objects
- [ ] Each chunk object contains a "Content" property
- [ ] Chunks visible as `[object Object],[object Object]...` in AuthorContent view
- [ ] Number of chunks matches user specification

### ✅ Content Authored - File Mode

**If ArticleType = "file"**:
- [ ] Title present (stored in pyLabel)
- [ ] Abstract present
- [ ] ArticleType field set to "file"
- [ ] ContentAttachment embedded page populated
- [ ] ContentAttachment visible as `[object Object]` in AuthorContent view
- [ ] pyAttachName contains filename (e.g., "policy-document.pdf")
- [ ] ExtractionMethod set to "Standard"
- [ ] Case has file attachment(s) in attachments list
- [ ] File attachment type is DATA-WORKATTACH-FILE (not LINK-ATTACHMENT)

### ✅ Case Status and Ingestion
- [ ] Status: "Resolved-Published" (or "Pending-Ingestion" if still processing)
- [ ] IndexStatus.status: "Completed" (when successful)
- [ ] No IngestionErrorMessage present
- [ ] Case reached Ingestion stage (ALT2) and then Resolve stage (PRIM3)

### ✅ Content Accessibility
- [ ] Content URL available (generated after publishing)
- [ ] Content accessible in knowledge base portal
- [ ] Content appears in search results (if tested)

---

## Verification Methods

### Method 1: Basic Case Verification

Use `get_case` to check high-level status:

```javascript
const caseData = await mcp__pega-dx-mcp__get_case({
  caseID: "KB-1023"
});

// Verify these fields:
console.log("Status:", caseData.pyStatusWork);
console.log("Collection:", caseData.CollectionName);
console.log("Data Source:", caseData.DataSourceName);
console.log("Title:", caseData.pyLabel);
console.log("Abstract:", caseData.Abstract);
console.log("Article Type:", caseData.ArticleType);
```

### Method 2: Detailed Configuration View

Use `get_case_view` with "Create" viewID to see configuration:

```javascript
const createView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "Create"
});

// Shows:
// - Collection (embedded page)
// - Datasource (embedded page)
// - IndexParams (if advanced settings)
// - ContentAccessConfigurations (page list)
```

### Method 3: Detailed Content View

Use `get_case_view` with "AuthorContent" viewID to see content:

```javascript
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"
});

// For text mode, shows:
// - Title
// - Abstract
// - Chunks as [object Object],[object Object]...

// For file mode, shows:
// - Title
// - Abstract
// - ContentAttachment as [object Object]
```

### Method 4: Attachments View (File Mode Only)

Use `get_case_attachments` to see attached files:

```javascript
const attachments = await mcp__pega-dx-mcp__get_case_attachments({
  caseID: "KB-1026"
});

// Verify:
// - At least one attachment present
// - Attachment has correct filename
// - Attachment type is DATA-WORKATTACH-FILE
```

---

## Verification Tips by Content Type

### Text Content Verification

1. **Check Chunks structure**:
   - Use `get_case_view` with "AuthorContent" viewID
   - Chunks should show as `[object Object],[object Object]...`
   - Each object contains a "Content" property

2. **Verify chunk count**:
   - Number of `[object Object]` entries matches number of chunks submitted

3. **Check ArticleType**:
   - Should be "text"

### File Content Verification

1. **Check ContentAttachment**:
   - Use `get_case_view` with "AuthorContent" viewID
   - Should show as `[object Object]`
   - Contains pyAttachName with filename

2. **Verify file attachment**:
   - Use `get_case_attachments`
   - Should list uploaded file
   - Attachment type should be DATA-WORKATTACH-FILE

3. **Check ArticleType**:
   - Should be "file"

4. **Check ExtractionMethod**:
   - Should be "Standard"

### Access Configuration Verification

1. **Check ContentAccessConfigurations**:
   - Use `get_case_view` with "Create" viewID
   - Should show as `[object Object]` for each role
   - Verify role names are correct

2. **Test access**:
   - Log in with configured role
   - Verify content is accessible in knowledge base

---

## Common Issues and Quick Fixes

### Issue: Empty Collection/Datasource

**Quick Check**:
```javascript
console.log(caseData.CollectionName);  // Should not be empty
console.log(caseData.DataSourceName);  // Should not be empty
```

**Fix**: See [error-handling.md](./error-handling.md) #1

### Issue: Empty Chunks (Text Mode)

**Quick Check**:
```javascript
const view = await get_case_view({ caseID: "...", viewID: "AuthorContent" });
console.log(view.Chunks);  // Should show [object Object]...
```

**Fix**: See [error-handling.md](./error-handling.md) #5

### Issue: Empty ContentAttachment (File Mode)

**Quick Check**:
```javascript
const view = await get_case_view({ caseID: "...", viewID: "AuthorContent" });
console.log(view.ContentAttachment);  // Should show [object Object]
```

**Fix**: See [error-handling.md](./error-handling.md) #7

### Issue: Ingestion Failed

**Quick Check**:
```javascript
console.log(caseData.pyStatusWork);  // Should be "Resolved-Published"
console.log(caseData.IngestionErrorMessage);  // Should be empty
```

**Fix**: See [error-handling.md](./error-handling.md) #8

---

## Reporting Success to User

After verifying all criteria, report success with:

### Minimum Information
- ✅ Success confirmation
- Case ID
- Title
- Status

### Recommended Information
- ✅ Success confirmation
- Case ID
- Title
- Collection name
- Data source name
- Status
- Content type (text or file)
- Number of chunks (text) or filename (file)

### Complete Information
- ✅ Success confirmation
- Case ID
- Title
- Abstract
- Collection name
- Data source name
- Status
- Content type
- Number of chunks or filename
- Content URL (if available)
- Access roles configured
- Chunking parameters (if advanced)

### Example Success Message

```
✅ Successfully created knowledge base article!

Case ID: KB-1023
Title: Understanding Pega DX API
Abstract: A comprehensive guide to using the Pega DX API

Configuration:
- Collection: knowledge
- Data Source: Knowledge_ingestion_test
- Access Role: Knowledge Buddy Public

Content:
- Type: Text
- Chunks: 3 content chunks ingested

Status: Resolved-Published
IndexStatus: Completed

The article is now available in the knowledge base.
```

---

## Troubleshooting

If any criteria are not met:

1. **Review step documentation**:
   - Check the relevant step reference (00-connection.md through 06-verify-success.md)
   - Verify all parameters are correct

2. **Check error handling**:
   - See [error-handling.md](./error-handling.md) for specific issue
   - Follow debug checklist

3. **Review technical details**:
   - See [technical-details.md](./technical-details.md) for patterns
   - Verify page instructions format

4. **Check examples**:
   - See [examples.md](./examples.md) for working code
   - Compare with your implementation

---

## Related References

- **[06-verify-success.md](./06-verify-success.md)** - Step-by-step verification workflow
- **[error-handling.md](./error-handling.md)** - Troubleshooting common issues
- **[technical-details.md](./technical-details.md)** - Technical patterns and structures
- **[examples.md](./examples.md)** - Complete working examples
