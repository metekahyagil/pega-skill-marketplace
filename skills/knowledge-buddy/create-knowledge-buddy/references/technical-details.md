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
   - `INSERT`: Add new item at index (shifts subsequent items)
   - `DELETE`: Remove item at index (shifts subsequent items)
   - `DELETE ALL`: Remove all items from nested page list
   - `APPEND`: Add new item to end (deprecated, use INSERT)
   - `REPLACE`: Replace entire page

### Pattern: Keep Existing Items (UPDATE with empty content)

Use for default configurations:

```javascript
{
  instruction: "UPDATE",
  target: ".BuddyAccessConfigurations",
  listIndex: 1,
  content: {}  // Empty = keep existing values
}
```

---

### Pattern: Modify Existing Item (UPDATE with content)

```javascript
{
  instruction: "UPDATE",
  target: ".ContextDataConfigs",
  listIndex: 1,
  content: {
    Name: "SEARCHRESULTS",
    MaxChunk: 10,
    MinSimilarityScore: 85
  }
}
```

---

### Pattern: Dynamic List - DELETE + INSERT + UPDATE

**Use case**: Replace all items with custom set

**Steps**:
1. DELETE existing items (repeat for each)
2. INSERT empty items at indices
3. UPDATE each item with values

**Example**: Replace 2 default access configs with 3 custom configs

```javascript
pageInstructions: [
  // Step 1: Delete existing defaults
  { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },
  { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },

  // Step 2: Insert empty items
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 1, instruction: "INSERT" },
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 2, instruction: "INSERT" },
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 3, instruction: "INSERT" },

  // Step 3: Update with values
  { content: { BuddyAccessType: "use" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },
  { content: { BuddyAccessType: "view" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },
  { content: { BuddyAccessType: "manage" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" },

  // Step 4: Add roles
  { content: { AccessRoleName: "KnowledgeBuddy:Admin" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },
  { content: { AccessRoleName: "KnowledgeBuddy:Internal" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },
  { content: { AccessRoleName: "KnowledgeBuddy:Public" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" }
]
```

**Why this pattern?**
- Ensures exact number of items (not dependent on defaults)
- Explicit control over all values
- Predictable state after execution

---

### Pattern: Nested Page List - DELETE ALL + INSERT

**Use case**: Configure nested lists (e.g., DataSources, ResponseAttributes)

**Steps**:
1. DELETE ALL items in nested list
2. INSERT new items at indices

**Example**: Configure data sources for context

```javascript
pageInstructions: [
  // Step 1: Clear existing data sources
  {
    instruction: "DELETE ALL",
    target: ".ContextDataConfigs(1).DataSources"
  },

  // Step 2: Insert new data sources
  {
    target: ".ContextDataConfigs(1).DataSources",
    content: { pyID: "SRC-2001" },
    listIndex: 1,
    instruction: "INSERT"
  },
  {
    target: ".ContextDataConfigs(1).DataSources",
    content: { pyID: "SRC-2" },
    listIndex: 2,
    instruction: "INSERT"
  }
]
```

**Note**: Parentheses notation `(1)` references item at index 1 of parent list

---

### Pattern: Multiple Contexts

**Use case**: Add additional context definitions beyond default SEARCHRESULTS

**Steps**:
1. UPDATE first context (SEARCHRESULTS)
2. INSERT empty item for second context
3. UPDATE second context with values
4. Configure nested lists for each context

**Example**: Two contexts with different collections

```javascript
pageInstructions: [
  // Context 1: Update SEARCHRESULTS
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: { pyID: "DC-1" },
      MaxChunk: 5
    }
  },

  // Context 2: Insert and configure
  {
    target: ".ContextDataConfigs",
    content: {},
    listIndex: 2,
    instruction: "INSERT"
  },
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 2,
    content: {
      Name: "ADDITIONAL_CONTEXT",
      Collection: { pyID: "DC-2" },
      MaxChunk: 3
    }
  },

  // Configure data sources for each context
  { instruction: "DELETE ALL", target: ".ContextDataConfigs(1).DataSources" },
  { target: ".ContextDataConfigs(1).DataSources", content: { pyID: "SRC-1" }, listIndex: 1, instruction: "INSERT" },

  { instruction: "DELETE ALL", target: ".ContextDataConfigs(2).DataSources" },
  { target: ".ContextDataConfigs(2).DataSources", content: { pyID: "SRC-2" }, listIndex: 1, instruction: "INSERT" }
]
```

---

### Pattern Reference Table

| Operation | Use Case | Steps |
|-----------|----------|-------|
| **Keep defaults** | Preserve existing values | UPDATE with `content: {}` |
| **Modify item** | Change specific fields | UPDATE with partial content |
| **Replace all items** | Dynamic number of configs | DELETE existing → INSERT empty → UPDATE values |
| **Configure nested list** | DataSources, Attributes | DELETE ALL → INSERT items |
| **Add context** | Multiple contexts | INSERT empty → UPDATE values |

---

### Index Behavior

**DELETE shifts indices down**:
```
Before DELETE at index 1: [item1, item2, item3]
After DELETE at index 1:  [item2, item3]
After DELETE at index 1:  [item3]
```

**INSERT shifts indices up**:
```
Before INSERT at index 2: [item1, item3]
After INSERT at index 2:  [item1, empty, item3]
After UPDATE at index 2:  [item1, item2, item3]
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
