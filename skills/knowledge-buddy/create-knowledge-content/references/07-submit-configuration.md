# Step 7: Submit Configuration

## Purpose

Submit the Create action using pageInstructions to associate collection, data source, access roles, and optionally chunking parameters.

## MCP Tool Call

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<from-step-1>",
  actionID: "Create",
  viewType: "page",
  content: {
    Collection: { pyID: "<collection-pyID>" },
    Datasource: { pyID: "<datasource-pyID>" },
    AdvancedSettings: <true|false>  // Based on Step 6
  },
  pageInstructions: [
    // See patterns below
  ]
})
```

## PageInstructions Patterns

### Pattern 1: Collection and Data Source (Required)

Always include UPDATE instructions for collection and data source:

```javascript
[
  {
    instruction: "UPDATE",
    target: ".Collection",
    content: {
      pyID: "<collection-pyID>",
      pzInsKey: "<collection-pzInsKey>"  // e.g., "PEGAFW-QNA-WORK DC-1"
    }
  },
  {
    instruction: "UPDATE",
    target: ".Datasource",
    content: {
      pyID: "<datasource-pyID>",
      pzInsKey: "<datasource-pzInsKey>"  // e.g., "PEGAFW-QNA-WORK SRC-1"
    }
  }
]
```

### Pattern 2: Access Roles (Required)

For each selected access role from Step 5, add an INSERT followed by UPDATE:

```javascript
// For first role
{
  target: ".ContentAccessConfigurations",
  content: {},
  listIndex: 1,
  instruction: "INSERT"
},
{
  content: {
    AccessRoleName: "KnowledgeBuddy:Admin"
  },
  target: ".ContentAccessConfigurations",
  listIndex: 1,
  instruction: "UPDATE"
},
// For second role
{
  target: ".ContentAccessConfigurations",
  content: {},
  listIndex: 2,
  instruction: "INSERT"
},
{
  content: {
    AccessRoleName: "KnowledgeBuddy:Author"
  },
  target: ".ContentAccessConfigurations",
  listIndex: 2,
  instruction: "UPDATE"
}
// Add INSERT+UPDATE pairs for each role
```

If the list ends with an empty INSERT, add a DELETE to clean up:
```javascript
{
  target: ".ContentAccessConfigurations",
  content: {},
  listIndex: <next-index>,
  instruction: "INSERT"
},
{
  target: ".ContentAccessConfigurations",
  listIndex: <next-index>,
  instruction: "DELETE"
}
```

### Pattern 3: Advanced Settings (Conditional)

If `AdvancedSettings: true` in Step 6, include UPDATE for IndexParams:

```javascript
{
  instruction: "UPDATE",
  target: ".IndexParams",
  content: {
    ChunkingMethod: "SIZE",  // TITLE, SIZE, NONE, or ABSTRACT
    ChunkSize: 1000,         // Omit if ChunkingMethod is "NONE"
    ChunkOverlap: 200        // Omit if ChunkingMethod is "NONE"
  }
}
```

If `AdvancedSettings: false`, **do not** include IndexParams pageInstruction.

## Complete Example

See [examples.md](examples.md) for complete working examples with actual values.

## Critical Rules

1. **Target paths MUST start with dot**: `.Collection` not `Collection`
2. **Use UPDATE for embedded pages**: Collection, Datasource, IndexParams
3. **Use INSERT+UPDATE for page lists**: ContentAccessConfigurations
4. **pzInsKey format**: `"PEGAFW-QNA-WORK <ID>"` with single space
5. **listIndex starts at 1**: First item is 1, not 0

## Expected Response

After successful configuration, extract the new assignment ID for authoring:
```
ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!DRAFT_FLOW
```

Case transitions from **Create (PRIM0)** to **Draft (PRIM2)** stage.

## Next Step

Proceed to [08-refresh-configuration.md](08-refresh-configuration.md) to refresh after configuration.
