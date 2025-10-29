# PageInstructions Patterns and Rules

## Overview

PageInstructions are the mechanism for manipulating complex data structures in Pega DX API. They allow precise control over embedded pages and page lists when performing assignment actions.

## Critical Rules

1. **Target paths MUST start with dot**: `.Collection` not `Collection`
2. **Embedded pages use UPDATE**: Collection, Datasource, IndexParams
3. **Page lists use INSERT+UPDATE or APPEND**: Chunks, ContentAccessConfigurations
4. **Complete references required**: Always include name, pyID, and pzInsKey
5. **pzInsKey format**: `"CLASS-NAME ID"` with single space (e.g., `"PEGAFW-QNA-WORK DC-1"`)

## Instruction Types

### UPDATE

Used for **embedded pages** (single objects within a case).

**When to use**: Setting or modifying a single-value reference like Collection, Datasource, or IndexParams.

**Example**:
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

### APPEND

Used for **page lists** (arrays) when adding items without managing indices.

**When to use**: Adding content chunks or access roles in a simple manner.

**Example**:
```javascript
{
  instruction: "APPEND",
  target: ".Chunks",
  content: {
    Content: "<p>Chunk text here</p>"
  }
}
```

### INSERT + UPDATE

Used for **page lists** when precise control over list indices is needed.

**When to use**: Building page lists with specific ordering or when the API requires explicit INSERT before UPDATE.

**Example**:
```javascript
// Insert empty item at index 1
{
  target: ".ContentAccessConfigurations",
  content: {},
  listIndex: 1,
  instruction: "INSERT"
},
// Update the item at index 1
{
  content: {
    AccessRoleName: "KnowledgeBuddy:Admin"
  },
  target: ".ContentAccessConfigurations",
  listIndex: 1,
  instruction: "UPDATE"
}
```

**Note**: listIndex starts at 1, not 0.

### REPLACE

Used for **embedded pages** when replacing entire structures.

**When to use**: Setting file attachments or replacing complex nested objects.

**Example**:
```javascript
{
  instruction: "REPLACE",
  target: ".ContentAttachment",
  content: {
    ID: "8f93ae83-78a8-44b5-a603-b7bd12e087c5"
  }
}
```

### DELETE

Used for **page lists** to remove items.

**When to use**: Cleaning up empty list items or removing unwanted entries.

**Example**:
```javascript
{
  target: ".ContentAccessConfigurations",
  listIndex: 3,
  instruction: "DELETE"
}
```

## Common Patterns

### Pattern 1: Set Embedded Page Reference

```javascript
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge",  // Display name
    pyID: "DC-1",                 // Identifier
    pzInsKey: "PEGAFW-QNA-WORK DC-1"  // Full reference key
  }
}
```

**Always include**: Display name field, pyID, and pzInsKey.

### Pattern 2: Build Page List with INSERT+UPDATE

```javascript
[
  // First item
  {
    target: ".ContentAccessConfigurations",
    content: {},
    listIndex: 1,
    instruction: "INSERT"
  },
  {
    content: { AccessRoleName: "KnowledgeBuddy:Admin" },
    target: ".ContentAccessConfigurations",
    listIndex: 1,
    instruction: "UPDATE"
  },
  // Second item
  {
    target: ".ContentAccessConfigurations",
    content: {},
    listIndex: 2,
    instruction: "INSERT"
  },
  {
    content: { AccessRoleName: "KnowledgeBuddy:Author" },
    target: ".ContentAccessConfigurations",
    listIndex: 2,
    instruction: "UPDATE"
  }
]
```

### Pattern 3: Append to Page List

```javascript
[
  {
    instruction: "APPEND",
    target: ".Chunks",
    content: { Content: "First chunk" }
  },
  {
    instruction: "APPEND",
    target: ".Chunks",
    content: { Content: "Second chunk" }
  }
]
```

### Pattern 4: Replace Attachment

```javascript
{
  instruction: "REPLACE",
  target: ".ContentAttachment",
  content: { ID: "<temp-attachment-id>" }
}
```

## Target Path Reference

Common target paths used in this skill:

| Target Path | Type | Instruction | Purpose |
|-------------|------|-------------|---------|
| `.Collection` | Embedded Page | UPDATE | Set collection reference |
| `.Datasource` | Embedded Page | UPDATE | Set data source reference |
| `.IndexParams` | Embedded Page | UPDATE | Set chunking parameters (advanced mode) |
| `.ContentAccessConfigurations` | Page List | INSERT+UPDATE or APPEND | Add access roles |
| `.Chunks` | Page List | INSERT+UPDATE or APPEND | Add content chunks (text mode) |
| `.ContentAttachment` | Embedded Page | REPLACE | Set file attachment (file mode) |

## Common Mistakes

❌ **Forgetting the dot**: `"Collection"` instead of `".Collection"`
❌ **Wrong instruction type**: Using APPEND for embedded pages
❌ **Incomplete references**: Missing pyID or pzInsKey
❌ **Wrong pzInsKey format**: `"PEGAFW-QNA-WORK-DC-1"` instead of `"PEGAFW-QNA-WORK DC-1"` (space required)
❌ **Starting listIndex at 0**: Use 1 as first index
❌ **Missing INSERT before UPDATE**: Some page lists require INSERT first

## Best Practices

✅ Always include complete reference objects (name, pyID, pzInsKey)
✅ Use UPDATE for single-value references
✅ Use INSERT+UPDATE or APPEND for lists based on API requirements
✅ Validate pzInsKey format (class name, space, ID)
✅ Test with simple examples before complex structures
✅ Review API response for structure hints

## Reference

For complete working examples, see [examples.md](examples.md).
