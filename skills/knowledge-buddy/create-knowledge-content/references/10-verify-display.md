# Step 10: Verify and Display

## Purpose

Retrieve final case summary and details. Summarize the information and display to user including case ID and status.

## MCP Tool Calls

### Call 1: Get Case Summary

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<from-step-1>",  // e.g., "PEGAFW-KB-WORK-ARTICLE KB-2020"
  viewID: "pyCaseSummary"
})
```

Extract from response:
- **caseID**: Case identifier
- **status**: Case status (e.g., "Resolved-Published", "Pending-Ingestion")
- **pyStatusWork**: Current stage

### Call 2: Get Case Details

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<from-step-1>",
  viewID: "pyDetailsTabContent"
})
```

Extract from response:
- **Title**: Article title
- **Abstract**: Article summary
- **Collection**: Collection information (Name, pyID)
- **Datasource**: Data source information (Name, pyID)
- **ArticleType**: "text" or "file"
- **Chunks**: Content chunks (if text mode)
- **ContentAttachment**: File attachment (if file mode)
- **ContentAccessConfigurations**: Access roles
- **IndexParams**: Chunking settings (if advanced mode)

## Success Criteria

Verify the following before displaying to user:

✓ **Status**: "Resolved-Published" (complete) or "Pending-Ingestion" (processing)
✓ **Collection**: CollectionName populated
✓ **Data Source**: DataSourceName populated
✓ **Content**: Title and Abstract present
✓ **Access Roles**: ContentAccessConfigurations has at least one role
✓ **Text Mode**: Chunks page list has content
✓ **File Mode**: ContentAttachment populated with file ID

## Display to User

Provide a clear summary with key information:

**Example for Text Content**:
```
✅ Successfully created knowledge base article!

Case ID: KB-2020
Status: Resolved-Published
Title: Understanding Pega DX API
Abstract: A comprehensive guide to using the Pega DX API
Collection: knowledge (DC-1)
Data Source: Knowledge_ingestion_test (SRC-1)
Content Type: Text
Chunks: 2 content chunks ingested
Access Roles: KnowledgeBuddy:Admin, KnowledgeBuddy:Author

The article is now indexed and available in the knowledge base.
```

**Example for File Content**:
```
✅ Successfully created knowledge base article!

Case ID: KB-2022
Status: Resolved-Published
Title: Company Policy Document
Abstract: Internal policies for employee handbook
Collection: documentation (DC-2)
Data Source: Corporate_docs (SRC-5)
Content Type: File
File: policy-handbook.pdf
Access Roles: KnowledgeBuddy:Internal

The document has been processed and indexed in the knowledge base.
```

## Ingestion Status

- **Resolved-Published**: Content successfully ingested and available for search
- **Pending-Ingestion**: Content is being processed (indexing in progress)

If status is "Pending-Ingestion", inform user that processing may take a few moments.

## Workflow Complete

Congratulate user on successful content creation. The content is now:
- Indexed in the knowledge base vector store
- Available for semantic search
- Accessible based on configured access roles
- Ready for use in Knowledge Buddy applications

## Next Steps for User

Suggest possible next actions:
- Create additional content articles
- Query the knowledge base to test the new content
- Configure additional data sources or collections
- Review content in the Knowledge Buddy portal
