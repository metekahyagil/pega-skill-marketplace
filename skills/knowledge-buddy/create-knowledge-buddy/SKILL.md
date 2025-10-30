---
name: create-knowledge-buddy
description: Create and configure Knowledge Buddy cases in Pega Knowledge Buddy applications through an interactive step-by-step workflow. Use when creating new knowledge buddies, cloning existing configurations, or setting up GenAI-powered Q&A assistants with custom prompts and context data.
---

# Create Knowledge Buddy

## Overview

Interactive workflow for creating Knowledge Buddy cases in Pega Knowledge Buddy applications. Guides users through each configuration step with focused questions, querying data only when needed.

## When to Use This Skill

- Creating new knowledge buddies for Q&A applications
- Cloning existing buddy configurations
- Setting up GenAI-powered assistants with custom prompts
- Configuring context data sources for semantic search
- User requests like "Create a new knowledge buddy" or "Set up a Q&A assistant"

## Workflow

### Step 0: Verify Pega Connection

Authenticate with Pega before starting workflow.

**Reference**: [00-connection.md](references/00-connection.md)

---

### Step 1: Create Knowledge Buddy Case

Create case using `PegaFW-QnA-Work-KnowledgeQnA` case type. Capture caseID and first assignmentID.

**Reference**: [03-create-case.md](references/03-create-case.md)

---

### Step 2: Ask Clone or New

**Single focused question**: "Do you want to use configuration from an existing buddy?"

- **Yes**: Query D_OriginalBuddyList → Proceed to Step 3a
- **No**: Proceed to Step 3b

**Reference**: [01-on-demand-queries.md](references/01-on-demand-queries.md)

---

### Step 3a: Gather Input for Cloning

**Single AskUserQuestion with multiple questions**:
1. Question 1: "What is the buddy name?"
2. Question 2: "What is the buddy description?"
3. Question 3: "Which existing buddy to clone from?" (show list from D_OriginalBuddyList)

**Reference**: [02a-initial-input-clone.md](references/02a-initial-input-clone.md)

---

### Step 3b: Gather Input for New Buddy

**Single AskUserQuestion with multiple questions**:
1. Question 1: "What is the buddy name?"
2. Question 2: "What is the buddy description?"

**Reference**: [02b-initial-input-new.md](references/02b-initial-input-new.md)

---

### Step 4: Submit Create Action and Refresh

Submit Create action with Name, Description, IsCloned flag, and OriginalBuddyId (if cloning). Then refresh case to get updated assignment state. Case transitions from Create → Secure stage. Capture Security assignment ID.

**Content Field Names** (use exactly as shown):
- `Name` (string: buddy name)
- `pyDescription` (string: buddy description, optional)
- `IsCloned` (boolean: true for clone, false for new)
- `OriginalBuddyId` (string: required only if IsCloned=true, e.g., "BUDDY-1")

**Reference**: [03-create-case.md](references/03-create-case.md)

---

### Step 5: Configure Access Control

**Single focused question**: "Do you want default access configuration or custom?"

- **Default**: Keep 2 default access configs → [04a-configure-access-default.md](references/04a-configure-access-default.md)
- **Custom**: Query D_BuddyAccessRoleList → Proceed to Step 5a

**Default configs**: "manage" for Admin role, "use" for Public role

---

### Step 5a: Configure Custom Access (if selected)

**Atomic interactions**:
1. Ask: "How many access configurations?" (user specifies number)
2. For each configuration, ask:
   - "Select access type" (manage / use / view)
   - "Select role" (from D_BuddyAccessRoleList)

Build pageInstructions with INSERT + UPDATE operations for each config.

**Property Names** (use exactly as shown):
- `BuddyAccessType` (string: "manage", "use", or "view")
- `AccessRoleName` (string: role name from query, e.g., "KnowledgeBuddy:Admin")

**Reference**: [04b-configure-access-custom.md](references/04b-configure-access-custom.md)

---

### Step 6: Submit Security Assignment

Submit ConfigureSecurity assignment with access configurations. Case transitions from Secure → Prompt stage. Capture Prompt assignment ID.

**Reference**: [04a-configure-access-default.md](references/04a-configure-access-default.md) or [04b-configure-access-custom.md](references/04b-configure-access-custom.md)

---

### Step 7: Configure Prompts

**Single AskUserQuestion with 2 questions**:
1. Question 1: "Do you want to use default Instructions or enter custom?"
   - Default: Use SystemMessage from default-prompts.md
   - Custom: User enters custom instructions

2. Question 2: "Do you want to use default Information or enter custom?"
   - Default: Use UserMessage from default-prompts.md
   - Custom: User enters custom information template

**Reference**: [05-configure-prompts.md](references/05-configure-prompts.md), [default-prompts.md](references/default-prompts.md)

---

### Step 8: Configure Context Data

**Single focused question**: "How do you want to configure context data?"

- **Option 1 - Use defaults**: SEARCHRESULTS context with default values → [06a-configure-context-default.md](references/06a-configure-context-default.md)
- **Option 2 - Edit default SEARCHRESULTS**: Modify the default SEARCHRESULTS context → Proceed to Step 8a
- **Option 3 - Add custom contexts**: Add additional contexts beyond SEARCHRESULTS (can add multiple) → Proceed to Step 8b

**Important Property Names** (use exactly as shown in reference files):
- `Name` (context name, e.g., "SEARCHRESULTS")
- `Collection: { pyID: "DC-1" }` (collection reference)
- `MustReturnResults` (boolean)
- `MaxChunk` (number)
- `MaxChunkTotalSize` (number)
- `MinSimilarityScore` (number)
- `DataSourcesCSV` (string, empty)
- `ResponseAttributesCSV` (string, empty)

---

### Step 8a: Edit Default SEARCHRESULTS Context (if selected)

**Query on-demand**:
- D_IndexList → Present collections
- D_DataSourceListByCollection (after collection selected) → Present data sources
- D_AttributeList (after collection selected) → Present response attributes

**CRITICAL Query Parameter**: When calling D_DataSourceListByCollection, use `dataViewParameters: { pyID: "DC-1", Available: "Yes" }` - the parameter name is `pyID`, NOT `CollectionID`.

**Atomic questions**:
1. "Select collection"
2. "How do you want to select data sources?" (Select all / Select none / Enter specific data sources)
3. "How do you want to select response attributes?" (Select all / Select none / Enter specific attributes)
4. "Do you want advanced settings?" → If yes, ask:
   - "Search must return results?" (true/false, default: true)
   - "Maximum number of chunks?" (default: 5)
   - "Maximum total size of chunks?" (default: 5000)
   - "Minimum similarity score?" (default: 80)

**Reference**: [06b-configure-context-edit-default.md](references/06b-configure-context-edit-default.md)

---

### Step 8b: Add Custom Contexts (if selected)

**Question**: "How many additional contexts do you want to add?" (can add multiple beyond SEARCHRESULTS)

**CRITICAL Query Parameter**: When calling D_DataSourceListByCollection for each context, use `dataViewParameters: { pyID: "DC-X", Available: "Yes" }` - the parameter name is `pyID`, NOT `CollectionID`.

**For each additional context**, ask:
1. "Context name?" (e.g., ADDITIONAL_CONTEXT, EXTRA_DOCS)
2. "Select collection"
3. "How do you want to select data sources?" (Select all / Select none / Enter specific data sources)
4. "How do you want to select response attributes?" (Select all / Select none / Enter specific attributes)
5. "Do you want advanced settings?" (same options as Step 8a)

Build pageInstructions with SEARCHRESULTS (index 1) + additional contexts (index 2, 3, ...).

**Reference**: [06c-configure-context-add-custom.md](references/06c-configure-context-add-custom.md)

---

### Step 9: Configure GenAI Model

**Single focused question**: "Do you want to use default GenAI model or select specific model?"

- **Default**: Leave GenAIModelId empty → [07a-configure-genai-default.md](references/07a-configure-genai-default.md)
- **Select model**: Query D_bxGenAIModelsForBuddy → Proceed to Step 9a

---

### Step 9a: Select GenAI Model (if selected)

**Query on-demand**: D_bxGenAIModelsForBuddy

**Atomic questions**:
1. "Select GenAI model" (show list)
2. If GPT model selected: "Select output format" (JSON / Custom)
3. "Apply text replacements to user requests?" (true/false)
4. "Apply auto filtering to user requests?" (true/false)

**Reference**: [07b-configure-genai-custom.md](references/07b-configure-genai-custom.md)

---

### Step 10: Submit Prompt Assignment

Submit ConfigurePrompt assignment combining:
- SystemMessage (Instructions)
- UserMessage (Information)
- ContextDataConfigs array (supports multiple contexts)
- GenAI settings (model, output format, filtering options)

Case auto-resolves after submission.

**Content Field Names** (use exactly as shown):
- `SystemMessage` (string)
- `UserMessage` (string)
- `GenAIModelId` (string, empty for default)
- `OutputFormat` (string, "Custom" or "JSON")
- `IncludeTextReplacements` (boolean)
- `IncludeAutoFiltering` (boolean)

**Important**: Use `perform_assignment_action` with pageInstructions for context configuration. No separate refresh needed - pageInstructions are processed atomically with content in a single submission.

**Reference**: [08-submit-prompt.md](references/08-submit-prompt.md)

---

### Step 11: Verify Success

Retrieve final case status, verify Resolved-Completed, report case ID and buddy URL to user.

**Reference**: [09-verify-success.md](references/09-verify-success.md)

---

## Technical References

### Core Patterns
- **[technical-details.md](references/technical-details.md)** - Case types, pageInstructions patterns (INSERT/DELETE/UPDATE), data structures, MCP tools

### On-Demand Queries
- **[01-on-demand-queries.md](references/01-on-demand-queries.md)** - When and how to query each data view (D_OriginalBuddyList, D_BuddyAccessRoleList, D_IndexList, etc.)

### Default Configuration
- **[default-prompts.md](references/default-prompts.md)** - Default SystemMessage and UserMessage templates

### Error Handling
- **[error-handling/index.md](references/error-handling/index.md)** - Common issues and solutions

### Working Examples
- **[examples.md](references/examples.md)** - Complete working example with cloning, custom access, multiple contexts, and GenAI selection

---

## Key Principles

1. **Grouped Related Inputs**: Use single AskUserQuestion with multiple questions for related data (name + description)
2. **On-Demand Queries**: Query data views only when user selects that path
3. **Progressive Disclosure**: Show advanced options only when requested
4. **Token Efficiency**: Minimal prompts, focused references

## Next Steps After Creation

- Buddy available in Knowledge Buddy portal
- Can answer questions based on configured context
- Editable via "Edit configuration" action
- Clonable via "Create from existing" option
