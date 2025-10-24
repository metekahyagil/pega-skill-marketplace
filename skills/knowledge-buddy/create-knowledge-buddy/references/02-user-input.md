# Step 2: Gather User Input

## Purpose

Collect user preferences for configuration mode, buddy details, and clone option using `AskUserQuestion` tool.

## Required Information

### 1. Configuration Mode

Ask: "Which configuration mode?"
- **Simple (Recommended)**: Default settings, fastest setup
- **Advanced**: Full customization

### 2. Buddy Details

Ask: "Buddy name and description?"
- **Name** (required): Unique identifier
- **Description** (recommended): Purpose and usage

### 3. Clone Option

Ask: "Create new or clone existing?"
- **Create new**: Start from scratch
- **Clone from**: Select from existing buddies (queried in Step 1)

If cloning, capture the original buddy ID.

## Advanced Mode: Additional Questions

If Advanced mode selected, gather additional preferences later in workflow:

- **Step 4**: Access role customization
- **Step 5**: Prompt review and editing
- **Step 6**: Context data collection selection
- **Step 7**: GenAI model selection

## Implementation Tool

Use `AskUserQuestion` with appropriate options based on data from Step 1.

## Output

Store user selections:
- `isAdvanced`: boolean
- `buddyName`: string
- `buddyDescription`: string
- `isCloned`: boolean
- `originalBuddyId`: string (if cloning)

## Next Step

[03-create-case.md](03-create-case.md)
