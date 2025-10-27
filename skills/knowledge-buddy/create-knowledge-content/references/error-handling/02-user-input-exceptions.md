# Step 2 Exceptions: User Input Failures

## AskUserQuestion Errors

### Symptom
`AskUserQuestion` tool fails or user provides invalid input.

### Common Causes
- Too many questions (>4) in single call
- Missing required question fields
- Invalid option structures
- User cancels or provides no input

### Fix
1. Limit to 4 questions per call
2. Ensure all required fields present: question, header, options, multiSelect
3. Each option needs label and description
4. Handle user cancellation gracefully

## Input Validation Errors

### Symptom
User-provided values are invalid for workflow.

### Common Issues

**Invalid Chunking Parameters**:
- ChunkOverlap >= ChunkSize (must be less than)
- Negative values
- Non-integer values

**Invalid File Path**:
- File doesn't exist
- No read permissions
- Unsupported format (not PDF, DOCX, TXT, MD)

**Empty Required Fields**:
- Title/Abstract empty
- No chunks provided for text mode
- No file path for file mode
- No access roles selected (should default to "KnowledgeBuddy:Public")

### Fix
1. Validate chunk overlap < chunk size
2. Verify file exists before proceeding: check with file system
3. Ensure title and abstract non-empty
4. For text mode: require at least 1 chunk
5. For file mode: validate file path and format
6. Default to "KnowledgeBuddy:Public" if no roles selected

## Selection Errors

### Symptom
User selections don't match queried data.

### Fix
- Present only options from Step 1 queries
- Store collection/data source with all three: name, pyID, pzInsKey
- Validate selected IDs exist in queried data
- Handle case where no collections/data sources available (inform user to create them first)

## Troubleshooting Checklist

- [ ] Maximum 4 questions per AskUserQuestion call
- [ ] All question fields populated
- [ ] Options have label + description
- [ ] Chunking: overlap < size
- [ ] File mode: file exists and supported format
- [ ] Text mode: at least 1 chunk
- [ ] Selections match Step 1 query results

**Happy Path**: [02-user-input.md](../02-user-input.md)
