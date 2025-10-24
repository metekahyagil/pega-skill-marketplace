# Technical Details

## Case Type

- **ID**: `PegaFW-QnA-Work-KnowledgeQnA`
- **Business ID**: `BUDDY-XXXX`
- **Full Case ID**: `PEGAFW-QNA-WORK BUDDY-XXXX`

## Stage Lifecycle

```
Create (PRIM0)
  ↓ Submit Create assignment
Secure (PRIM3)
  ↓ Submit ConfigureSecurity assignment
Prompt (PRIM1)
  ↓ Submit ConfigurePrompt assignment
Resolve (PRIM2) - Auto-completes
```

## Assignment Format

```
ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-XXXX!<FLOW-NAME>
```

- **Create**: `!CREATEFORM_DEFAULT`
- **Secure**: `!SECURE_FLOW`
- **Prompt**: `!PROMPT_FLOW`

## Key Data Structures

### BuddyAccessConfigurations (Page List)

Access control entries:
- `BuddyAccessType`: "manage" | "use" | "view"
- `AccessRoleName`: Role identifier (e.g., "KnowledgeBuddy:BuddyManager")

**Default**: 2 entries (manage for BuddyManager, use for Public)

### ContextDataConfigs (Page List)

Context data source definitions:
- `Name`: Variable name (e.g., "SEARCHRESULTS")
- `Collection`: { pyID, pzInsKey } reference
- `MaxChunk`: Max chunks (default: 5)
- `MaxChunkTotalSize`: Max size (default: 5000)
- `MinSimilarityScore`: 0-100 (default: 80)
- `MustReturnResults`: boolean (default: true)

**Default**: 1 entry (SEARCHRESULTS with empty collection)

## PageInstructions Patterns

### Critical Rules

1. **Target paths MUST start with dot**: `.BuddyAccessConfigurations`
2. **listIndex is 1-based**: First item = 1
3. **Empty content keeps defaults**: `content: {}` preserves existing values
4. **Instruction types**:
   - `UPDATE`: Modify existing item at index
   - `APPEND`: Add new item to end
   - `REPLACE`: Replace entire page

### UPDATE Example

```javascript
{
  instruction: "UPDATE",
  target: ".ContextDataConfigs",
  listIndex: 1,
  content: {
    Name: "SEARCHRESULTS",
    MaxChunk: 10
  }
}
```

### APPEND Example

```javascript
{
  instruction: "APPEND",
  target: ".BuddyAccessConfigurations",
  content: {
    BuddyAccessType: "view",
    AccessRoleName: "KnowledgeBuddy:Internal"
  }
}
```

## Default Initialization

Pega initializes defaults on stage transitions:

- **Create → Secure**: Initializes BuddyAccessConfigurations (2 entries)
- **Secure → Prompt**: Initializes SystemMessage, UserMessage, ContextDataConfigs (1 entry)

## Common Values

### Assignment Actions

- **Create stage**: `Create`
- **Secure stage**: `ConfigureSecurity`
- **Prompt stage**: `ConfigurePrompt`

### Default Access Roles

- `KnowledgeBuddy:BuddyManager` (manage)
- `KnowledgeBuddy:Public` (use)

### Context Defaults

- SEARCHRESULTS: MaxChunk: 5, MaxChunkTotalSize: 5000, MinSimilarityScore: 80

## MCP Tools Reference

- **authenticate_pega**: Verify connection
- **get_list_data_view**: Query data pages
- **create_case**: Create new case
- **perform_assignment_action**: Submit assignments
- **get_case**: Retrieve case details

See [response-structures.md](response-structures.md) for response formats.
