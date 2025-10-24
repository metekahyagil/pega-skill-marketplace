---
name: create-knowledge-content
description: Simplified workflow for creating knowledge base content articles in Pega Knowledge Buddy applications. Use this skill when the user wants to create, author, and ingest content into a knowledge base with collection, data source, access configuration, and chunking settings. Handles the complete end-to-end workflow from case creation through content ingestion with support for multiple content chunks and access roles.
---

# Create Knowledge Content

## Overview

This skill provides a streamlined workflow for creating knowledge base content articles in Pega Knowledge Buddy applications. It automates the multi-step process of creating a Content case, configuring collection and data source associations, setting access roles, configuring chunking parameters, authoring content (text or file), and submitting for ingestion into the knowledge base vector store.

The workflow supports two content formats:
- **Text**: Manual text entry with multiple chunks
- **File**: Document upload (PDF, DOCX, TXT) with automatic extraction

And two configuration modes:
- **Simple**: Uses default chunking settings (recommended for most cases)
- **Advanced**: Custom chunking parameters for specialized use cases

## When to Use This Skill

- Creating new knowledge base articles or content
- Adding content to existing collections and data sources
- Ingesting documentation, policies, or reference materials into a knowledge base
- Uploading documents for automatic text extraction and indexing
- User requests like "Create a new article about X", "Add content to the knowledge base", or "Upload this document to the knowledge base"

## Workflow

### Step 0: Verify Pega Connection

Authenticate and verify connection to Pega before starting the workflow.

**Reference**: [00-connection.md](references/00-connection.md)

**Key Action**: Call `mcp__pega-dx-mcp__authenticate_pega({})` to validate OAuth2 authentication and server connectivity.

---

### Step 1: Query Available Data

Discover existing collections and data sources in the system to present options to the user.

**Reference**: [01-query-data.md](references/01-query-data.md)

**Key Actions**:
- Query collections using `get_list_data_view` with `D_IndexList`
- Query data sources using `get_list_data_view` with `D_DataSourceList`
- Capture CollectionName, pyID, and pzInsKey for each option

---

### Step 2: Gather User Input

Collect user preferences for settings mode (simple/advanced), content format (text/file), and content details using the `AskUserQuestion` tool.

**Reference**: [02-user-input.md](references/02-user-input.md)

**Key Decisions**:
- **Settings Mode**: Simple (default chunking) or Advanced (custom chunking)
- **Content Format**: Text (manual entry) or File (document upload)
- **Content Details**: Title, abstract, and content (text chunks or file path)

---

### Step 3: Create Content Case

Create a new Content case using the `PegaFW-KB-Work-Article` case type.

**Reference**: [03-create-case.md](references/03-create-case.md)

**Key Action**: Call `mcp__pega-dx-mcp__create_case` with case type ID and empty content. Capture caseID and assignmentID for next steps.

---

### Step 4: Configure Collection, Data Source, and Access

Perform the Create action using pageInstructions to associate collection, data source, access roles, and optionally chunking parameters.

**Reference**: [04-configure-case.md](references/04-configure-case.md)

**Key Actions**:
- Use UPDATE pageInstruction for `.Collection` and `.Datasource` embedded pages
- Include `.IndexParams` UPDATE if Advanced settings selected
- Use APPEND pageInstruction for `.ContentAccessConfigurations` page list
- All target paths must start with a dot (e.g., `.Collection`)
- pzInsKey format: `"CLASS-NAME ID"` with single space

**Critical**: Embedded pages MUST be set via pageInstructions, not content properties.

---

### Step 5: Author Content

Author the article content based on the selected format (text or file).

**Reference**: [05-author-content.md](references/05-author-content.md)

**Option A - Text Content**:
- Set Title and Abstract in content parameter
- Set ArticleType: "text"
- Use APPEND pageInstructions for `.Chunks` page list
- Each chunk object has a "Content" property

**Option B - File Content** (3-step process):
1. Upload file using `upload_attachment` to get temporary attachment ID
2. Refresh assignment using `refresh_assignment_action` with pageInstructions
3. Submit using `perform_assignment_action` with same pageInstructions
- Set ArticleType: "file"
- Use REPLACE pageInstruction for `.ContentAttachment`
- Must include pageInstructions in both refresh AND submit

---

### Step 6: Verify Success

Retrieve final case status to confirm successful ingestion and report results to user.

**Reference**: [06-verify-success.md](references/06-verify-success.md)

**Key Actions**:
- Use `get_case` to retrieve final case status
- Verify status is "Resolved-Published" (or "Pending-Ingestion" if processing)
- Check CollectionName, DataSourceName, Title, and Abstract are populated
- For text: Verify Chunks page list populated
- For file: Verify ContentAttachment and file attachment present
- Report case ID and content URL to user

---

## Technical References

### Core Patterns and Structures
- **[technical-details.md](references/technical-details.md)** - Case types, assignment actions, pageInstructions patterns, content formats, data structures, and MCP tools reference

### Configuration Modes
- **[default-settings.md](references/default-settings.md)** - Simple vs advanced settings, chunking methods (TITLE, SIZE, NONE, ABSTRACT), chunk size and overlap parameters, and configuration examples

### Error Handling
- **[error-handling/index.md](references/error-handling/index.md)** - Step-by-step error handling for each workflow stage, including connection failures, query issues, input validation, configuration errors, and ingestion problems

### Verification
- **[success-criteria.md](references/success-criteria.md)** - Complete verification checklist for case creation, configuration, content authoring, and ingestion status

### Working Examples
- **[examples.md](references/examples.md)** - Complete working examples with actual values for text content (advanced settings), file content (simple settings), and minimal single-chunk workflow

---

## Quick Reference

### PageInstructions Critical Rules

1. **Target paths MUST start with dot**: `.Collection` not `Collection`
2. **Embedded pages use UPDATE**: Collection, Datasource, IndexParams
3. **Page lists use APPEND**: Chunks, ContentAccessConfigurations
4. **pzInsKey format**: `"PEGAFW-QNA-WORK DC-1"` (with space)
5. **Complete references required**: Always include name, pyID, and pzInsKey

### Content Format Patterns

**Text Mode**:
```javascript
content: { Title: "...", Abstract: "...", ArticleType: "text" }
pageInstructions: [
  { instruction: "APPEND", target: ".Chunks", content: { Content: "..." } }
]
```

**File Mode**:
```javascript
// 1. Upload → 2. Refresh → 3. Submit
pageInstructions: [
  { instruction: "REPLACE", target: ".ContentAttachment", content: { ID: tempAttachmentID } }
]
```

### Common Values

- **Case Type**: `PegaFW-KB-Work-Article`
- **Collection Class**: `PEGAFW-QNA-WORK`
- **Default Access Role**: `KnowledgeBuddy:Public`
- **Default Chunking**: SIZE method, 1000 size, 200 overlap

---

## Next Steps After Content Creation

After successful content creation:
- Content is indexed in vector store
- Available for semantic search in knowledge base
- Accessible via knowledge base portal
- Can be updated or re-ingested if needed

For troubleshooting, error handling, or detailed technical information, refer to the comprehensive reference documents listed above.
