# Technical Details

This document provides technical reference information about the Pega Knowledge Buddy content creation workflow.

## Case Types Used

### 1. PegaFW-KB-Work-Article
**Purpose**: Content case type for knowledge base articles

**Stages**:
1. **Create** (PRIM0) - Initial case creation and configuration
2. **Draft** (PRIM2) - Content authoring
3. **Ingestion** (ALT2) - Content processing and vector store indexing
4. **Resolve** (PRIM3) - Final published state

### 2. PegaFW-QnA-Work-Index
**Purpose**: Collection (Data Collection) case type

Represents vector store collections that organize knowledge base content.

### 3. PegaFW-QnA-Work-DataSource
**Purpose**: Data Source case type

Defines content types and how content is organized within collections.

## Assignment Actions

### 1. Create Action

**Assignment ID Format**: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!CREATEFORM_DEFAULT`

**Purpose**: Configure collection and data source associations

**Usage**:
- Uses `perform_assignment_action`
- Requires pageInstructions for embedded page references
- Sets Collection, Datasource, IndexParams (optional), and ContentAccessConfigurations

**Stage Transition**: PRIM0 (Create) → PRIM2 (Draft)

### 2. AuthorContent Action

**Assignment ID Format**: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!DRAFT_FLOW`

**Purpose**: Provide article title, abstract, and content

**Usage**:
- Content parameter for scalar fields (Title, Abstract, ArticleType)
- PageInstructions for Chunks (text mode) or ContentAttachment (file mode)
- Supports two content format options: text and file

**Stage Transition**: PRIM2 (Draft) → ALT2 (Ingestion) → PRIM3 (Resolve)

## Page Instructions for Embedded Pages

### Critical Pattern

Embedded page references (Collection, Datasource, IndexParams) **cannot be set via content properties**. They require pageInstructions with the UPDATE instruction.

### Page Instruction Structure

```json
{
  "instruction": "UPDATE",
  "target": ".EmbeddedPageName",
  "content": {
    "Property": "value",
    "pyID": "ID",
    "pzInsKey": "CLASS-NAME ID"
  }
}
```

### Important Rules

1. **Target path**: MUST start with a dot (e.g., ".Collection", ".Datasource", ".IndexParams")
2. **Instruction**: Use UPDATE to add/update fields while preserving page structure
3. **Complete references**: Always include pyID and pzInsKey for object references
4. **pzInsKey format**: "CLASS-NAME ID" (with space, e.g., "PEGAFW-QNA-WORK DC-1")

### Available Instructions

| Instruction | Usage | Example |
|------------|-------|---------|
| `UPDATE` | Add/update fields in existing embedded page | Collection, Datasource, IndexParams |
| `REPLACE` | Replace entire embedded page (removes existing fields) | ContentAttachment (file mode) |
| `DELETE` | Remove embedded page | Rarely used |
| `APPEND` | Add items to page lists (arrays) | Chunks, ContentAccessConfigurations |

## Content Format Options

The AuthorContent form supports two content format types controlled by the **ArticleType** field:

### 1. Text (ArticleType="text")

**Purpose**: Manual text entry

**Characteristics**:
- Use for typing or pasting text content directly
- Content stored in Chunks page list
- Each chunk is an object with a "Content" property
- Supports multiple content chunks

**Implementation**:
- Set ArticleType: "text" in content parameter
- Use pageInstructions with APPEND for each chunk
- Target: ".Chunks"

### 2. File (ArticleType="file")

**Purpose**: File upload with automatic extraction

**Characteristics**:
- Use for uploading documents (PDF, DOCX, TXT, etc.)
- File must be uploaded and attached to case first
- Content stored in ContentAttachment embedded page
- ExtractionMethod automatically set to "Standard"
- System extracts text from document for ingestion

**Implementation**:
- Upload file using upload_attachment
- Set ArticleType: "file" in content parameter
- Use pageInstructions with REPLACE for ContentAttachment
- Requires refresh before submit

## Content Chunks Structure

**Chunks** is a page list (array) that stores multiple content pieces.

### Structure

```javascript
// Each chunk in the Chunks page list has this structure:
{
  Content: "text content for this chunk..."
}
```

### Adding Chunks

Use APPEND pageInstruction for each chunk:

```javascript
{
  instruction: "APPEND",
  target: ".Chunks",
  content: {
    Content: "text content for this chunk..."
  }
}
```

**Key Points**:
- **Target**: ".Chunks" (with dot prefix)
- **Content object**: Must contain "Content" property with the text
- **Multiple chunks**: Add separate APPEND instruction for each chunk
- **Single chunk**: Can add just one APPEND instruction for simple content

### Storage Format

After submission:
- Chunks is stored as a page list (array of objects)
- Each object contains a "Content" property
- Use `get_case_view` with viewID "AuthorContent" to see chunks as `[object Object],[object Object]...`

## Access Configurations Structure

**ContentAccessConfigurations** is a page list (array) that stores access role permissions.

### Structure

```javascript
// Each access configuration in the page list has this structure:
{
  AccessRoleName: "Knowledge Buddy Public"  // or other role names
}
```

### Adding Access Configurations

Use APPEND pageInstruction for each role:

```javascript
{
  instruction: "APPEND",
  target: ".ContentAccessConfigurations",
  content: {
    AccessRoleName: "Knowledge Buddy Public"
  }
}
```

**Key Points**:
- **Target**: ".ContentAccessConfigurations" (with dot prefix)
- **Content object**: Must contain "AccessRoleName" property
- **Multiple roles**: Add separate APPEND instruction for each role
- **Common roles**: "Knowledge Buddy Public", or other custom access roles

### Storage Format

After submission:
- ContentAccessConfigurations stored as page list
- Use `get_case_view` with viewID "Create" to see as `[object Object]`

## ContentAttachment Structure (File Mode)

**ContentAttachment** is an embedded page that stores file attachment references for file-based content.

### Structure

```javascript
{
  ID: "<attachment-id>",          // Temporary attachment ID from upload
  pyAttachName: "filename.pdf",   // Retrieved automatically by Pega
  pxObjClass: "Data-WorkAttach-File",
  // ... other metadata fields
}
```

### Setting ContentAttachment

Use REPLACE pageInstruction:

```javascript
{
  instruction: "REPLACE",
  target: ".ContentAttachment",
  content: {
    ID: tempAttachmentID  // Only need ID, Pega fills in the rest
  }
}
```

**Key Points**:
- **Target**: ".ContentAttachment" (with dot prefix)
- **Instruction**: REPLACE (not UPDATE or APPEND)
- **Content**: Only needs `{ID: tempAttachmentID}` - Pega retrieves file metadata automatically
- **Must include in both** refresh_assignment_action AND perform_assignment_action

### Upload Workflow

1. **Upload file**: Use `upload_attachment` to get temporary attachment ID (valid 2 hours)
2. **Refresh assignment**: Call `refresh_assignment_action` with pageInstructions to initialize ContentAttachment
3. **Submit action**: Call `perform_assignment_action` with same pageInstructions to finalize

## pzInsKey Format

The `pzInsKey` field is a critical reference format in Pega.

### Format

```
"CLASS-NAME ID"
```

**Note**: There is exactly ONE space between CLASS-NAME and ID.

### Examples

| Object Type | Class Name | ID | pzInsKey |
|------------|-----------|-----|----------|
| Collection | PEGAFW-QNA-WORK | DC-1 | "PEGAFW-QNA-WORK DC-1" |
| Data Source | PEGAFW-QNA-WORK | SRC-1001 | "PEGAFW-QNA-WORK SRC-1001" |

### Usage

Always provide pzInsKey when referencing objects:
- Collections require: CollectionName, pyID, pzInsKey
- Data Sources require: Name, pyID, pzInsKey

## Data View IDs

### D_IndexList

**Purpose**: Query available collections

**Key Fields**:
- CollectionName
- pyID
- pyLabel
- pyStatusWork

### D_DataSourceList

**Purpose**: Query available data sources

**Key Fields**:
- Name
- pyID
- pyLabel
- CollectionName
- pyStatusWork

## MCP Tools Reference

### Core Tools

| Tool | Purpose | Step |
|------|---------|------|
| `authenticate_pega` | Verify Pega connection | Step 0 |
| `get_list_data_view` | Query collections and data sources | Step 1 |
| `create_case` | Create Content case | Step 3 |
| `perform_assignment_action` | Execute assignment actions | Steps 4, 5 |
| `refresh_assignment_action` | Refresh assignment (file mode only) | Step 5b |
| `upload_attachment` | Upload file (file mode only) | Step 5a |
| `get_case` | Retrieve case status | Step 6 |
| `get_case_view` | Retrieve detailed view | Step 6 |
| `get_case_attachments` | List attachments (file mode) | Step 6 |

See individual step references for detailed usage of each tool.
