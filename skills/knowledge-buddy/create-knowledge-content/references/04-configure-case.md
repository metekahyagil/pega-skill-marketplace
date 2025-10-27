# Step 4: Configure Collection, Data Source, and Access

## Purpose

Submit Create assignment to configure collection, data source, access roles, and optionally chunking parameters via pageInstructions.

## Assignment Action

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<from-step-3>",
  actionID: "Create",
  content: { /* mode-specific */ },
  pageInstructions: [ /* see below */ ]
})
```

## Configuration Modes

### Simple Mode (Default Chunking)

Use when user selected "Simple" settings. No `AdvancedSettings` field, no `IndexParams`.

```javascript
{
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {},
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
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "KnowledgeBuddy:Public"
      }
    }
  ]
}
```

**Defaults**: ChunkingMethod: SIZE, ChunkSize: 1000, ChunkOverlap: 200

### Advanced Mode (Custom Chunking)

Use when user selected "Advanced" settings. Includes `AdvancedSettings: true` and `IndexParams`.

```javascript
{
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
        ChunkingMethod: "SIZE",  // TITLE, SIZE, NONE, or ABSTRACT
        ChunkSize: 2000,
        ChunkOverlap: 300
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "KnowledgeBuddy:Public"
      }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "KnowledgeBuddy:Internal"
      }
    }
  ]
}
```

## PageInstructions Rules

**Critical**:
- Targets MUST start with dot: `.Collection` not `Collection`
- Use `UPDATE` for embedded pages (Collection, Datasource, IndexParams)
- Use `APPEND` for page lists (ContentAccessConfigurations)
- Collection/Datasource require all three: name field, pyID, pzInsKey
- pzInsKey format: `"CLASS-NAME ID"` with single space (e.g., `"PEGAFW-QNA-WORK DC-1"`)

**Access Roles**: Add one APPEND instruction per selected role. If no selection, default to `"KnowledgeBuddy:Public"`.

## Response

Case transitions: Create (PRIM0) → Draft (PRIM2)

**Extract**: New assignment ID for "Author content" action:
```
ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!DRAFT_FLOW
```

## Next Step

[05-author-content.md](05-author-content.md)

**Exceptions**: See [error-handling/04-configure-case-exceptions.md](error-handling/04-configure-case-exceptions.md) for troubleshooting.
