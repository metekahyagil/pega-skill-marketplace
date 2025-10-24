# Step 1 Exceptions: Query Data Failures

This document covers error handling and troubleshooting for Step 1 (Query Available Data).

**Happy Path**: [01-query-data.md](../01-query-data.md)

---

## Data View Query Failures

### Problem

Queries to data views (D_IndexList, D_DataSourceList, D_BuddyAccessRoleList) fail when calling `get_list_data_view`.

### Symptoms

- Error messages about data view not found
- Query execution failures
- Timeout errors
- 400 or 500 HTTP errors

### Solutions

#### 1. Verify Data View Exists

Ensure the data view is available in your Pega instance:

**Data Views Required:**
- `D_IndexList` - For collections
- `D_DataSourceList` - For data sources
- `D_BuddyAccessRoleList` - For access roles

**How to Check:**
- Verify in Pega: Dev Studio → Data Model → Data View
- Confirm Knowledge Buddy application is installed
- Check ruleset access

**Common Issues:**
- Knowledge Buddy application not installed
- Data views not exposed via DX API
- Incorrect data view ID (case sensitive)
- Ruleset not accessible

#### 2. Check Authentication

Verify connection is established:

**Prerequisites:**
- Step 0 authentication must succeed
- Valid access token obtained
- Permissions to query data views

**Solutions:**
- Re-run authentication: `mcp__pega-dx-mcp__authenticate_pega({})`
- Check token hasn't expired
- Verify API access permissions

#### 3. Verify Query Structure

Ensure query parameters are correctly formatted:

**Correct Structure:**
```javascript
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

**Common Mistakes:**
- Wrong data view ID casing
- Invalid field names in select
- Missing query or paging objects
- Invalid pageSize value

---

## Empty Result Sets

### Problem

Data view queries succeed but return no results.

### Symptoms

- Empty data array in response
- No collections, data sources, or roles returned
- Valid query structure but zero results

### Solutions

#### 1. Verify Data Exists

Check if entities exist in Pega:

**For Collections:**
- Navigate to Knowledge Buddy → Collections
- Verify at least one collection exists
- Check collection status (must be Open/Active)

**For Data Sources:**
- Navigate to Knowledge Buddy → Data Sources
- Verify at least one data source exists
- Check data source status (must be Open/Active)
- Confirm data source is associated with a collection

**For Access Roles:**
- Navigate to Designer Studio → Security → Access Roles
- Verify Knowledge Buddy roles exist
- Check if roles are active

#### 2. Check Filtering Conditions

If using query filters, verify they don't exclude all results:

```javascript
// May return empty if filter too restrictive
query: {
  select: [...],
  filter: {
    filterConditions: [
      { field: "pyStatusWork", comparator: "EQ", value: "Open" }
    ]
  }
}
```

**Solutions:**
- Remove filters temporarily to test
- Verify filter values match data
- Check comparator logic

#### 3. Verify Permissions

Ensure user has permission to view entities:

**Access Requirements:**
- Read access to Knowledge Buddy work objects
- Access to appropriate access groups
- Data visibility rules not restricting results

---

## Permission Errors

### Problem

Query fails due to insufficient permissions.

### Symptoms

- 403 Forbidden errors
- "Access denied" messages
- Empty results despite data existing
- Permission-related error messages

### Solutions

#### 1. Verify DX API Permissions

Check OAuth client has required permissions:

**Required Access:**
- Read access to PegaFW-QnA-Work class
- Read access to PegaFW-KB-Work class
- Data view access via DX API
- Access to Knowledge Buddy application

**How to Check:**
- Review OAuth client registration in Pega
- Verify access group assignments
- Check privilege mappings

#### 2. Check Data Access Rules

Ensure When rules don't block access:

**Common Restrictions:**
- Organization-based access rules
- Work pool restrictions
- Status-based filtering
- Custom security rules

**Solutions:**
- Review When rules on data views
- Check access control policies
- Verify operator's access group
- Test with administrative credentials

#### 3. Verify Ruleset Access

Confirm access to required rulesets:

**Required Rulesets:**
- PegaFW-QnA (Knowledge Buddy framework)
- PegaFW-KB (Knowledge base)
- Application rulesets

---

## Invalid Field Names

### Problem

Query fails due to invalid field names in select clause.

### Symptoms

- Error about unknown property
- Field not found errors
- Query syntax errors

### Solutions

#### 1. Use Correct Field Names

Verify field names match data view definition:

**Collections (D_IndexList):**
- `CollectionName` ✅
- `pyID` ✅
- `pyLabel` ✅
- `pyStatusWork` ✅
- `pzInsKey` ⚠️ (computed, may not be selectable)

**Data Sources (D_DataSourceList):**
- `Name` ✅
- `pyID` ✅
- `pyLabel` ✅
- `CollectionName` ✅
- `pyStatusWork` ✅

**Access Roles (D_BuddyAccessRoleList):**
- `AccessRoleName` ✅
- `pyLabel` ✅

#### 2. Compute pzInsKey Locally

Don't query pzInsKey directly; compute it:

```javascript
// After query, compute pzInsKey
const pzInsKey = `PEGAFW-QNA-WORK ${pyID}`;
```

**Format:**
- Collection: `"PEGAFW-QNA-WORK DC-1"`
- Data Source: `"PEGAFW-QNA-WORK SRC-1001"`

---

## Network and Timeout Issues

### Problem

Query times out or network errors occur.

### Symptoms

- Timeout errors
- Connection reset
- Network unreachable
- Slow response times

### Solutions

#### 1. Check Network Connectivity

Verify connection to Pega instance:

```bash
# Test basic connectivity
ping your-instance.pega.com

# Test HTTPS
curl -I https://your-instance.pega.com/prweb/api/v1/
```

#### 2. Optimize Page Size

Reduce page size if queries are timing out:

```javascript
// Instead of
paging: { pageSize: 1000 }

// Try
paging: { pageSize: 50 }
```

#### 3. Check System Load

Verify Pega instance performance:
- System resource usage
- Database performance
- Network latency
- Other concurrent operations

---

## No Collections or Data Sources Available

### Problem

Queries succeed but no collections or data sources exist to choose from.

### Symptoms

- Empty options for user selection
- Cannot proceed with workflow
- Fresh Pega instance with no data

### Solutions

#### 1. Create Initial Collection

User must create collection first:

**In Pega:**
1. Navigate to Knowledge Buddy → Collections
2. Create new collection
3. Provide name and configuration
4. Save and activate

**Via API** (if supported):
- Create collection case
- Configure collection settings
- Activate collection

#### 2. Create Initial Data Source

User must create data source:

**In Pega:**
1. Navigate to Knowledge Buddy → Data Sources
2. Create new data source
3. Associate with collection
4. Configure settings
5. Save and activate

#### 3. Inform User

If no entities exist, inform user to create them:

```
⚠️ No collections found in the system.

Before creating content, you need to:
1. Create at least one collection
2. Create at least one data source
3. Associate data source with collection

Please create these in Pega Knowledge Buddy first.
```

---

## Malformed Response Data

### Problem

Query returns data but structure is unexpected.

### Symptoms

- Missing expected fields
- Null or undefined values
- Data type mismatches
- Nested structure issues

### Solutions

#### 1. Check API Version

Verify DX API version compatibility:
- Data view schema may differ by version
- Field names may change
- Structure may vary

#### 2. Handle Missing Fields

Add defensive checks:

```javascript
const collections = response.data || [];
collections.forEach(collection => {
  const name = collection.CollectionName || "Unknown";
  const id = collection.pyID || "N/A";
  // Handle missing fields gracefully
});
```

#### 3. Validate Response

Check response before processing:

```javascript
if (!response || !response.data || !Array.isArray(response.data)) {
  // Handle invalid response
  throw new Error("Invalid data view response");
}

if (response.data.length === 0) {
  // Handle empty results
  console.log("No collections found");
}
```

---

## Troubleshooting Checklist

When data queries fail, verify:

- [ ] Authentication succeeded in Step 0
- [ ] Data views exist in Pega instance
- [ ] Knowledge Buddy application is installed
- [ ] Collections exist in system (at least one)
- [ ] Data sources exist in system (at least one)
- [ ] Access roles are configured
- [ ] Data view IDs are correct (case sensitive)
- [ ] Field names in select are valid
- [ ] User has permission to query data views
- [ ] Network connectivity is stable
- [ ] Page size is reasonable
- [ ] Response data structure is valid

---

## Fallback Options

If queries continue to fail:

### Option 1: Manual Input

Allow user to manually enter collection/data source details:

```javascript
AskUserQuestion([
  { question: "Enter collection name:", ... },
  { question: "Enter collection ID (pyID):", ... }
])
```

### Option 2: Use Defaults

If environment has known collections:

```javascript
// Provide known defaults
const defaultCollection = {
  CollectionName: "knowledge",
  pyID: "DC-1",
  pzInsKey: "PEGAFW-QNA-WORK DC-1"
};
```

### Option 3: Retry with Backoff

Implement retry logic for transient failures:

```javascript
let retries = 3;
while (retries > 0) {
  try {
    const result = await get_list_data_view({...});
    break;
  } catch (error) {
    retries--;
    await sleep(1000 * (4 - retries));
  }
}
```

---

## Related Documentation

- **Happy Path**: [01-query-data.md](../01-query-data.md)
- **Previous Step Errors**: [00-connection-exceptions.md](./00-connection-exceptions.md)
- **Next Step Errors**: [02-user-input-exceptions.md](./02-user-input-exceptions.md)
- **Error Handling Index**: [index.md](./index.md)
