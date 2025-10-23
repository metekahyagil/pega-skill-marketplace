# Step 4 Exceptions: Case Configuration Failures

This document covers error handling and troubleshooting for Step 4 (Configure Collection, Data Source, and Access).

**Happy Path**: [04-configure-case.md](../04-configure-case.md)

---

## 1. Empty Collection/Datasource

### Problem

Collection or Datasource fields remain empty after performing the Create action.

### Symptoms

- CollectionName is empty or undefined after Step 4
- DataSourceName is empty or undefined after Step 4
- Fields not populated in case data
- Verification shows null or missing values

### Root Cause

Embedded pages (Collection, Datasource) **cannot be set via content properties**. They MUST be set using pageInstructions.

---

### Solutions

#### ✅ Verify pageInstructions are used (not content properties)

**Wrong Approach** (will not work):
```javascript
content: {
  Collection: { CollectionName: "knowledge" },  // ❌ Won't work
  Datasource: { Name: "test" }                  // ❌ Won't work
}
```

**Correct Approach** (use pageInstructions):
```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".Collection",
    content: {
      CollectionName: "knowledge",
      pyID: "DC-1",
      pzInsKey: "PEGAFW-QNA-WORK DC-1"
    }
  }
]
```

#### ✅ Check target paths start with a dot

**Wrong**:
```javascript
{ target: "Collection" }          // ❌ Missing dot
{ target: "Datasource" }          // ❌ Missing dot
```

**Correct**:
```javascript
{ target: ".Collection" }         // ✅ With dot
{ target: ".Datasource" }         // ✅ With dot
```

**Why**: The dot prefix indicates an embedded page property in Pega's data model.

#### ✅ Use UPDATE instruction (not REPLACE)

**Wrong**:
```javascript
{ instruction: "REPLACE", target: ".Collection", ... }  // ❌ Removes existing fields
```

**Correct**:
```javascript
{ instruction: "UPDATE", target: ".Collection", ... }   // ✅ Preserves structure
```

**Why**: UPDATE adds/updates fields while preserving the existing embedded page structure. REPLACE would remove all existing fields and replace with only what you provide.

#### ✅ Include all three properties

Both Collection and Datasource require **complete references**:

**Collection requires**:
- `CollectionName` - The collection identifier (e.g., "knowledge")
- `pyID` - The collection ID (e.g., "DC-1")
- `pzInsKey` - The instance key (e.g., "PEGAFW-QNA-WORK DC-1")

**Datasource requires**:
- `Name` - The data source identifier (e.g., "Knowledge_ingestion_test")
- `pyID` - The data source ID (e.g., "SRC-1001")
- `pzInsKey` - The instance key (e.g., "PEGAFW-QNA-WORK SRC-1001")

**Incomplete Example** (will fail):
```javascript
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge"  // ❌ Missing pyID and pzInsKey
  }
}
```

**Complete Example** (correct):
```javascript
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge",
    pyID: "DC-1",
    pzInsKey: "PEGAFW-QNA-WORK DC-1"  // ✅ All three properties
  }
}
```

#### ✅ Verify pzInsKey format has space

The `pzInsKey` format is critical: `"CLASS-NAME ID"` with **exactly one space** between class name and ID.

**Wrong Formats**:
```javascript
pzInsKey: "PEGAFW-QNA-WORKDC-1"          // ❌ No space
pzInsKey: "PEGAFW-QNA-WORK  DC-1"        // ❌ Double space
pzInsKey: "PEGAFW-QNA-WORK DC-1 "        // ❌ Trailing space
pzInsKey: " PEGAFW-QNA-WORK DC-1"        // ❌ Leading space
```

**Correct Format**:
```javascript
pzInsKey: "PEGAFW-QNA-WORK DC-1"         // ✅ Single space
```

**How to construct**:
```javascript
// For Collection
const pzInsKey = `PEGAFW-QNA-WORK ${pyID}`;  // e.g., "PEGAFW-QNA-WORK DC-1"

// For Datasource
const pzInsKey = `PEGAFW-QNA-WORK ${pyID}`;  // e.g., "PEGAFW-QNA-WORK SRC-1001"
```

---

## 2. BAD_REQUEST Error (400)

### Problem

HTTP 400 error when calling `perform_assignment_action` for the Create action.

### Symptoms

- Request fails with "BAD_REQUEST" or 400 status code
- Error message may mention "invalid parameters" or "invalid structure"
- Action fails before executing

### Common Causes

#### ❌ Missing dot prefix in target path

```javascript
{ target: "Collection" }          // Wrong
{ target: ".Collection" }         // Correct
```

#### ❌ Trying to set embedded pages via content

```javascript
// Wrong
content: {
  Collection: { CollectionName: "knowledge" }
}

// Correct
pageInstructions: [{
  instruction: "UPDATE",
  target: ".Collection",
  content: { CollectionName: "knowledge", pyID: "DC-1", pzInsKey: "..." }
}]
```

#### ❌ Incorrect pzInsKey format

```javascript
pzInsKey: "PEGAFW-QNA-WORKDC-1"   // Wrong - no space
pzInsKey: "PEGAFW-QNA-WORK DC-1"  // Correct - with space
```

#### ❌ Missing required fields (pyID or pzInsKey)

```javascript
// Wrong - incomplete
{
  CollectionName: "knowledge"
}

// Correct - all fields
{
  CollectionName: "knowledge",
  pyID: "DC-1",
  pzInsKey: "PEGAFW-QNA-WORK DC-1"
}
```

#### ❌ Wrong instruction type

```javascript
{ instruction: "APPEND", target: ".Collection" }   // Wrong - Collection is not a list
{ instruction: "UPDATE", target: ".Collection" }   // Correct
```

#### ❌ Invalid property names

```javascript
{
  CollectionID: "DC-1"  // Wrong property name
}

{
  pyID: "DC-1"         // Correct property name
}
```

---

## 3. Empty ContentAccessConfigurations

### Problem

Access roles are not set after Create action.

### Symptoms

- ContentAccessConfigurations field is empty
- No access roles configured
- Content may not be accessible after publishing

### Root Cause

ContentAccessConfigurations is a **page list** (array), not a single embedded page. Must use APPEND instruction.

### Solutions

#### ✅ Use APPEND instruction (not UPDATE)

**Wrong**:
```javascript
{
  instruction: "UPDATE",  // ❌ UPDATE doesn't add to list
  target: ".ContentAccessConfigurations",
  content: { AccessRoleName: "Knowledge Buddy Public" }
}
```

**Correct**:
```javascript
{
  instruction: "APPEND",  // ✅ APPEND adds to list
  target: ".ContentAccessConfigurations",
  content: { AccessRoleName: "Knowledge Buddy Public" }
}
```

#### ✅ Target must start with dot

```javascript
{ target: ".ContentAccessConfigurations" }  // ✅ Correct
{ target: "ContentAccessConfigurations" }   // ❌ Wrong
```

#### ✅ Each object must have AccessRoleName property

```javascript
// Wrong
{ content: { Role: "Knowledge Buddy Public" } }

// Correct
{ content: { AccessRoleName: "Knowledge Buddy Public" } }
```

#### ✅ Multiple roles need multiple APPEND instructions

```javascript
pageInstructions: [
  {
    instruction: "APPEND",
    target: ".ContentAccessConfigurations",
    content: { AccessRoleName: "Knowledge Buddy Public" }
  },
  {
    instruction: "APPEND",  // Second APPEND for second role
    target: ".ContentAccessConfigurations",
    content: { AccessRoleName: "Internal Users" }
  }
]
```

---

## 4. Empty IndexParams (Advanced Settings)

### Problem

Custom chunking parameters not set when Advanced settings selected.

### Symptoms

- IndexParams is empty or undefined
- Using default chunking instead of custom values
- AdvancedSettings flag set but no custom parameters

### Solutions

#### ✅ Set AdvancedSettings: true in content

```javascript
content: {
  AdvancedSettings: true  // Required to enable advanced UI
}
```

#### ✅ Use UPDATE instruction for IndexParams

```javascript
{
  instruction: "UPDATE",
  target: ".IndexParams",
  content: {
    ChunkingMethod: "SIZE",
    ChunkSize: 1000,
    ChunkOverlap: 200
  }
}
```

#### ✅ All three IndexParams properties required

```javascript
// Wrong - incomplete
{
  ChunkingMethod: "SIZE"
}

// Correct - all three
{
  ChunkingMethod: "SIZE",
  ChunkSize: 1000,
  ChunkOverlap: 200
}
```

#### ✅ Validate values

- **ChunkingMethod**: Must be one of: "TITLE", "SIZE", "NONE", "ABSTRACT"
- **ChunkSize**: Must be positive integer (typically 500-5000)
- **ChunkOverlap**: Must be positive integer, LESS than ChunkSize

---

## 5. Assignment Action Fails

### Problem

The perform_assignment_action call itself fails.

### Symptoms

- Error executing action
- Assignment not found
- Action not available

### Solutions

#### ✅ Verify assignmentID is correct

The assignmentID should be from Step 3 (case creation):
```
ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT
```

**Common Issues**:
- Using caseID instead of assignmentID
- Wrong assignment from different case
- Typo in assignment ID
- Assignment already completed

#### ✅ Verify actionID is "Create"

```javascript
actionID: "Create"  // Must be exactly "Create"
```

**Case sensitive**: "create" or "CREATE" will not work.

#### ✅ Case must be in Create stage

If case has already progressed past Create stage, you cannot perform the Create action again.

**Check case status**:
```javascript
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });
console.log(caseData.pyStatusWork);  // Should be in Create stage
```

---

## Troubleshooting Checklist

When configuration fails, verify:

- [ ] Using **pageInstructions** for Collection and Datasource (not content)
- [ ] All target paths start with **dot** (`.Collection`, `.Datasource`)
- [ ] Using **UPDATE** instruction for embedded pages
- [ ] Using **APPEND** instruction for page lists
- [ ] **All three properties** provided for Collection (CollectionName, pyID, pzInsKey)
- [ ] **All three properties** provided for Datasource (Name, pyID, pzInsKey)
- [ ] **pzInsKey format** is correct: `"CLASS-NAME ID"` with single space
- [ ] Assignment ID is correct from Step 3
- [ ] Action ID is exactly "Create"
- [ ] Case is still in Create stage
- [ ] For advanced settings: AdvancedSettings: true and IndexParams provided

---

## Verification After Configuration

After Step 4 completes, verify configuration was successful:

```javascript
// Get case data
const caseData = await mcp__pega-dx-mcp__get_case({ caseID: "KB-1023" });

// Verify Collection
console.log("Collection:", caseData.CollectionName);  // Should not be empty
console.log("Vector Store:", caseData.VectorStoreCollectionName);  // Should match collection

// Verify Datasource
console.log("Data Source:", caseData.DataSourceName);  // Should not be empty

// Get detailed view
const createView = await mcp__pega-dx-mcp__get_case_view({
  caseID: "KB-1023",
  viewID: "Create"
});

console.log("Collection details:", createView.Collection);
console.log("Datasource details:", createView.Datasource);
console.log("IndexParams:", createView.IndexParams);  // If advanced settings
console.log("Access:", createView.ContentAccessConfigurations);
```

Expected results:
- CollectionName and VectorStoreCollectionName populated
- DataSourceName populated
- ContentAccessConfigurations shows `[object Object]`
- IndexParams populated if Advanced settings used

---

## Related Documentation

- **Happy Path**: [04-configure-case.md](../04-configure-case.md)
- **Next Step**: [05-author-content.md](../05-author-content.md)
- **Error Handling Index**: [index.md](./index.md)
- **Technical Details**: [technical-details.md](../technical-details.md)
