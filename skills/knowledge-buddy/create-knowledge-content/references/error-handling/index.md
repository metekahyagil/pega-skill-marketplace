# Error Handling and Troubleshooting

This document covers common issues and their solutions when creating knowledge base content.

## 1. Empty Collection/Datasource

**Problem**: Collection or Datasource fields remain empty after Create action.

**Symptoms**:
- CollectionName is empty or undefined
- DataSourceName is empty or undefined
- Fields not populated in case data

**Solutions**:

✅ **Verify pageInstructions are used** (not content properties)
- Embedded pages MUST be set via pageInstructions
- Content properties do not work for Collection/Datasource

✅ **Check target paths start with a dot**
- Correct: `.Collection`, `.Datasource`
- Incorrect: `Collection`, `Datasource`

✅ **Use UPDATE instruction** (not REPLACE)
- UPDATE preserves existing page structure
- REPLACE removes all fields

✅ **Verify pzInsKey format is correct**
- Must be: `"CLASS-NAME ID"` with a space
- Example: `"PEGAFW-QNA-WORK DC-1"`
- Not: `"PEGAFW-QNA-WORKDC-1"` (missing space)

✅ **All required fields are provided**

For Collection:
```javascript
{
  CollectionName: "knowledge",
  pyID: "DC-1",
  pzInsKey: "PEGAFW-QNA-WORK DC-1"
}
```

For Datasource:
```javascript
{
  Name: "Knowledge_ingestion_test",
  pyID: "SRC-1001",
  pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
}
```

---

## 2. BAD_REQUEST Error (400)

**Problem**: HTTP 400 error when calling `perform_assignment_action`.

**Symptoms**:
- Request fails with "BAD_REQUEST" or 400 status code
- Error message may mention invalid parameters or structure

**Common Causes**:

❌ **Missing dot prefix in target path**
```javascript
// Wrong
{ target: "Collection" }

// Correct
{ target: ".Collection" }
```

❌ **Trying to set embedded pages via content instead of pageInstructions**
```javascript
// Wrong
content: {
  Collection: { CollectionName: "knowledge" }
}

// Correct
pageInstructions: [{
  instruction: "UPDATE",
  target: ".Collection",
  content: { CollectionName: "knowledge", pyID: "DC-1", pzInsKey: "PEGAFW-QNA-WORK DC-1" }
}]
```

❌ **Incorrect pzInsKey format**
```javascript
// Wrong - missing space
pzInsKey: "PEGAFW-QNA-WORKDC-1"

// Correct
pzInsKey: "PEGAFW-QNA-WORK DC-1"
```

❌ **Missing required fields (pyID or pzInsKey)**
```javascript
// Wrong - incomplete
{ CollectionName: "knowledge" }

// Correct - all fields
{ CollectionName: "knowledge", pyID: "DC-1", pzInsKey: "PEGAFW-QNA-WORK DC-1" }
```

---

## 3. Case Creation Fails

**Problem**: Cannot create Content case.

**Symptoms**:
- `create_case` call fails
- Error about permissions or case type

**Solutions**:

✅ **Verify authentication**
- Run `authenticate_pega` first (Step 0)
- Check OAuth2 credentials are valid
- Ensure access token is obtained

✅ **Check permissions**
- User must have permission to create cases
- Verify access to PegaFW-KB-Work-Article case type

✅ **Verify case type is available**
- Case type: `PegaFW-KB-Work-Article`
- Must exist in Pega instance
- Knowledge Buddy application must be installed

**Note**: Content case accepts empty content, so creation should normally succeed.

---

## 4. Ingestion Pending

**Problem**: Case remains in "Pending-Ingestion" status for extended time.

**Symptoms**:
- Status: "Pending-Ingestion"
- IndexStatus not showing "Completed"
- Content not appearing in knowledge base

**This is Normal**:
- Content is being processed and indexed
- Vector embeddings are being generated
- May take several minutes for large content

**When to Investigate**:
- If pending for more than 10-15 minutes
- If IngestionErrorMessage field appears
- If IndexStatus shows error

**Solutions**:

✅ **Check system logs** for ingestion errors

✅ **Check IngestionErrorMessage field** in case data

✅ **Verify content format** is supported

✅ **Wait and check again** - ingestion may still be processing

---

## 5. Empty Chunks

**Problem**: Chunks remain empty after AuthorContent submission (text mode).

**Symptoms**:
- Chunks field is empty or undefined
- Content not stored in case
- No chunks visible in case view

**Solutions**:

✅ **Verify you used pageInstructions with APPEND** (not content property)
```javascript
// Wrong - using content
content: {
  Chunks: "some text"
}

// Correct - using pageInstructions
pageInstructions: [{
  instruction: "APPEND",
  target: ".Chunks",
  content: { Content: "some text" }
}]
```

✅ **Target must be `.Chunks`** (with dot prefix)
```javascript
// Wrong
{ target: "Chunks" }

// Correct
{ target: ".Chunks" }
```

✅ **Each chunk object must have "Content" property** (capital C)
```javascript
// Wrong
{ content: { text: "some text" } }

// Correct
{ content: { Content: "some text" } }
```

✅ **Setting Chunks as a string in content parameter may not persist**
- Always use pageInstructions with APPEND
- Each chunk is an object with Content property

---

## 6. Empty ContentAccessConfigurations

**Problem**: Access roles are not set after Create action.

**Symptoms**:
- ContentAccessConfigurations is empty
- No access roles configured
- Content may not be accessible

**Solutions**:

✅ **Verify you used pageInstructions with APPEND** (not content property)

✅ **Target must be `.ContentAccessConfigurations`** (with dot prefix)

✅ **Each access config object must have "AccessRoleName" property**
```javascript
{
  instruction: "APPEND",
  target: ".ContentAccessConfigurations",
  content: {
    AccessRoleName: "Knowledge Buddy Public"
  }
}
```

✅ **Setting ContentAccessConfigurations as a string in content parameter will not work**

---

## 7. File Upload Issues

**Problem**: File-based content fails to upload or attach.

**Symptoms**:
- ContentAttachment is empty or undefined
- File not attached to case
- Ingestion fails for file content
- Filename shows as "undefined"

**Solutions**:

✅ **MUST use upload → refresh → submit workflow (3 steps)**

Step-by-step:
1. Upload file: `upload_attachment`
2. Refresh: `refresh_assignment_action` with pageInstructions
3. Submit: `perform_assignment_action` with same pageInstructions

✅ **DO NOT use `add_case_attachments`**
- This creates LINK-ATTACHMENT, not ContentAttachment
- ContentAttachment is an embedded page, not a case attachment list item

✅ **MUST call `refresh_assignment_action` with pageInstructions BEFORE submit**
- Refresh initializes the ContentAttachment embedded page
- Cannot skip this step

✅ **MUST include pageInstructions in BOTH refresh AND submit calls**
```javascript
// Both calls need the same pageInstructions
pageInstructions: [{
  instruction: "REPLACE",
  target: ".ContentAttachment",
  content: { ID: tempAttachmentID }
}]
```

✅ **Use REPLACE instruction with target `.ContentAttachment`**
```javascript
{
  instruction: "REPLACE",  // Not UPDATE or APPEND
  target: ".ContentAttachment",  // With dot prefix
  content: { ID: tempAttachmentID }
}
```

✅ **Content only needs `{ID: tempAttachmentID}`**
- Filename retrieved automatically by Pega
- Don't manually specify pyAttachName unless needed

✅ **Set ArticleType to "file"** (not "text") in content parameter
```javascript
content: {
  Title: "...",
  Abstract: "...",
  ArticleType: "file"  // Critical
}
```

✅ **If ContentAttachment shows "undefined" filename**
- Check that upload included fileName and mimeType
- Verify upload_attachment returned valid ID
- Ensure ID is passed correctly to refresh and submit

---

## 8. Ingestion Failures

**Problem**: Status shows "Resolved-IngestionFailed".

**Symptoms**:
- Status: "Resolved-IngestionFailed"
- IngestionErrorMessage contains error details
- Content not indexed in vector store

**Solutions**:

✅ **Check IngestionErrorMessage field** for specific error
- Example: "Implementation resulted in an exception"
- Provides clues about what went wrong

✅ **Verify file format is supported** by extraction method

**Supported Formats**:
- ✅ PDF (.pdf)
- ✅ DOCX (.docx)
- ✅ TXT (.txt)

**May Have Issues**:
- ⚠️ MD (.md) - Markdown files may not extract correctly
- ⚠️ Other text formats

✅ **For text files, ensure proper encoding (UTF-8)**

✅ **For binary files (PDF, DOCX), ensure files are not corrupted**
- Try opening file locally to verify
- Check file size is reasonable

✅ **Use "Re-ingest content" action to retry ingestion**
- Available if extraction fails
- May succeed on retry

**Important Note**: The workflow can complete successfully (case reaches Ingestion stage) even if extraction fails. This indicates the file attachment process worked correctly, but the content extraction encountered an issue.

---

## Debug Checklist

When troubleshooting, verify:

### Step 0: Connection
- [ ] `authenticate_pega` succeeds
- [ ] Environment variables are set correctly
- [ ] Pega DX API is enabled

### Step 1: Query Data
- [ ] Collections query returns results
- [ ] Data sources query returns results
- [ ] pzInsKey values are captured correctly

### Step 2: User Input
- [ ] All required fields collected
- [ ] Settings mode determined (simple/advanced)
- [ ] Content format determined (text/file)

### Step 3: Create Case
- [ ] Case created successfully
- [ ] caseID captured
- [ ] assignmentID captured

### Step 4: Configure Case
- [ ] pageInstructions used for Collection/Datasource
- [ ] Target paths have dot prefix
- [ ] UPDATE instruction used
- [ ] pzInsKey format correct (with space)
- [ ] All required fields provided

### Step 5: Author Content

**For Text Mode**:
- [ ] ArticleType set to "text"
- [ ] Title and Abstract in content parameter
- [ ] Chunks use pageInstructions with APPEND
- [ ] Target is ".Chunks" (with dot)
- [ ] Each chunk has "Content" property

**For File Mode**:
- [ ] File uploaded successfully
- [ ] Temporary attachment ID captured
- [ ] refresh_assignment_action called with pageInstructions
- [ ] perform_assignment_action called with same pageInstructions
- [ ] Target is ".ContentAttachment" (with dot)
- [ ] REPLACE instruction used
- [ ] ArticleType set to "file"

### Step 6: Verify
- [ ] Case status checked
- [ ] Collection and Datasource populated
- [ ] Content present (Chunks or ContentAttachment)
- [ ] IndexStatus shows completion

---

## Getting Help

If issues persist:

1. **Review step-by-step references**:
   - [00-connection.md](./00-connection.md)
   - [01-query-data.md](./01-query-data.md)
   - [02-user-input.md](./02-user-input.md)
   - [03-create-case.md](./03-create-case.md)
   - [04-configure-case.md](./04-configure-case.md)
   - [05-author-content.md](./05-author-content.md)
   - [06-verify-success.md](./06-verify-success.md)

2. **Check technical references**:
   - [technical-details.md](./technical-details.md) - Core patterns and structures
   - [examples.md](./examples.md) - Working examples with actual values

3. **Verify success criteria**:
   - [success-criteria.md](./success-criteria.md) - Complete verification checklist

4. **Review Pega logs** for system-level errors

5. **Check Pega DX API documentation** for API-specific issues
