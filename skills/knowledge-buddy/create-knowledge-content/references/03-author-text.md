# Author Text Content

This guide covers authoring text-based content using manual entry with multiple content chunks.

## When to Use Text Mode

Use text mode when:
- Typing or pasting content directly into the knowledge base
- Content is already in text format
- You want to manually structure content into chunks
- No file upload is needed

## Perform AuthorContent Action

Submit the content using the AuthorContent assignment from the Draft stage:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<author-content-assignment-id>",
  actionID: "AuthorContent",
  content: {
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>",
    ArticleType: "text"  // Explicitly set to text mode
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "<first-chunk-text>"
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "<second-chunk-text>"
      }
    }
    // Add more chunks as needed
  ]
})
```

**Returns:** Submits content for ingestion and moves case to Ingestion stage.

## Key Requirements

### Scalar Fields (via content parameter)
- **Title:** Article title (stored in `pyLabel`)
- **Abstract:** Summary/description
- **ArticleType:** Must be `"text"` for manual entry

### Chunks (via pageInstructions)
- **Target:** `.Chunks` (with dot prefix)
- **Instruction:** `APPEND` (adds to page list)
- **Content object:** Must have `Content` property (capital C)
- **Multiple chunks:** Add one APPEND instruction per chunk

## Content Chunks Structure

**Chunks** is a page list (array) where each item has this structure:

```javascript
{
  Content: "text content for this chunk..."
}
```

**Adding chunks:**
- Each APPEND instruction adds one chunk to the list
- Can add as many chunks as needed
- Each chunk is independent
- System will process chunks according to chunking configuration set in IndexParams

## Complete Example

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!DRAFT_FLOW",
  actionID: "AuthorContent",
  content: {
    Title: "Understanding Pega DX API",
    Abstract: "A comprehensive guide to using the Pega DX API",
    ArticleType: "text"
  },
  pageInstructions: [
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Introduction: This article covers the fundamentals of the Pega DX API. The DX API provides a modern REST interface for interacting with Pega applications."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Authentication: The DX API uses OAuth 2.1 with PKCE for secure access. You'll need to register your application and obtain client credentials."
      }
    },
    {
      instruction: "APPEND",
      target: ".Chunks",
      content: {
        Content: "Best Practices: Follow these guidelines for optimal API integration: use proper error handling, implement retry logic, and cache tokens appropriately."
      }
    }
  ]
});
```

**Result:**
- Case moves to Ingestion stage
- Status becomes "Pending-Ingestion" while processing
- Chunks are stored as page list with 3 objects
- Each object contains a Content property

## Common Issues

1. **Empty Chunks after submission:**
   - Verify `pageInstructions` with APPEND are used (not content property)
   - Target must be `.Chunks` (with dot)
   - Each chunk object must have `Content` property (capital C)

2. **Title or Abstract not saved:**
   - These must be in `content` parameter (not pageInstructions)
   - Use exact field names: `Title` and `Abstract`

3. **Wrong content format:**
   - Ensure `ArticleType: "text"` is set
   - Don't try to set Chunks as a string in content parameter

## Verification

After submission, verify the content was stored:

```javascript
mcp__pega-dx-mcp__get_case_view({
  caseID: "<case-id>",
  viewID: "AuthorContent"
})
```

**What to check:**
- `Title` - Should contain the title
- `Abstract` - Should contain the abstract
- `ArticleType` - Should be "text"
- `Chunks` - Should show as `[object Object],[object Object]...`

For more verification details, see **[05-verification.md](./05-verification.md)**.

## Next Steps

- **Verify success:** See **[05-verification.md](./05-verification.md)**
- **For file upload instead:** See **[04-author-file.md](./04-author-file.md)**
- **Technical details:** See **[technical-reference.md](./technical-reference.md)**
