# Step 3 Exceptions: Case Creation Failures

## Case Creation Errors

### Symptom
`create_case` fails with 400/500 errors.

### Common Causes

**Case Type Issues**:
- Case type ID incorrect: must be `PegaFW-KB-Work-Article` (case-sensitive)
- Case type not accessible
- Knowledge Buddy app not installed

**Permission Issues**:
- User lacks create permission for case type
- Access group restrictions
- Application access denied

**Content Parameter Issues**:
- Invalid content structure (should be empty `{}` for this case type)
- Extra/invalid fields in content

### Fix
1. Use exact case type ID: `"PegaFW-KB-Work-Article"`
2. Use empty content: `content: {}`
3. Verify Knowledge Buddy app installed
4. Check user has create permission
5. Confirm authentication still valid

## Assignment Extraction Errors

### Symptom
Case created but assignment ID not found in response.

### Fix
- Extract from `assignments[0].ID`
- Verify response structure
- Assignment format: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!CREATEFORM_DEFAULT`
- If no assignments, check case status and user worklist access

## Case Type Access Errors

### Symptom
"Access denied" or "Case type not found".

### Fix
- Verify `PegaFW-KB-Work-Article` accessible to user
- Check application access (Knowledge Buddy)
- Confirm ruleset access
- Verify OAuth client has case creation permissions

## Troubleshooting Checklist

- [ ] Case type ID: `PegaFW-KB-Work-Article`
- [ ] Content: `{}`
- [ ] Knowledge Buddy app installed
- [ ] User has create permission
- [ ] Authentication valid
- [ ] Assignment ID extracted

**Happy Path**: [03-create-case.md](../03-create-case.md)
