# Step 6 Exceptions: Verification Failures

## Case Retrieval Errors

### Symptom
`get_case` fails or returns incomplete data.

### Fix
- Verify caseID format: `"KB-####"`
- Confirm authentication valid
- Check user has view permission for case
- Use exact caseID from Step 3

## Ingestion Failures

### Symptom
Case status shows error or remains "Pending-Ingestion".

### Common Issues

**Status: "Pending-Ingestion" (Long Time)**:
- Content still processing (wait longer)
- Check IndexStatus.status field
- Large files take longer

**Status: "Open" or "Resolved-Error"**:
- Ingestion failed
- Check IngestionErrorMessage field
- Common: Invalid chunk format, file extraction failed, vector store connection issue

**Missing Collection/DataSource**:
- CollectionName empty: collection not associated
- DataSourceName empty: data source not configured
- Review Step 4 pageInstructions

### Fix
1. Check pyStatusWork for case status
2. If "Pending-Ingestion": wait and re-check
3. If error status: check IngestionErrorMessage
4. Verify IndexStatus.status: "Completed"
5. Confirm CollectionName and DataSourceName populated

## Content Verification Errors

### Symptom
Content missing or incomplete in case data.

### Issues by Mode

**Text Mode**:
- Chunks page list empty: Step 5 pageInstructions failed
- Content property missing: wrong field name used
- ArticleType not "text": check Step 5 content parameter

**File Mode**:
- ContentAttachment empty: 3-step process not followed
- No file attachments: upload or refresh step failed
- ArticleType not "file": check Step 5 content parameter

### Fix
1. Use `get_case_view` with "AuthorContent" viewID for detailed data
2. For text: verify Chunks array populated
3. For file: verify ContentAttachment and `get_case_attachments` returns file
4. Check ArticleType matches expected mode

## Access Configuration Missing

### Symptom
ContentAccessConfigurations empty.

### Fix
- Review Step 4 APPEND instructions for access roles
- Verify at least one role added
- Default should be "KnowledgeBuddy:Public"

## IndexParams Missing (Advanced Mode)

### Symptom
Advanced mode selected but IndexParams not saved.

### Fix
- Verify AdvancedSettings: true in Step 4 content
- Confirm IndexParams UPDATE pageInstruction included
- Check ChunkingMethod, ChunkSize, ChunkOverlap values

## Troubleshooting Checklist

- [ ] Case status: "Resolved-Published" or "Pending-Ingestion"
- [ ] CollectionName populated
- [ ] DataSourceName populated
- [ ] Title and Abstract present
- [ ] ContentAccessConfigurations populated
- [ ] Text mode: Chunks array present
- [ ] File mode: ContentAttachment present
- [ ] Advanced mode: IndexParams present
- [ ] No IngestionErrorMessage

**Happy Path**: [06-verify-success.md](../06-verify-success.md)

**Full Criteria**: [success-criteria.md](../success-criteria.md)
