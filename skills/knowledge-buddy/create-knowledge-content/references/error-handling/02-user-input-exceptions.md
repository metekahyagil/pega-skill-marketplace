# Step 2 Exceptions: User Input Validation Failures

This document covers error handling and troubleshooting for Step 2 (Gather User Input).

**Happy Path**: [02-user-input.md](../02-user-input.md)

---

## Invalid File Path

### Problem

User provides file path that doesn't exist or is not accessible.

### Symptoms

- File not found errors
- Path doesn't exist
- Permission denied when reading file
- Invalid path format

### Solutions

#### 1. Validate File Exists

Check file existence before proceeding:

```javascript
// Validate file path
const fs = require('fs');
if (!fs.existsSync(filePath)) {
  throw new Error(`File not found: ${filePath}`);
}
```

#### 2. Check File Permissions

Verify file is readable:

```javascript
try {
  fs.accessSync(filePath, fs.constants.R_OK);
} catch (err) {
  throw new Error(`File not readable: ${filePath}`);
}
```

#### 3. Validate File Format

Ensure file format is supported:

**Supported Formats:**
- `.pdf` - PDF documents
- `.docx` - Word documents
- `.txt` - Plain text files
- `.md` - Markdown (may have extraction issues)

**Validation:**
```javascript
const path = require('path');
const ext = path.extname(filePath).toLowerCase();
const supported = ['.pdf', '.docx', '.txt', '.md'];

if (!supported.includes(ext)) {
  throw new Error(`Unsupported file format: ${ext}`);
}
```

#### 4. Ask User to Correct Path

If validation fails, re-prompt user:

```javascript
AskUserQuestion([{
  question: "File path is invalid. Please provide the absolute path to your file:",
  header: "File Path",
  options: [
    { label: "Enter path", description: "Provide correct file path" }
  ],
  multiSelect: false
}])
```

**Common Path Issues:**
- Relative paths (use absolute paths)
- Tilde expansion not working (`~/file` vs `/home/user/file`)
- Windows vs Unix path separators
- Spaces in path not escaped
- File moved or deleted after selection

---

## Invalid Chunking Parameters

### Problem

User provides invalid chunking settings in advanced mode.

### Symptoms

- Chunk overlap >= chunk size
- Negative values
- Zero or extremely small chunk size
- Unreasonably large values

### Solutions

#### 1. Validate Chunk Size

Ensure chunk size is reasonable:

```javascript
const MIN_CHUNK_SIZE = 100;
const MAX_CHUNK_SIZE = 10000;

if (chunkSize < MIN_CHUNK_SIZE || chunkSize > MAX_CHUNK_SIZE) {
  throw new Error(
    `Chunk size must be between ${MIN_CHUNK_SIZE} and ${MAX_CHUNK_SIZE}`
  );
}
```

#### 2. Validate Chunk Overlap

Ensure overlap is less than chunk size:

```javascript
if (chunkOverlap >= chunkSize) {
  throw new Error(
    `Chunk overlap (${chunkOverlap}) must be less than chunk size (${chunkSize})`
  );
}

if (chunkOverlap < 0) {
  throw new Error("Chunk overlap cannot be negative");
}
```

**Typical Ranges:**
- Chunk Size: 500-5000 characters
- Chunk Overlap: 50-500 characters
- Overlap should be 10-30% of chunk size

#### 3. Re-prompt with Corrected Values

If validation fails:

```javascript
AskUserQuestion([
  {
    question: `Chunk overlap (${chunkOverlap}) must be less than chunk size (${chunkSize}). Please provide valid values:`,
    header: "Chunk Size",
    options: [
      { label: "1000", description: "Default chunk size" },
      { label: "2000", description: "Larger chunks" },
      { label: "500", description: "Smaller chunks" }
    ],
    multiSelect: false
  },
  {
    question: "Chunk overlap (must be less than chunk size):",
    header: "Overlap",
    options: [
      { label: "200", description: "Default overlap" },
      { label: "100", description: "Less overlap" },
      { label: "300", description: "More overlap" }
    ],
    multiSelect: false
  }
])
```

---

## Empty or Missing Required Fields

### Problem

User leaves required fields empty or provides only whitespace.

### Symptoms

- Empty title or abstract
- Blank chunk content
- Null or undefined values
- Whitespace-only strings

### Solutions

#### 1. Validate Required Fields

Check all required fields are provided:

```javascript
function validateRequired(value, fieldName) {
  if (!value || value.trim().length === 0) {
    throw new Error(`${fieldName} is required and cannot be empty`);
  }
}

validateRequired(title, "Title");
validateRequired(abstract, "Abstract");
```

#### 2. Validate Minimum Length

Ensure fields meet minimum requirements:

```javascript
const MIN_TITLE_LENGTH = 5;
const MIN_ABSTRACT_LENGTH = 20;
const MIN_CHUNK_LENGTH = 10;

if (title.trim().length < MIN_TITLE_LENGTH) {
  throw new Error(`Title must be at least ${MIN_TITLE_LENGTH} characters`);
}

if (abstract.trim().length < MIN_ABSTRACT_LENGTH) {
  throw new Error(`Abstract must be at least ${MIN_ABSTRACT_LENGTH} characters`);
}
```

#### 3. Re-prompt for Missing Fields

Ask user to provide missing information:

```javascript
AskUserQuestion([{
  question: "Title cannot be empty. Please provide an article title:",
  header: "Title",
  options: [
    { label: "Enter title", description: "Provide article title" }
  ],
  multiSelect: false
}])
```

---

## Invalid Number of Chunks

### Problem

User requests too many or too few chunks for text content.

### Symptoms

- Zero chunks requested
- Extremely high number of chunks (>100)
- Number doesn't match actual content provided

### Solutions

#### 1. Enforce Reasonable Limits

Set practical limits:

```javascript
const MIN_CHUNKS = 1;
const MAX_CHUNKS = 20;  // Reasonable limit for manual entry

if (numChunks < MIN_CHUNKS || numChunks > MAX_CHUNKS) {
  throw new Error(
    `Number of chunks must be between ${MIN_CHUNKS} and ${MAX_CHUNKS}`
  );
}
```

**Recommended:**
- For manual entry: 1-10 chunks
- For simple content: 1-3 chunks
- For detailed content: 3-10 chunks

#### 2. Suggest Appropriate Number

Guide user based on content:

```javascript
AskUserQuestion([{
  question: "How many content chunks would you like to create?",
  header: "Chunks",
  options: [
    { label: "1", description: "Single chunk (simple content)" },
    { label: "3", description: "Multiple chunks (recommended)" },
    { label: "5", description: "Many chunks (detailed content)" },
    { label: "10", description: "Maximum chunks (very detailed)" }
  ],
  multiSelect: false
}])
```

#### 3. Handle File-Based Content

For file uploads, chunks are auto-generated:

```javascript
if (contentFormat === "file") {
  // Don't ask for chunk count
  // Chunking handled by Pega based on IndexParams
  console.log("Chunks will be auto-generated from file content");
}
```

---

## Empty Chunk Content

### Problem

User provides empty or whitespace-only chunk content.

### Symptoms

- Empty strings in chunk array
- All whitespace chunks
- Single-character chunks

### Solutions

#### 1. Validate Each Chunk

Check all chunks have meaningful content:

```javascript
chunks.forEach((chunk, index) => {
  if (!chunk || chunk.trim().length < MIN_CHUNK_LENGTH) {
    throw new Error(
      `Chunk ${index + 1} is too short or empty. Minimum ${MIN_CHUNK_LENGTH} characters required.`
    );
  }
});
```

#### 2. Filter Empty Chunks

Remove empty chunks automatically:

```javascript
const validChunks = chunks.filter(chunk =>
  chunk && chunk.trim().length >= MIN_CHUNK_LENGTH
);

if (validChunks.length === 0) {
  throw new Error("No valid chunks provided. All chunks are empty.");
}
```

#### 3. Re-collect Chunk Content

If chunks are invalid, ask again:

```javascript
AskUserQuestion([{
  question: `Chunk ${index + 1} content (must be at least ${MIN_CHUNK_LENGTH} characters):`,
  header: `Chunk ${index + 1}`,
  options: [
    { label: "Enter content", description: "Provide chunk content" }
  ],
  multiSelect: false
}])
```

---

## No Access Roles Selected

### Problem

User doesn't select any access roles, or selection is invalid.

### Symptoms

- Empty access role array
- Invalid role names
- Null or undefined selection

### Solutions

#### 1. Use Default Role

Fall back to default if no selection:

```javascript
const DEFAULT_ROLE = "KnowledgeBuddy:Public";

if (!accessRoles || accessRoles.length === 0) {
  console.log(`No access roles selected. Using default: ${DEFAULT_ROLE}`);
  accessRoles = [DEFAULT_ROLE];
}
```

#### 2. Validate Role Names

Ensure selected roles exist:

```javascript
const availableRoles = await queryAccessRoles();  // From Step 1
const availableRoleNames = availableRoles.map(r => r.AccessRoleName);

const invalidRoles = accessRoles.filter(
  role => !availableRoleNames.includes(role)
);

if (invalidRoles.length > 0) {
  throw new Error(`Invalid access roles: ${invalidRoles.join(", ")}`);
}
```

#### 3. Present Multi-Select Clearly

Ensure user understands multi-select:

```javascript
AskUserQuestion([{
  question: "Which access role(s) should have access to this content? (You can select multiple)",
  header: "Access Roles",
  options: accessRoles.map(role => ({
    label: role.pyLabel || role.AccessRoleName,
    description: `Grant access to ${role.AccessRoleName}`
  })),
  multiSelect: true  // Enable multi-select
}])
```

---

## AskUserQuestion Limitations

### Problem

Hitting the 4-question limit per call or complex input requirements.

### Symptoms

- Too many questions for single call
- Need conditional questions
- Complex validation requirements
- Multi-step input flow needed

### Solutions

#### 1. Break Into Multiple Calls

Split questions across multiple `AskUserQuestion` calls:

```javascript
// Call 1: Initial configuration
const config = await AskUserQuestion([
  { question: "Collection?", ... },
  { question: "Data source?", ... },
  { question: "Settings mode?", ... },
  { question: "Content format?", ... }
]);

// Call 2: Conditional on settings mode
if (config.settingsMode === "Advanced") {
  const chunking = await AskUserQuestion([
    { question: "Chunking method?", ... },
    { question: "Chunk size?", ... },
    { question: "Chunk overlap?", ... }
  ]);
}

// Call 3: Content details
const content = await AskUserQuestion([
  { question: "Access roles?", ... },
  { question: "Title?", ... },
  { question: "Abstract?", ... }
]);
```

#### 2. Collect Complex Input via "Other" Option

For free-form text or complex input:

```javascript
AskUserQuestion([{
  question: "Provide chunk content:",
  header: "Content",
  options: [
    { label: "Short", description: "Brief content (< 500 chars)" },
    { label: "Medium", description: "Medium content (500-1000 chars)" },
    { label: "Long", description: "Long content (> 1000 chars)" }
    // User can choose "Other" to enter custom text
  ],
  multiSelect: false
}])
```

#### 3. Use Reasonable Defaults

Reduce questions by using smart defaults:

```javascript
// Instead of asking every parameter
const chunkingDefaults = {
  method: "SIZE",
  size: 1000,
  overlap: 200
};

// Only ask if user selects "Advanced"
```

---

## File Size Issues

### Problem

File is too large to upload or process efficiently.

### Symptoms

- Upload timeout
- Memory errors
- Processing takes too long
- API limits exceeded

### Solutions

#### 1. Check File Size

Validate file size before upload:

```javascript
const fs = require('fs');
const stats = fs.statSync(filePath);
const fileSizeInMB = stats.size / (1024 * 1024);

const MAX_FILE_SIZE_MB = 50;  // Adjust based on system limits

if (fileSizeInMB > MAX_FILE_SIZE_MB) {
  throw new Error(
    `File size (${fileSizeInMB.toFixed(2)} MB) exceeds maximum (${MAX_FILE_SIZE_MB} MB)`
  );
}
```

#### 2. Inform User of Limits

Set expectations upfront:

```javascript
AskUserQuestion([{
  question: `Provide file path (max ${MAX_FILE_SIZE_MB} MB):`,
  header: "File Path",
  options: [
    { label: "Enter path", description: "Absolute path to file" }
  ],
  multiSelect: false
}])
```

#### 3. Suggest Alternatives

For large files:
- Split into multiple smaller documents
- Use text mode with manual chunks
- Compress or optimize file
- Use external processing

---

## Troubleshooting Checklist

When user input fails validation, verify:

- [ ] File path is absolute and exists
- [ ] File format is supported (.pdf, .docx, .txt)
- [ ] File has read permissions
- [ ] File size is within limits
- [ ] Chunk size > chunk overlap
- [ ] Chunk parameters are positive integers
- [ ] Title and abstract are not empty
- [ ] Title meets minimum length (≥5 chars)
- [ ] Abstract meets minimum length (≥20 chars)
- [ ] Number of chunks is reasonable (1-20)
- [ ] All chunks have content (≥10 chars each)
- [ ] At least one access role selected (or default used)
- [ ] Access role names are valid

---

## Validation Summary

**Required Validations:**

| Field | Validation |
|-------|------------|
| Title | Non-empty, ≥5 characters |
| Abstract | Non-empty, ≥20 characters |
| File Path | Exists, readable, supported format |
| File Size | ≤50 MB (or system limit) |
| Chunk Size | 100-10000, integer |
| Chunk Overlap | 0 to (chunk size - 1), integer |
| Chunks Count | 1-20 for manual entry |
| Chunk Content | Non-empty, ≥10 characters each |
| Access Roles | At least one, or use default |

---

## Related Documentation

- **Happy Path**: [02-user-input.md](../02-user-input.md)
- **Previous Step Errors**: [01-query-data-exceptions.md](./01-query-data-exceptions.md)
- **Next Step Errors**: [03-create-case-exceptions.md](./03-create-case-exceptions.md)
- **Error Handling Index**: [index.md](./index.md)
