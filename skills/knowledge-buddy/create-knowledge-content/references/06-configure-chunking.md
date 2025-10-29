# Step 6: Configure Chunking Settings

## Purpose

Use `AskUserQuestion` tool to determine if user wants advanced chunking settings. If No, use defaults. If Yes, collect chunking method, size, and overlap.

## Decision Flow

### Question 1: Enable Advanced Settings

Use `AskUserQuestion` tool:
- **Question**: "Do you want to configure advanced chunking settings?"
- **Options**:
  - **No (Recommended)**: Use default chunking settings
  - **Yes**: Customize chunking method, size, and overlap

### If User Selects "No" (Default Settings)

Use the following defaults:
```javascript
{
  AdvancedSettings: false,
  // Default chunking will be applied by system:
  // ChunkingMethod: "SIZE"
  // ChunkSize: 1000
  // ChunkOverlap: 200
}
```

**Do not include `IndexParams` in pageInstructions for Step 7.**

### If User Selects "Yes" (Advanced Settings)

#### Question 2: Select Chunking Method

Use `AskUserQuestion` tool:
- **Question**: "Select the chunking method"
- **Options**:
  - **NONE**: No chunking (entire document as single chunk)
  - **TITLE**: Chunk by document titles/headers
  - **SIZE** (Recommended): Fixed-size chunks with overlap
  - **ABSTRACT**: Chunk by abstract/summary sections

**Important**: Only one method can be selected.

#### Question 3 & 4: Chunk Size and Overlap (Only if method is NOT "NONE")

If user selected TITLE, SIZE, or ABSTRACT:

Use `AskUserQuestion` tool or direct input:
- **Chunk Size**: Integer value (default: 1000, typical range: 500-5000)
- **Chunk Overlap**: Integer value (default: 200, must be < chunk size)

If user selected NONE:
- **Skip** chunk size and overlap questions
- Only ChunkingMethod: "NONE" will be sent

## Data to Capture

Store for Step 7:

**For Default Settings**:
```javascript
{
  AdvancedSettings: false
}
```

**For Advanced Settings**:
```javascript
{
  AdvancedSettings: true,
  IndexParams: {
    ChunkingMethod: "<NONE|TITLE|SIZE|ABSTRACT>",
    ChunkSize: <integer>,      // Omit if ChunkingMethod is "NONE"
    ChunkOverlap: <integer>    // Omit if ChunkingMethod is "NONE"
  }
}
```

## Next Step

Proceed to [07-submit-configuration.md](07-submit-configuration.md) to submit the configuration with pageInstructions.
