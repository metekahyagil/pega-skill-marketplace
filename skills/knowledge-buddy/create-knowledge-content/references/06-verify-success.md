# Step 6: Verify Success

After authoring content, verify that the content was successfully ingested into the knowledge base.

## Quick Checklist

- [ ] Call `get_case` with caseID to retrieve case status
- [ ] Verify status is "Resolved-Published" (or "Pending-Ingestion")
- [ ] Verify CollectionName is populated
- [ ] Verify DataSourceName is populated
- [ ] For text mode: Verify Chunks populated
- [ ] For file mode: Verify ContentAttachment and file attachment present
- [ ] Check IndexStatus.status is "Completed"
- [ ] Report success with case ID and content URL to user

---

## Table of Contents

- [Retrieve Case Status](#retrieve-case-status)
- [What to Verify](#what-to-verify)
  - [Case Status](#case-status)
  - [Collection and Data Source](#collection-and-data-source)
  - [Content Data](#content-data)
  - [Access Configuration](#access-configuration)
- [Detailed Content Verification](#detailed-content-verification)
  - [View Configuration (Create Action)](#view-configuration-create-action)
  - [View Content (AuthorContent Action)](#view-content-authorcontent-action)
  - [View Attachments (File Mode Only)](#view-attachments-file-mode-only)
- [Success Criteria](#success-criteria)
- [Example Verification](#example-verification)
- [Ingestion Status](#ingestion-status)
- [Content URL](#content-url)
- [Report to User](#report-to-user)
- [Complete Workflow](#complete-workflow)

---

## Retrieve Case Status

Use `get_case` to retrieve the final case status:

```javascript
mcp__pega-dx-mcp__get_case({
  caseID: "<case-id>"
})
```

## What to Verify

Check the following fields in the response:

### Case Status
- **Status**: Should be "Resolved-Published" (or "Pending-Ingestion" if still processing)
- **IndexStatus**: Should show completion status

### Collection and Data Source
- **CollectionName**: Should match selected collection
- **DataSourceName**: Should match selected data source (with pyID and pzInsKey)
- **VectorStoreCollectionName**: Should match the collection

### Content Data
- **Title**: Should be stored (also in pyLabel)
- **Abstract**: Should be present
- **ArticleType**: "text" or "file" based on content format
- **Chunks** (for text): Contains objects (page list) with Content property
- **ContentAttachment** (for file): Contains embedded page with file metadata

### Access Configuration
- **ContentAccessConfigurations**: Should contain access role objects

## Detailed Content Verification

For more detailed verification, use `get_case_view` with specific view IDs:

### View Configuration (Create Action)

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "Create"
})
```

**Shows:**
- Collection (embedded page)
- Datasource (embedded page)
- IndexParams (chunking configuration)
- ContentAccessConfigurations (page list)

### View Content (AuthorContent Action)

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "AuthorContent"
})
```

**Shows:**
- Title
- Abstract
- ArticleType
- Chunks (for text mode) - displays as `[object Object],[object Object]...`
- ContentAttachment (for file mode) - displays as `[object Object]`

### View Attachments (File Mode Only)

```javascript
mcp__pega-dx-mcp__get_case_attachments({
  caseID: "<case-id>"
})
```

**Shows:** List of attached files (for file mode)

## Success Criteria

A successful content creation includes:

### ✅ Basic Verification
- Case created with valid ID
- Status: "Resolved-Published" (or "Pending-Ingestion")
- IndexStatus.status: "Completed" (when successful)

### ✅ Configuration Verification
- Collection associated (CollectionName populated)
- Data Source associated (DataSourceName with pyID and pzInsKey)
- VectorStoreCollectionName matches collection
- ContentAccessConfigurations page list populated

### ✅ Content Verification (Text Mode)
- Title present (stored in pyLabel)
- Abstract present
- Chunks page list populated with Content objects
- ArticleType: "text"

### ✅ Content Verification (File Mode)
- Title present (stored in pyLabel)
- Abstract present
- ContentAttachment embedded page populated
- ArticleType: "file"
- ExtractionMethod: "Standard"
- Case has file attachment(s)

### ✅ Advanced Settings Verification (If Enabled)
- IndexParams populated with custom values
- ChunkingMethod, ChunkSize, ChunkOverlap set correctly

## Example Verification

```javascript
// Get case status
const caseData = await mcp__pega-dx-mcp__get_case({
  caseID: "KB-1023"
});

// Verify key fields
console.log("Status:", caseData.pyStatusWork);  // "Resolved-Published"
console.log("Collection:", caseData.CollectionName);  // "knowledge"
console.log("Data Source:", caseData.DataSourceName);  // "Knowledge_ingestion_test"
console.log("Title:", caseData.pyLabel);  // "Understanding Pega DX API"
console.log("Abstract:", caseData.Abstract);  // "A comprehensive guide..."

// For detailed view
const authorView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "AuthorContent"
});

// Verify content structure
console.log("Chunks:", authorView.Chunks);  // [object Object],[object Object]...
```

## Ingestion Status

The content goes through these states:

1. **Pending-Ingestion**: Content is being processed and indexed
2. **Resolved-Published**: Content successfully ingested and available in knowledge base

## Content URL

After successful publishing, the case will have a content URL for accessing the article in the knowledge base portal.

## Report to User

Provide the user with:
- ✅ Success confirmation
- Case ID (e.g., "KB-1023")
- Content URL (if available)
- Brief summary of what was created

**Example:**
```
✅ Successfully created knowledge base article!

Case ID: KB-1023
Title: Understanding Pega DX API
Collection: knowledge
Data Source: Knowledge_ingestion_test
Status: Resolved-Published
Chunks: 3 content chunks ingested

The article is now available in the knowledge base.
```

---

**Exceptions**: See **[error-handling/06-verify-success-exceptions.md](error-handling/06-verify-success-exceptions.md)** for verification failures, ingestion issues, and troubleshooting.

**Success Criteria**: See **[success-criteria.md](./success-criteria.md)** for complete verification checklist.

## Complete Workflow

This completes the end-to-end workflow:
1. ✅ Step 0: Verify Pega Connection
2. ✅ Step 1: Query Available Data
3. ✅ Step 2: Gather User Input
4. ✅ Step 3: Create Content Case
5. ✅ Step 4: Configure Collection, Data Source, and Access
6. ✅ Step 5: Author Content
7. ✅ **Step 6: Verify Success** ← You are here

The content is now ingested into the Pega Knowledge Buddy knowledge base!
