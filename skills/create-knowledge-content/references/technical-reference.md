# Technical Reference

This document provides technical details about page instructions, data structures, and the Pega Knowledge Buddy case structure.

## Page Instructions

Page instructions are used to manipulate embedded pages (single objects) and page lists (arrays) in Pega cases. They allow precise control over nested data structures.

### Instruction Types

#### UPDATE
**Purpose:** Modify an embedded page (single object)

**Usage:**
- Setting Collection, Datasource, IndexParams
- Updating scalar values in embedded pages

**Example:**
```javascript
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge",
    pyID: "DC-1",
    pzInsKey: "PEGAFW-QNA-WORK DC-1"
  }
}
```

**Rules:**
- Target must start with dot (`.`)
- Content contains properties to set/update
- Preserves existing page structure
- Use for single pages, not page lists

#### APPEND
**Purpose:** Add items to a page list (array)

**Usage:**
- Adding chunks to Chunks page list
- Adding access roles to ContentAccessConfigurations
- Adding any list item

**Example:**
```javascript
{
  instruction: "APPEND",
  target: ".Chunks",
  content: {
    Content: "chunk text here..."
  }
}
```

**Rules:**
- Target must start with dot (`.`)
- Content is the object to append
- Creates new list item each call
- Can append multiple times in same action

#### REPLACE
**Purpose:** Replace entire embedded page

**Usage:**
- Less common, use UPDATE instead for partial updates
- Replaces entire object structure

**Warning:** Can lose existing data if not careful. Prefer UPDATE.

#### DELETE
**Purpose:** Remove embedded page or list item

**Usage:**
- Removing specific items from lists
- Clearing embedded pages

**Less common in content creation workflow.**

### Target Path Rules

All page instruction targets must:
1. Start with a dot (`.`) character
2. Reference the property name exactly as it appears in the data model
3. Use proper case (e.g., `.Chunks` not `.chunks`)

**Examples:**
- `.Collection` ✓
- `Collection` ✗ (missing dot)
- `.Datasource` ✓
- `.datasource` ✗ (wrong case)
- `.Chunks` ✓
- `.ContentAccessConfigurations` ✓

## Data Structures

### Collection Reference
**Type:** Embedded page (single object)

**Required fields:**
- `CollectionName` (string) - Collection identifier
- `pyID` (string) - Collection ID (e.g., "DC-1")
- `pzInsKey` (string) - Full key: `"PEGAFW-QNA-WORK <pyID>"`

**Example:**
```javascript
{
  CollectionName: "knowledge",
  pyID: "DC-1",
  pzInsKey: "PEGAFW-QNA-WORK DC-1"
}
```

### Datasource Reference
**Type:** Embedded page (single object)

**Required fields:**
- `Name` (string) - Datasource identifier
- `pyID` (string) - Datasource ID (e.g., "SRC-1001")
- `pzInsKey` (string) - Full key: `"PEGAFW-QNA-WORK <pyID>"`

**Example:**
```javascript
{
  Name: "Knowledge_ingestion_test",
  pyID: "SRC-1001",
  pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
}
```

### ContentAccessConfigurations
**Type:** Page list (array of objects)

**Item structure:**
```javascript
{
  AccessRoleName: "Knowledge Buddy Public"
}
```

**Multiple roles example:**
```javascript
[
  { AccessRoleName: "Knowledge Buddy Public" },
  { AccessRoleName: "Knowledge Buddy Admin" }
]
```

**How to populate:**
- Use APPEND instruction for each role
- Cannot set entire array at once
- Each APPEND adds one object to the list

### Chunks
**Type:** Page list (array of objects)

**Item structure:**
```javascript
{
  Content: "text content for this chunk..."
}
```

**Multiple chunks example:**
```javascript
[
  { Content: "First chunk of text..." },
  { Content: "Second chunk of text..." },
  { Content: "Third chunk of text..." }
]
```

**How to populate:**
- Use APPEND instruction for each chunk
- One APPEND per chunk
- Content property must have capital C

### IndexParams
**Type:** Embedded page (single object)

**Optional fields:**
- `ChunkingMethod` (string) - "TITLE", "SIZE", "NONE", "ABSTRACT"
- `ChunkSize` (integer) - Size in characters (for SIZE method)
- `ChunkOverlap` (integer) - Overlap between chunks

**Example:**
```javascript
{
  ChunkingMethod: "SIZE",
  ChunkSize: 1000,
  ChunkOverlap: 200
}
```

**Chunking methods:**
- **TITLE** - Split by document sections/headings
- **SIZE** - Fixed character chunks (requires ChunkSize)
- **NONE** - No chunking, keep as single unit
- **ABSTRACT** - Split by abstract structure

### ContentAttachment
**Type:** Embedded page (single object)

**Auto-populated by system** when:
- ArticleType is set to "file"
- File has been attached to case via `add_case_attachments`

**Do not set manually** - system manages this field.

## pzInsKey Format

The `pzInsKey` (instance key) is a critical identifier format in Pega.

**Format:** `"<CLASS-NAME> <ID>"`

**Rules:**
1. Class name in uppercase with hyphens
2. **Single space** between class name and ID
3. ID matches the pyID field

**Examples:**
- Collection: `"PEGAFW-QNA-WORK DC-1"`
- Datasource: `"PEGAFW-QNA-WORK SRC-1001"`

**Common mistakes:**
- Missing space: `"PEGAFW-QNA-WORK-DC-1"` ✗
- No space: `"PEGAFW-QNA-WORKDC-1"` ✗
- Multiple spaces: `"PEGAFW-QNA-WORK  DC-1"` ✗
- Correct: `"PEGAFW-QNA-WORK DC-1"` ✓

## Case Types and Actions

### PegaFW-KB-Work-Article

Main case type for content articles.

**Stages:**
1. **New** - Initial creation
2. **Draft** - Configuration phase
3. **Ingestion** - Content submitted, being processed

**Assignment Actions:**

#### Create (CREATEFORM_DEFAULT)
**Purpose:** Configure collection, datasource, access, and chunking

**Appears after:** Case creation

**Parameters:**
- `content.AdvancedSettings` (boolean) - Enable chunking configuration
- `pageInstructions` - For Collection, Datasource, ContentAccessConfigurations, IndexParams

**Result:** Creates "Author content" assignment (DRAFT_FLOW)

#### AuthorContent (DRAFT_FLOW)
**Purpose:** Submit content (text or file)

**Appears after:** Create action

**Parameters:**
- `content.Title` (string) - Article title
- `content.Abstract` (string) - Article description
- `content.ArticleType` (string) - "text" or "file"
- `pageInstructions` - For Chunks (text mode only)

**Result:** Moves to Ingestion stage, status becomes Pending-Ingestion

## Case Field Reference

**Scalar fields (in content parameter):**
- `Title` - Article title (also stored as `pyLabel`)
- `Abstract` - Article description
- `ArticleType` - "text" or "file"
- `ExtractionMethod` - Auto-set to "Standard" for files
- `IngestionErrorMessage` - Error details if ingestion fails

**Embedded pages (via pageInstructions UPDATE):**
- `Collection` - Collection reference object
- `Datasource` - Datasource reference object
- `IndexParams` - Chunking configuration
- `ContentAttachment` - File reference (auto-populated)

**Page lists (via pageInstructions APPEND):**
- `Chunks` - Text content chunks
- `ContentAccessConfigurations` - Access role list

**System fields:**
- `pyID` - Case ID (e.g., "KB-1023")
- `pyLabel` - Case label (usually same as Title)
- `pyStatusWork` - Case status
- `pxObjClass` - Case type class
- `pxCreateDateTime` - Creation timestamp

## Assignment ID Format

Assignment IDs follow this pattern:

```
ASSIGN-WORKLIST <CASE-TYPE> <CASE-ID>!<ACTION-NAME>
```

**Examples:**
- `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT`
- `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW`

**Components:**
- `ASSIGN-WORKLIST` - Assignment type prefix
- Case type and ID
- `!` separator
- Action/flow name

## Common Patterns

### Setting a Single Page Property
```javascript
pageInstructions: [{
  instruction: "UPDATE",
  target: ".PropertyName",
  content: { field1: "value1", field2: "value2" }
}]
```

### Adding Multiple List Items
```javascript
pageInstructions: [
  {
    instruction: "APPEND",
    target: ".ListName",
    content: { item: "first" }
  },
  {
    instruction: "APPEND",
    target: ".ListName",
    content: { item: "second" }
  }
]
```

### Mixed Operations
```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".SinglePage",
    content: { /* object */ }
  },
  {
    instruction: "APPEND",
    target: ".ListPage",
    content: { /* item */ }
  }
]
```

## API Error Codes

**400 BAD_REQUEST:**
- Invalid pzInsKey format
- Missing required fields
- Malformed JSON
- Invalid instruction type

**404 NOT_FOUND:**
- Case doesn't exist
- Assignment doesn't exist
- Invalid case type

**500 INTERNAL_SERVER_ERROR:**
- Pega system error
- Processing failure
- Check Pega logs

## Best Practices

1. **Always use pageInstructions** for Collection, Datasource, Chunks, and ContentAccessConfigurations
2. **Verify pzInsKey format** includes space between class and ID
3. **Use UPDATE for single pages**, APPEND for page lists
4. **Set ArticleType explicitly** ("text" or "file")
5. **Attach files before** submitting AuthorContent in file mode
6. **Check IngestionErrorMessage** after submission for file mode
7. **Store IDs consistently** when querying collections/datasources
8. **Use exact property names** (case-sensitive)

## Related Documentation

- **[01-query-data.md](./01-query-data.md)** - Query collections and datasources
- **[02-create-and-configure.md](./02-create-and-configure.md)** - Create and configure cases
- **[03-author-text.md](./03-author-text.md)** - Author text content
- **[04-author-file.md](./04-author-file.md)** - Author file content
- **[05-verification.md](./05-verification.md)** - Verify success
- **[examples.md](./examples.md)** - Complete working examples
