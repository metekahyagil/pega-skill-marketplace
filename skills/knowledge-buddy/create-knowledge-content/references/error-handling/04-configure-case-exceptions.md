# Step 4 Exceptions: Configuration Failures

## PageInstructions Errors

### Symptom
`perform_assignment_action` fails with pageInstructions errors.

### Common Pega-Specific Issues

**Target Path Missing Dot**:
```javascript
// ❌ Wrong
target: "Collection"

// ✅ Correct
target: ".Collection"
```

**Wrong Instruction Type**:
- Use `UPDATE` for embedded pages (Collection, Datasource, IndexParams)
- Use `APPEND` for page lists (ContentAccessConfigurations)
- Using wrong type causes "cannot set property" errors

**Incomplete References**:
```javascript
// ❌ Missing fields
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge"  // Missing pyID and pzInsKey!
  }
}

// ✅ Complete
{
  instruction: "UPDATE",
  target: ".Collection",
  content: {
    CollectionName: "knowledge",
    pyID: "DC-1",
    pzInsKey: "PEGAFW-QNA-WORK DC-1"
  }
}
```

**pzInsKey Format**:
```javascript
// ❌ Wrong formats
"PEGAFW-QNA-WORK-DC-1"     // No space
"PEGAFW-QNA-WORK  DC-1"    // Extra space
"DC-1 PEGAFW-QNA-WORK"     // Reversed

// ✅ Correct (single space)
"PEGAFW-QNA-WORK DC-1"
```

### Fix
1. Add dot to all targets: `.Collection`, `.Datasource`, `.IndexParams`, `.ContentAccessConfigurations`
2. Use UPDATE for embedded pages, APPEND for lists
3. Include all three fields: name, pyID, pzInsKey
4. Format pzInsKey: `"PEGAFW-QNA-WORK <ID>"` with single space

## Configuration Mode Errors

### Symptom
Advanced settings not applied or IndexParams missing.

### Fix
- Simple mode: `content: {}`, NO IndexParams
- Advanced mode: `content: { AdvancedSettings: true }`, INCLUDE IndexParams
- IndexParams only works when AdvancedSettings: true

## Access Role Errors

### Symptom
ContentAccessConfigurations not populated or invalid roles.

### Fix
- Use APPEND for each role (one instruction per role)
- Role format: `"KnowledgeBuddy:Public"` (exact casing)
- If no roles selected, default to `"KnowledgeBuddy:Public"`
- Each APPEND needs only: `{ AccessRoleName: "role-name" }`

## Assignment Transition Errors

### Symptom
Case doesn't transition to Draft stage.

### Fix
- Verify actionID is `"Create"`
- Confirm all required pageInstructions present
- Check response for new assignment ID (DRAFT_FLOW)
- If stuck in Create stage, check validation errors

## Troubleshooting Checklist

- [ ] All targets start with dot (`.Collection`)
- [ ] UPDATE for embedded pages, APPEND for lists
- [ ] Collection/Datasource have name, pyID, pzInsKey
- [ ] pzInsKey format: `"CLASS-NAME ID"` with single space
- [ ] Advanced mode: AdvancedSettings: true + IndexParams
- [ ] Simple mode: no IndexParams
- [ ] At least one access role

**Happy Path**: [04-configure-case.md](../04-configure-case.md)
