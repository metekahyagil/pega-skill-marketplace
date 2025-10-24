# Step 4: Configure Collection, Data Source, and Access

After creating the Content case, perform the Create action to configure collection, data source, access roles, and optionally chunking parameters.

## Quick Checklist

- [ ] Use `perform_assignment_action` with assignmentID from Step 3
- [ ] Set actionID to "Create"
- [ ] Add pageInstructions with UPDATE for Collection (with pyID and pzInsKey)
- [ ] Add pageInstructions with UPDATE for Datasource (with pyID and pzInsKey)
- [ ] If Advanced settings: Add pageInstructions with UPDATE for IndexParams
- [ ] Add pageInstructions with APPEND for each selected access role
- [ ] Verify all target paths start with dot (.)
- [ ] Verify pzInsKey format has single space
- [ ] Capture new assignmentID for Draft stage
- [ ] Proceed to Step 5 (Author Content)

---

## Table of Contents

- [Assignment Action](#assignment-action)
- [Configuration Modes](#configuration-modes)
  - [Simple Settings (Default)](#simple-settings-default)
  - [Advanced Settings](#advanced-settings)
- [Critical Points](#critical-points)
  - [PageInstructions for Embedded Pages](#pageinstructions-for-embedded-pages)
  - [Target Paths](#target-paths)
  - [Instructions](#instructions)
  - [Complete References](#complete-references)
  - [pzInsKey Format](#pzinskey-format)
  - [Access Roles](#access-roles)
- [What This Returns](#what-this-returns)
- [Example with Actual Values](#example-with-actual-values)
  - [Simple Settings Example](#simple-settings-example)
  - [Advanced Settings Example](#advanced-settings-example)
- [Next Step](#next-step)

---

## Assignment Action

Use the `perform_assignment_action` MCP tool with:
- **assignmentID**: From Step 3 (case creation)
- **actionID**: `"Create"`
- **content**: Configuration settings
- **pageInstructions**: For embedded pages (Collection, Datasource, IndexParams, ContentAccessConfigurations)

## Configuration Modes

The configuration differs based on the **Settings Mode** selected by the user in Step 2:

### Simple Settings (Default)

Use this when user selected "Simple" settings in Step 2a.

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id-from-step-3>",
  actionID: "Create",
  content: {},  // No AdvancedSettings
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
    // Add APPEND for each selected access role from Step 2c
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "<selected-role-1>"  // e.g., "KnowledgeBuddy:Public"
      }
    }
    // Add more APPEND instructions if user selected multiple roles
    // No IndexParams - system uses defaults
  ]
})
```

**Simple Settings Defaults:**
- ChunkingMethod: SIZE
- ChunkSize: 1000
- ChunkOverlap: 200

### Advanced Settings

Use this when user selected "Advanced" settings in Step 2a.

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id-from-step-3>",
  actionID: "Create",
  content: {
    AdvancedSettings: true  // Enables advanced configuration UI
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
        ChunkingMethod: "<user-selected-method>",  // TITLE, SIZE, NONE, or ABSTRACT
        ChunkSize: <user-selected-size>,           // Integer value
        ChunkOverlap: <user-selected-overlap>      // Integer value
      }
    },
    // Add APPEND for each selected access role from Step 2c
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: {
        AccessRoleName: "<selected-role-1>"  // e.g., "KnowledgeBuddy:Public"
      }
    }
    // Add more APPEND instructions if user selected multiple roles
  ]
})
```

**Advanced Settings:**
- User-configured chunking from Step 2b
- Set `AdvancedSettings: true` in content
- Include IndexParams pageInstruction

## Critical Points

### PageInstructions for Embedded Pages

**MUST use pageInstructions** for Collection, Datasource, and IndexParams. They cannot be set via content properties.

### Target Paths

Target paths **MUST start with a dot**:
- `.Collection` ✅ (correct)
- `Collection` ❌ (will fail)

### Instructions

- **UPDATE**: Use for embedded pages (Collection, Datasource, IndexParams)
  - Adds/updates fields while preserving page structure
- **APPEND**: Use for page lists (ContentAccessConfigurations)
  - Adds items to arrays

### Complete References

Collection and Datasource require **all three properties**:
1. Primary identifier (`CollectionName` or `Name`)
2. `pyID` - The object ID
3. `pzInsKey` - The instance key

### pzInsKey Format

Format: `"CLASS-NAME ID"` (with single space)

Examples:
- Collection: `"PEGAFW-QNA-WORK DC-1"`
- Data Source: `"PEGAFW-QNA-WORK SRC-1001"`

### Access Roles

ContentAccessConfigurations is a **page list** (array):
- Use APPEND instruction for each selected role from Step 2c
- User can select one or more roles
- Create one APPEND instruction per selected role
- If no roles selected, default to "KnowledgeBuddy:Public"

**For single role:**
```javascript
{
  instruction: "APPEND",
  target: ".ContentAccessConfigurations",
  content: {
    AccessRoleName: "KnowledgeBuddy:Public"
  }
}
```

**For multiple roles:**
```javascript
// If user selected ["KnowledgeBuddy:Public", "KnowledgeBuddy:Internal"]
selectedRoles.forEach(role => {
  pageInstructions.push({
    instruction: "APPEND",
    target: ".ContentAccessConfigurations",
    content: {
      AccessRoleName: role
    }
  });
});

// Results in multiple APPEND instructions:
// 1. APPEND KnowledgeBuddy:Public
// 2. APPEND KnowledgeBuddy:Internal
```

## What This Returns

After successful configuration:
- Case progresses to **Draft** stage (PRIM2)
- New assignment created: "Author content"
- Assignment ID format: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!DRAFT_FLOW`

Example: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW`

## Example with Actual Values

### Simple Settings Example

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
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
})
```

### Advanced Settings Example

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
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
})
```

**Note**: This example shows two access roles selected by the user. Add one APPEND instruction per selected role.

## Next Step

After configuration, proceed to **[05-author-content.md](./05-author-content.md)** to author the article content.

---

**Exceptions**: See **[error-handling/04-configure-case-exceptions.md](error-handling/04-configure-case-exceptions.md)** for configuration failures and troubleshooting.
