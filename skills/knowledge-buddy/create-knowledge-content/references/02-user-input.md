# Step 2: Gather User Input

## Purpose

Collect user preferences for settings mode, content format, and content details using `AskUserQuestion` tool.

## Required Information

Due to 4-question limit per `AskUserQuestion` call, this requires 2-3 calls depending on choices.

### First Call: Configuration

1. **Collection Selection**: Present available collections from Step 1
2. **Data Source Selection**: Present available data sources from Step 1
3. **Settings Mode**: Simple (default chunking) or Advanced (custom chunking)
4. **Content Format**: Text (manual entry) or File (document upload)

### Second Call (if Advanced): Chunking Settings

1. **Chunking Method**: TITLE, SIZE (default), NONE, or ABSTRACT
2. **Chunk Size**: Integer (default: 1000, range: 500-5000)
3. **Chunk Overlap**: Integer (default: 200, must be < chunk size)

### Final Call: Content Details

1. **Access Roles**: Multi-select from Step 1 roles (default: KnowledgeBuddy:Public)
2. **Title**: Article title
3. **Abstract**: Article summary/description
4. **Content Input**:
   - **Text**: Number of chunks and content for each
   - **File**: Absolute file path (validate exists)

## Default Settings (Simple Mode)

- ChunkingMethod: SIZE
- ChunkSize: 1000
- ChunkOverlap: 200

## Data to Capture

Store for subsequent steps:
- Collection: CollectionName, pyID, pzInsKey
- Data Source: Name, pyID, pzInsKey
- Settings: mode (simple/advanced), chunking params (if advanced)
- Format: text or file
- Content: Title, Abstract, AccessRoles[], Chunks[] or FilePath

## Next Step

[03-create-case.md](03-create-case.md)

**Exceptions**: See [error-handling/02-user-input-exceptions.md](error-handling/02-user-input-exceptions.md) for validation failures.
