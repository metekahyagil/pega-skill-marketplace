# Step 2: Gather User Input

After discovering available collections and data sources, collect user preferences and content details using the `AskUserQuestion` tool.

## Quick Checklist

- [ ] Ask initial configuration questions (collection, data source, settings mode, content format)
- [ ] If Advanced settings: Ask chunking configuration questions
- [ ] Ask content details questions (access roles, title, abstract)
- [ ] For text content: Collect chunk content
- [ ] For file content: Collect and validate file path
- [ ] Verify all required data captured
- [ ] Proceed to Step 3 (Create Case)

---

## Table of Contents

- [Overview](#overview)
- [Step 2a: Initial Configuration Questions](#step-2a-initial-configuration-questions)
  - [1. Collection Selection](#1-collection-selection)
  - [2. Data Source Selection](#2-data-source-selection)
  - [3. Settings Mode](#3-settings-mode)
  - [4. Content Format](#4-content-format)
- [Step 2b: Advanced Settings Questions (Conditional)](#step-2b-advanced-settings-questions-conditional)
- [Step 2c: Content Details Questions](#step-2c-content-details-questions)
  - [Access Roles](#access-roles-required-for-both-formats)
  - [For Text Content](#for-text-content)
  - [For File Content](#for-file-content)
- [Example: Simple Settings + Text Content](#example-simple-settings--text-content)
- [Example: Advanced Settings + File Content](#example-advanced-settings--file-content)
- [Data to Capture](#data-to-capture)
- [Next Step](#next-step)

---

## Overview

Due to the 4-question limit per `AskUserQuestion` call, this may require 2-3 calls depending on user choices.

## Step 2a: Initial Configuration Questions

Ask these questions in the first `AskUserQuestion` call:

### 1. Collection Selection
Present available collections from Step 1 (01-query-data.md) as options.

### 2. Data Source Selection
Present available data sources from Step 1, optionally filtered by selected collection.

### 3. Settings Mode
Ask if user wants **Simple** or **Advanced** settings:

- **Simple** (Recommended): Use default chunking settings
  - ChunkingMethod: SIZE
  - ChunkSize: 1000
  - ChunkOverlap: 200

- **Advanced**: Configure custom chunking parameters
  - User can customize all chunking settings

### 4. Content Format
Ask whether content will be **Text** or **File**:

- **Text**: Manual text entry with support for multiple chunks
- **File**: Upload document (PDF, DOCX, TXT) for automatic extraction

## Step 2b: Advanced Settings Questions (Conditional)

**Only if user selected "Advanced" settings**, ask in a second `AskUserQuestion` call:

### 1. Chunking Method
Options:
- **TITLE**: Chunks content by document title/sections
- **SIZE** (default): Chunks content by fixed character/token size
- **NONE**: No chunking, treats content as single unit
- **ABSTRACT**: Chunks based on abstract/summary structure

### 2. Chunk Size
Integer value for chunk size:
- Default: 1000
- Typical range: 500-5000 characters

### 3. Chunk Overlap
Integer value for overlap between chunks:
- Default: 200
- Typical range: 50-500 characters
- **Must be less than chunk size**

**Tip**: Present default values as suggestions that users can accept or modify.

## Step 2c: Content Details Questions

In the final `AskUserQuestion` call, first ask about access roles (applies to both formats), then ask content-specific questions.

### Access Roles (Required for Both Formats)

1. **Access Roles**: Which role(s) should have access to this content?
   - Present roles from Step 1 (01-query-data.md Step 3)
   - Enable **multi-select** (user can choose multiple roles)
   - Default: "KnowledgeBuddy:Public" if no selection
   - Common options:
     - KnowledgeBuddy:Public - Public access
     - KnowledgeBuddy:Internal - Internal users only
     - KnowledgeBuddy:Author - Content authors
     - KnowledgeBuddy:BuddyManager - Knowledge base managers
     - KnowledgeBuddy:DataSourceManager - Data source administrators
     - KnowledgeBuddy:Admin - Full administrative access

### For Text Content

2. **Title**: Article title
3. **Abstract**: Article summary/description
4. **Number of Chunks**: How many content chunks to create (1-10)
5. **Content for each chunk**: Ask for each chunk separately or provide instructions for multi-chunk input

**Note**: Collect all chunk content from user input.

### For File Content

2. **Title**: Article title
3. **Abstract**: Article summary/description
4. **File Path**: Absolute path to the file on the system (e.g., `/path/to/document.pdf`)

**Important**: Validate file path exists before proceeding with upload workflow.

## Example: Simple Settings + Text Content

```javascript
// First call - all 4 questions
AskUserQuestion([
  { question: "Which collection?", options: [...] },
  { question: "Which data source?", options: [...] },
  { question: "Settings mode?", options: ["Simple", "Advanced"] },
  { question: "Content format?", options: ["Text", "File"] }
])

// Second call - content details for text
AskUserQuestion([
  { question: "Article title?", ... },
  { question: "Article abstract?", ... },
  { question: "Number of chunks?", options: ["1", "2", "3", ...] }
])
// Then collect chunk content based on number
```

## Example: Advanced Settings + File Content

```javascript
// First call - all 4 questions
AskUserQuestion([
  { question: "Which collection?", options: [...] },
  { question: "Which data source?", options: [...] },
  { question: "Settings mode?", options: ["Simple", "Advanced"] },
  { question: "Content format?", options: ["Text", "File"] }
])

// Second call - advanced chunking settings
AskUserQuestion([
  { question: "Chunking method?", options: ["TITLE", "SIZE", "NONE", "ABSTRACT"] },
  { question: "Chunk size?", ... },
  { question: "Chunk overlap?", ... }
])

// Third call - file content details
AskUserQuestion([
  { question: "Article title?", ... },
  { question: "Article abstract?", ... },
  { question: "File path?", ... }
])
```

## Data to Capture

After gathering all user input, you should have:

### From Step 1 (Query Data)
- Collection: `CollectionName`, `pyID`, `pzInsKey`
- Data Source: `Name`, `pyID`, `pzInsKey`
- Access Roles: Array of available `AccessRoleName` values

### From Step 2a
- Settings mode: "Simple" or "Advanced"
- Content format: "Text" or "File"

### From Step 2b (if Advanced)
- `ChunkingMethod`: "TITLE" | "SIZE" | "NONE" | "ABSTRACT"
- `ChunkSize`: integer
- `ChunkOverlap`: integer

### From Step 2c
- `AccessRoles`: Array of selected role names (e.g., ["KnowledgeBuddy:Public", "KnowledgeBuddy:Internal"])
- `Title`: string
- `Abstract`: string
- **For Text**: Array of chunk content strings
- **For File**: File path string

## Next Step

After gathering all user input, proceed to **[03-create-case.md](./03-create-case.md)** to create the Content case.

---

**Exceptions**: See **[error-handling/02-user-input-exceptions.md](error-handling/02-user-input-exceptions.md)** for input validation failures and troubleshooting.
