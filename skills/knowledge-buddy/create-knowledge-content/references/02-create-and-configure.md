# Create and Configure Content Case

This guide covers creating the Content case and configuring all required associations (collection, data source, access roles, and chunking parameters).

## Step 1: Create Content Case

Create a new Content case using the Article case type:

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
})
```

**Returns:**
- Case ID (e.g., "PEGAFW-KB-WORK-ARTICLE KB-1023")
- Assignment ID for the Create action (e.g., "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT")

**Note:** The case accepts empty content on creation. All configuration happens in the next step.

## Step 2: Configure Case with Collection, Data Source, Access, and Chunking

Perform the Create action with `pageInstructions` to set all configurations in one operation:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id-from-step-1>",
  actionID: "Create",
  content: {
    AdvancedSettings: true  // Optional: enable if configuring chunking
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "<selected-collection-name>",
        pyID: "<selected-collection-id>",
        pzInsKey: "<selected-collection-inskey>"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "<selected-datasource-name>",
        pyID: "<selected-datasource-id>",
        pzInsKey: "<selected-datasource-inskey>"
      }
    },
    {
      instruction: "UPDATE",
      target: ".IndexParams",
      content: {
        ChunkingMethod: "TITLE",  // Optional: "TITLE", "SIZE", "NONE", "ABSTRACT"
        ChunkSize: 1000,           // Optional: for SIZE method
        ChunkOverlap: 200          // Optional: overlap for context
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "Knowledge Buddy Public"
      }
    }
    // Add more access roles with additional APPEND instructions if needed
  ]
})
```

**Returns:**
- Progresses case to Draft stage
- Creates "Author content" assignment (e.g., "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW")

## Critical Requirements

### Collection and Datasource Configuration

Both MUST be set using `pageInstructions` with the `UPDATE` instruction:

- **Target paths:** Must start with a dot (`.Collection`, `.Datasource`)
- **Instruction:** Use `UPDATE` (not REPLACE) to preserve page structure
- **Complete references required:**
  - `CollectionName` or `Name`
  - `pyID`
  - `pzInsKey` in format: `"PEGAFW-QNA-WORK <ID>"` (with space)

### Access Configurations

- **Target:** `.ContentAccessConfigurations` (page list)
- **Instruction:** `APPEND` (adds to list)
- **Content:** Object with `AccessRoleName` property
- **Multiple roles:** Add multiple APPEND instructions

### Chunking Configuration (Optional)

- **Target:** `.IndexParams`
- **Instruction:** `UPDATE`
- **ChunkingMethod options:**
  - `TITLE` - Chunk by document sections/titles
  - `SIZE` - Fixed size chunks (requires ChunkSize)
  - `NONE` - No chunking (single unit)
  - `ABSTRACT` - Chunk by abstract structure
- **ChunkSize:** Integer (e.g., 1000, 5000) - for SIZE method
- **ChunkOverlap:** Integer (e.g., 200) - overlap between chunks

**Note:** Omit IndexParams to use system defaults.

## Complete Example

```javascript
// Step 1: Create case
const createResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article"
});
// Returns: KB-1023 with CREATEFORM_DEFAULT assignment

// Step 2: Configure everything
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    AdvancedSettings: true
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: {
        CollectionName: "knowledge",
        pyID: "DC-1",
        pzInsKey: "PEGAFW-QNA-WORK DC-1"
      }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: {
        Name: "Knowledge_ingestion_test",
        pyID: "SRC-1001",
        pzInsKey: "PEGAFW-QNA-WORK SRC-1001"
      }
    },
    {
      instruction: "UPDATE",
      target: ".IndexParams",
      content: {
        ChunkingMethod: "SIZE",
        ChunkSize: 1000,
        ChunkOverlap: 200
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "Knowledge Buddy Public"
      }
    }
  ]
});
// Returns: Assignment for DRAFT_FLOW (author content)
```

## Common Issues

1. **Empty Collection/Datasource fields:**
   - Verify `pageInstructions` are used (not `content` properties)
   - Check target paths start with dot
   - Ensure `UPDATE` instruction is used
   - Verify `pzInsKey` format: "CLASS-NAME ID" with space

2. **BAD_REQUEST errors:**
   - Missing dot prefix in target
   - Incorrect `pzInsKey` format
   - Missing required fields (pyID or pzInsKey)

3. **Empty access configurations:**
   - Use APPEND instruction (not UPDATE)
   - Target must be `.ContentAccessConfigurations`
   - Content must have `AccessRoleName` property

## Next Steps

After configuration, proceed to content authoring:
- **Text content:** See **[03-author-text.md](./03-author-text.md)**
- **File content:** See **[04-author-file.md](./04-author-file.md)**

For more technical details, see **[technical-reference.md](./technical-reference.md)**.
