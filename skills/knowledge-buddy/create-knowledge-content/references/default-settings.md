# Default Settings and Configuration Modes

This document explains the configuration modes available when creating knowledge base content.

## Overview

This skill supports two configuration modes:
1. **Simple Settings** (Recommended) - Uses system defaults
2. **Advanced Settings** - User-configurable chunking parameters

The mode is selected by the user in Step 2a (User Input).

---

## Simple Settings (Recommended)

### When to Use

Use simple settings when:
- User is not familiar with chunking concepts
- Default settings are appropriate for the content
- Quick content creation is desired
- No special chunking requirements exist

### Default Values

When simple settings are selected, the system uses these defaults:

| Parameter | Default Value | Description |
|-----------|--------------|-------------|
| **ChunkingMethod** | SIZE | Chunks content by fixed character/token size |
| **ChunkSize** | 1000 | 1000 characters per chunk |
| **ChunkOverlap** | 200 | 200 characters overlap between chunks |

### Implementation

In Step 4 (Configure Case), **omit the IndexParams pageInstruction** and do not set AdvancedSettings:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id>",
  actionID: "Create",
  content: {},  // No AdvancedSettings
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: { /* collection data */ }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: { /* datasource data */ }
    },
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: { AccessRoleName: "Knowledge Buddy Public" }
    }
    // No IndexParams - system uses defaults
  ]
})
```

### Key Points

- ✅ **Do NOT set** `AdvancedSettings: true` in content
- ✅ **Do NOT include** IndexParams pageInstruction
- ✅ System automatically applies default chunking parameters
- ✅ Suitable for most use cases

---

## Advanced Settings

### When to Use

Use advanced settings when:
- User has specific chunking requirements
- Content has special structure (titles, sections, abstracts)
- Custom chunk sizes needed for performance optimization
- User wants to experiment with different chunking methods

### Configurable Parameters

#### 1. ChunkingMethod

Controls how content is divided into chunks.

| Method | Description | Use Case |
|--------|-------------|----------|
| **TITLE** | Chunks by document title/sections | Structured documents with clear section boundaries |
| **SIZE** (default) | Chunks by fixed character/token size | General-purpose, most content types |
| **NONE** | No chunking, single unit | Short content, already segmented |
| **ABSTRACT** | Chunks based on abstract/summary structure | Academic papers, research documents |

#### 2. ChunkSize

Integer value controlling the size of each chunk.

- **Default**: 1000
- **Typical range**: 500-5000 characters
- **Considerations**:
  - Larger chunks: Better context, fewer embeddings
  - Smaller chunks: More granular search, more embeddings
  - Too large: May exceed model limits
  - Too small: Loss of context

#### 3. ChunkOverlap

Integer value controlling overlap between adjacent chunks.

- **Default**: 200
- **Typical range**: 50-500 characters
- **Constraint**: **Must be less than ChunkSize**
- **Considerations**:
  - Overlap helps maintain context across chunk boundaries
  - Larger overlap: Better context preservation, more redundancy
  - Smaller overlap: Less redundancy, potential context loss
  - Zero overlap: Maximum efficiency, may lose cross-boundary context

### Implementation

In Step 4 (Configure Case), **include IndexParams pageInstruction** and set AdvancedSettings:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<assignment-id>",
  actionID: "Create",
  content: {
    AdvancedSettings: true  // Enables advanced configuration UI
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".Collection",
      content: { /* collection data */ }
    },
    {
      instruction: "UPDATE",
      target: ".Datasource",
      content: { /* datasource data */ }
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
    {
      instruction: "APPEND",
      target: ".ContentAccessConfigurations",
      content: { AccessRoleName: "Knowledge Buddy Public" }
    }
  ]
})
```

### Key Points

- ✅ **Set** `AdvancedSettings: true` in content parameter
- ✅ **Include** IndexParams pageInstruction with UPDATE
- ✅ **Target** must be `.IndexParams` (with dot prefix)
- ✅ All three parameters must be provided (ChunkingMethod, ChunkSize, ChunkOverlap)
- ✅ Validate ChunkOverlap < ChunkSize

---

## Content Format Independence

**Important**: The ArticleType (text vs file) is **independent** of the settings mode:

- Simple Settings can be used with both text and file content
- Advanced Settings can be used with both text and file content

The ArticleType is determined by user selection in Step 2a, separate from the settings mode.

---

## Example Workflow Variations

### 1. Simple Settings + Text Content

**User Selections (Step 2)**:
- Settings Mode: Simple
- Content Format: Text
- Number of Chunks: 2

**Step 4 - Configuration**:
```javascript
// No IndexParams, no AdvancedSettings
content: {},
pageInstructions: [
  { Collection }, { Datasource }, { ContentAccessConfigurations }
]
```

**Step 5 - Author Content**:
```javascript
// Text mode with 2 chunks
content: { Title: "...", Abstract: "...", ArticleType: "text" },
pageInstructions: [
  { APPEND chunk 1 },
  { APPEND chunk 2 }
]
```

**Result**: Content with default chunking (SIZE, 1000, 200) and 2 manually entered chunks

---

### 2. Advanced Settings + Text Content

**User Selections (Step 2)**:
- Settings Mode: Advanced
- Chunking Method: SIZE
- Chunk Size: 2000
- Chunk Overlap: 300
- Content Format: Text
- Number of Chunks: 3

**Step 4 - Configuration**:
```javascript
// With IndexParams and AdvancedSettings
content: { AdvancedSettings: true },
pageInstructions: [
  { Collection },
  { Datasource },
  { IndexParams: { ChunkingMethod: "SIZE", ChunkSize: 2000, ChunkOverlap: 300 } },
  { ContentAccessConfigurations }
]
```

**Step 5 - Author Content**:
```javascript
// Text mode with 3 chunks
content: { Title: "...", Abstract: "...", ArticleType: "text" },
pageInstructions: [
  { APPEND chunk 1 },
  { APPEND chunk 2 },
  { APPEND chunk 3 }
]
```

**Result**: Content with custom chunking (SIZE, 2000, 300) and 3 manually entered chunks

---

### 3. Simple Settings + File Content

**User Selections (Step 2)**:
- Settings Mode: Simple
- Content Format: File
- File Path: /path/to/document.pdf

**Step 4 - Configuration**:
```javascript
// No IndexParams, no AdvancedSettings
content: {},
pageInstructions: [
  { Collection }, { Datasource }, { ContentAccessConfigurations }
]
```

**Step 5 - Author Content**:
```javascript
// File mode with 3-step upload workflow
1. upload_attachment
2. refresh_assignment_action with ContentAttachment pageInstructions
3. perform_assignment_action with same pageInstructions
   content: { Title: "...", Abstract: "...", ArticleType: "file" }
```

**Result**: File content with default chunking (SIZE, 1000, 200) and automatic extraction

---

### 4. Advanced Settings + File Content

**User Selections (Step 2)**:
- Settings Mode: Advanced
- Chunking Method: TITLE (for document structure)
- Chunk Size: 1500
- Chunk Overlap: 250
- Content Format: File
- File Path: /path/to/document.pdf

**Step 4 - Configuration**:
```javascript
// With IndexParams and AdvancedSettings
content: { AdvancedSettings: true },
pageInstructions: [
  { Collection },
  { Datasource },
  { IndexParams: { ChunkingMethod: "TITLE", ChunkSize: 1500, ChunkOverlap: 250 } },
  { ContentAccessConfigurations }
]
```

**Step 5 - Author Content**:
```javascript
// File mode with 3-step upload workflow
1. upload_attachment
2. refresh_assignment_action with ContentAttachment pageInstructions
3. perform_assignment_action with same pageInstructions
   content: { Title: "...", Abstract: "...", ArticleType: "file" }
```

**Result**: File content with custom chunking (TITLE method, 1500, 250) and automatic extraction

---

## Chunking Method Guidelines

### SIZE Method (Default)

**Best for**:
- General-purpose content
- Unstructured text
- Content without clear section boundaries

**How it works**: Divides content into fixed-size chunks with specified overlap

**Example**:
- ChunkSize: 1000
- ChunkOverlap: 200
- Content is split into 1000-character chunks with 200 characters overlapping between each chunk

### TITLE Method

**Best for**:
- Structured documents with clear section titles
- Documentation with headings
- Books or articles with chapter/section structure

**How it works**: Divides content based on document structure (titles, headings)

**Example**:
- Document with sections: Introduction, Methods, Results, Conclusion
- Each section becomes a chunk (respecting ChunkSize limits)

### NONE Method

**Best for**:
- Short content (less than optimal chunk size)
- Pre-segmented content
- Content that should not be divided

**How it works**: Treats entire content as single unit (no chunking)

**Example**:
- Short FAQ answer
- Single concept explanation
- Already-segmented content piece

### ABSTRACT Method

**Best for**:
- Academic papers
- Research documents
- Content with abstract/summary structure

**How it works**: Divides based on abstract and document structure

**Example**:
- Paper with: Abstract, Introduction, Literature Review, Methods, Results, Discussion
- Chunks follow abstract structure with consideration for logical boundaries

---

## Access Configuration

**Note**: Access roles are independent of settings mode.

### Default Access Role

Both simple and advanced settings use the same default access role:
- **"Knowledge Buddy Public"** - Public access to knowledge base content

### Multiple Access Roles

For both modes, multiple access roles can be added with multiple APPEND instructions:

```javascript
{
  instruction: "APPEND",
  target: ".ContentAccessConfigurations",
  content: { AccessRoleName: "Knowledge Buddy Public" }
},
{
  instruction: "APPEND",
  target: ".ContentAccessConfigurations",
  content: { AccessRoleName: "Internal Users" }
}
```

---

## Choosing the Right Mode

### Use Simple Settings when:
- ✅ User is unfamiliar with chunking concepts
- ✅ Content is general-purpose text or documentation
- ✅ Default parameters are appropriate (most cases)
- ✅ Quick implementation is desired
- ✅ User doesn't have specific requirements

### Use Advanced Settings when:
- ✅ Content has special structure (sections, abstracts)
- ✅ User has specific performance requirements
- ✅ Custom chunk sizes needed for specific use case
- ✅ User wants to experiment with different approaches
- ✅ User is familiar with chunking and vector search concepts

**Recommendation**: Start with simple settings and move to advanced only if specific needs arise.

---

## Related References

- **[02-user-input.md](./02-user-input.md)** - Gathering user preferences for settings mode
- **[04-configure-case.md](./04-configure-case.md)** - Implementing simple vs advanced configuration
- **[technical-details.md](./technical-details.md)** - Page instructions and data structures
- **[examples.md](./examples.md)** - Working examples with both modes
