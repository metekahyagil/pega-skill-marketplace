# Step 1 Exceptions: Query Data Failures

## Data View Query Errors

### Symptom
`get_list_data_view` fails with 400/500 errors or "data view not found".

### Common Causes

**Data View Not Available**:
- Knowledge Buddy application not installed
- Data view not exposed via DX API
- Incorrect data view ID (case-sensitive: `D_IndexList`, `D_ContentAccessDataSourcesByCollection`, `D_BuddyAccessRoleList`)

**Query Structure Invalid**:
```javascript
// Correct structure
{
  dataViewID: "D_IndexList",  // Exact case
  query: {
    select: [
      { field: "CollectionName" },
      { field: "pyID" }
    ]
  },
  paging: { pageSize: 100 }
}
```

**Authentication Issues**:
- Token expired (re-run `authenticate_pega`)
- No permission to query data views

### Fix
1. Verify Knowledge Buddy app installed
2. Check data view ID spelling (case-sensitive)
3. Confirm authentication valid
4. Verify field names in select clause

## Empty Result Sets

### Symptom
Query succeeds but returns empty array.

### Common Causes
- No collections/data sources created yet
- Filters too restrictive (e.g., `CollectionID` filter in data source query)
- User lacks access to view data

### Fix
1. Create collections/data sources in Pega UI first
2. Remove/adjust filters (e.g., set `CollectionID: ""` for all data sources)
3. Verify user access permissions
4. Query without filters to confirm data exists

## Field Access Errors

### Symptom
Query fails with "field not accessible" or returns incomplete data.

### Fix
- Remove unavailable fields from select clause
- Use only documented fields: CollectionName, pyID, pzInsKey, AccessRoleName
- Check field casing (e.g., `pyID` not `PyID`)

## Troubleshooting Checklist

- [ ] Authentication successful
- [ ] Knowledge Buddy app installed
- [ ] Data view ID correct (case-sensitive)
- [ ] Query structure valid JSON
- [ ] Collections/data sources exist
- [ ] User has view permissions
- [ ] Field names correct

**Happy Path**: [01-query-data.md](../01-query-data.md)
