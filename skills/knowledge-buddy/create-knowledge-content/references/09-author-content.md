# Step 9: Author Content

## Purpose

Use `AskUserQuestion` tool to select content format (Text or File). Collect title, abstract, and content based on selected format. Submit authoring action.

## Decision Flow

### Question 1: Select Content Format

Use `AskUserQuestion` tool:
- **Question**: "What format is your content?"
- **Options**:
  - **Text**: Manual text entry with one or more content chunks
  - **File**: Upload a document (PDF, DOCX, TXT, MD) for automatic extraction

## Option A: Text Content

### User Input Collection

Use `AskUserQuestion` tool to collect:
1. **Title**: Article title
2. **Abstract**: Article summary/description
3. **Number of content chunks**: How many separate content pieces (integer, minimum 1)

Then for each chunk, use `AskUserQuestion` tool to collect:
4. **Content for chunk N**: Text content for the chunk (can be HTML formatted)

Repeat step 4 for each chunk based on the number specified.

### Submit Text Content

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<DRAFT_FLOW-assignment-from-step-7>",
  actionID: "AuthorContent",
  viewType: "form",
  content: {
    ArticleType: "text",
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>"
  },
  pageInstructions: [
    // For first chunk
    {
      target: ".Chunks",
      content: {},
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      content: {
        Content: "<p>First chunk content here</p>"
      },
      target: ".Chunks",
      listIndex: 1,
      instruction: "UPDATE"
    },
    // For second chunk
    {
      target: ".Chunks",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },
    {
      content: {
        Content: "<p>Second chunk content here</p>"
      },
      target: ".Chunks",
      listIndex: 2,
      instruction: "UPDATE"
    }
    // Add INSERT+UPDATE pairs for each chunk
  ]
})
```

### Optional: Refresh After Text Submission

After submitting text content, optionally refresh the assignment:
```javascript
mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "<DRAFT_FLOW-assignment-from-step-7>",
  actionID: "AuthorContent"
})
```

## Option B: File Content (3-Step Process)

### User Input Collection

Use `AskUserQuestion` tool to collect:
1. **Title**: Article title
2. **Abstract**: Article summary/description
3. **File Path**: Absolute path to the file to upload

**Validate**: Ensure file exists before proceeding.

### Step 9a: Upload File

```javascript
const uploadResponse = await mcp__pega-dx-mcp__upload_attachment({
  filePath: "<user-provided-file-path>",
  appendUniqueIdToFileName: true
});
const tempAttachmentID = uploadResponse.ID;
```

**Extract**: `ID` from response (e.g., "8f93ae83-78a8-44b5-a603-b7bd12e087c5")

### Step 9b: Refresh Assignment with Attachment

```javascript
await mcp__pega-dx-mcp__refresh_assignment_action({
  assignmentID: "<DRAFT_FLOW-assignment-from-step-7>",
  actionID: "AuthorContent",
  content: {
    ArticleType: "file",
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>"
  },
  pageInstructions: [{
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID },
    instruction: "REPLACE"
  }]
});
```

### Step 9c: Submit Action with Attachment

```javascript
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "<DRAFT_FLOW-assignment-from-step-7>",
  actionID: "AuthorContent",
  viewType: "form",
  content: {
    ArticleType: "file",
    Title: "<user-provided-title>",
    Abstract: "<user-provided-abstract>"
  },
  pageInstructions: [{
    target: ".ContentAttachment",
    content: { ID: tempAttachmentID },
    instruction: "REPLACE"
  }]
});
```

**Critical**: PageInstructions must be included in BOTH refresh (Step 9b) AND submit (Step 9c).

## Supported File Formats

- PDF (.pdf)
- Microsoft Word (.docx)
- Plain Text (.txt)
- Markdown (.md)

## Expected Result

After successful authoring:
- Case transitions from Draft (PRIM2) to Ingestion (ALT2) stage
- Status changes to "Pending-Ingestion" (processing) or "Resolved-Published" (complete)

## Next Step

Proceed to [10-verify-display.md](10-verify-display.md) to verify and display case summary.
