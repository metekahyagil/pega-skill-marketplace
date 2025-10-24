# Create Knowledge Buddy Skill

Create and configure Knowledge Buddy cases in Pega Knowledge Buddy applications with support for simple (default settings) and advanced (full customization) configuration modes.

## Overview

This skill enables Claude Code to create Knowledge Buddies in Pega's Knowledge Buddy application. A Knowledge Buddy is a GenAI-powered conversational assistant that answers questions based on context data from knowledge base collections using Retrieval-Augmented Generation (RAG).

The skill supports two configuration modes:
- **Simple Mode**: Quick setup with default settings (~30 seconds)
- **Advanced Mode**: Full customization of all settings (~2-3 minutes)

The workflow follows Pega's 4-stage case lifecycle:
1. **Create**: Basic buddy information (name, description, clone option)
2. **Secure**: Access control configuration (who can manage/use the buddy)
3. **Prompt**: GenAI prompts and context data source configuration
4. **Resolve**: Automatic resolution upon completion

## Features

- **Two Configuration Modes**: Simple for quick setup, Advanced for full control
- **Clone Existing Buddies**: Replicate configurations from existing buddies
- **Guided Workflow**: Step-by-step case creation with automatic stage transitions
- **Access Control**: Configure who can manage, use, or view the buddy
- **GenAI Configuration**: Set up custom system instructions and user prompts
- **Context Data**: Link buddies to knowledge base collections for RAG
- **Model Selection**: Choose from available GenAI models (including Pega-Default)
- **Default Values**: Intelligent defaults for quick buddy creation
- **Verification**: End-to-end validation of buddy configuration

## Prerequisites

### Required

- **Pega Instance**: Knowledge Buddy application installed (Pega 8.8+)
- **MCP Server**: `@marco-looy/pega-dx-mcp` configured in Claude Code
- **Authentication**: Valid Pega credentials with appropriate roles
- **User Roles**: `KnowledgeBuddy:Author` or higher

### Optional

- Existing knowledge base collections (for linking context data)
- Custom GenAI model configurations
- Existing buddies to clone from

## Installation

This skill is part of the Pega Skill Marketplace. Install using one of these methods:

### Method 1: Marketplace Command (Recommended)

```bash
/plugin marketplace add https://github.com/metekahyagil/pega-skill-marketplace
```

### Method 2: User Skills

Copy to your user skills directory:

```bash
cp -r skills/knowledge-buddy/create-knowledge-buddy ~/.claude/skills/pega-knowledge-buddy/
```

### Method 3: Project Skills

Copy to your project's skills directory:

```bash
cp -r skills/knowledge-buddy/create-knowledge-buddy .claude/skills/pega-knowledge-buddy/
```

## Usage

### Slash Command

```bash
/create-knowledge-buddy
```

### Natural Language

Claude will automatically invoke this skill when you ask to create a Knowledge Buddy:

```
"Create a new Knowledge Buddy for customer support"
"Set up a GenAI buddy for product documentation"
"I need to create a buddy that answers HR policy questions"
"Clone the existing Product Helper buddy"
```

## Configuration Modes

### Simple Mode (Default)

Perfect for quick buddy creation with sensible defaults:

**What's included:**
- Default access control (KnowledgeBuddy:BuddyManager + KnowledgeBuddy:Public)
- Default GenAI prompts (customer service representative template)
- Empty collection reference (can be configured later)
- Default GenAI model (Pega-Default)
- Custom output format
- No text replacements or auto-filtering

**Setup time:** ~30 seconds

### Advanced Mode

Full control over all buddy settings:

**What you can customize:**
- Custom access roles and permissions
- Review and edit GenAI prompts (Instructions and Information)
- Select specific collection for context data
- Configure context parameters (MaxChunk, MaxChunkTotalSize, MinSimilarityScore)
- Add multiple context definitions
- Select specific GenAI model
- Choose output format (Custom or JSON)
- Enable text replacements
- Enable auto-filtering

**Setup time:** ~2-3 minutes

## Configuration

### MCP Server Setup

Ensure `@marco-looy/pega-dx-mcp` is configured in your Claude Code settings:

```json
{
  "mcpServers": {
    "pega-dx-mcp": {
      "command": "npx",
      "args": ["-y", "@marco-looy/pega-dx-mcp"],
      "env": {
        "PEGA_URL": "https://your-pega-instance.com",
        "PEGA_USERNAME": "your-username",
        "PEGA_PASSWORD": "your-password"
      }
    }
  }
}
```

### Buddy Configuration Options

#### Basic Information (Both Modes)
- **Name** (required): Unique name for the buddy
- **Description** (recommended): Purpose and usage description
- **Clone Option**: Copy settings from an existing buddy

#### Access Control
**Simple Mode**: Uses defaults
- Manage: `KnowledgeBuddy:BuddyManager`
- Use: `KnowledgeBuddy:Public`

**Advanced Mode**: Select custom roles from:
- `KnowledgeBuddy:Admin` - Buddy administrator
- `KnowledgeBuddy:DataSourceManager` - Data source manager
- `KnowledgeBuddy:Internal` - Internal users only
- `KnowledgeBuddy:Author` - Knowledge buddy author
- `KnowledgeBuddy:BuddyManager` - Knowledge buddy manager
- `KnowledgeBuddy:Public` - Public access

#### GenAI Prompts
**Simple Mode**: Uses defaults (see [default-prompts.md](./references/default-prompts.md))

**Advanced Mode**: Review and customize:
- **SystemMessage (Instructions)**: LLM behavior constraints and guidelines
- **UserMessage (Information)**: Template with {SEARCHRESULTS} and {QUESTION} placeholders

#### Context Data Configuration
**Simple Mode**: Empty collection reference (configure later)

**Advanced Mode**: Configure SEARCHRESULTS context:
- **Collection**: Select from available collections (D_IndexList)
- **MaxChunk**: Maximum chunks to return (default: 5)
- **MaxChunkTotalSize**: Maximum total size of chunks (default: 5000)
- **MinSimilarityScore**: Similarity threshold 0-100 (default: 80)
- **MustReturnResults**: Whether results are required (default: true)
- **Additional Contexts**: Option to add more context definitions

#### GenAI Model Selection
**Simple Mode**: Uses Pega-Default model

**Advanced Mode**: Select from available models:
- **Pega-Default** models:
  - Apply text replacements to user request (checkbox)
  - Apply auto filtering to user request (checkbox)
- **Other models**:
  - Choose output format (Custom or JSON)
  - Apply text replacements to user request (checkbox)
  - Apply auto filtering to user request (checkbox)

## Examples

### Example 1: Simple Mode - Quick Setup

```
User: Create a Knowledge Buddy called "Customer Support Buddy"

Claude will:
1. Authenticate with Pega
2. Query available data (collections, models, roles, existing buddies)
3. Create case PegaFW-QnA-Work-KnowledgeQnA
4. Set name to "Customer Support Buddy"
5. Use default access control (Manager + Public)
6. Use default GenAI prompts
7. Configure empty collection (for later setup)
8. Use default GenAI settings
9. Verify buddy created successfully (status: Resolved-Completed)
10. Report: "Knowledge Buddy 'Customer Support Buddy' created successfully (BUDDY-1234)"
```

### Example 2: Advanced Mode - Full Customization

```
User: Create a buddy called "Product Helper" with advanced settings

Claude will:
1. Authenticate with Pega
2. Query available data
3. Ask for settings mode → User selects "Advanced"
4. Ask for name, description → User provides details
5. Ask for clone option → User selects "No"
6. Create case
7. Submit basic info
8. Ask for access roles → User customizes or keeps defaults
9. Submit security configuration
10. Present default prompts for review → User reviews/edits
11. Ask for collection selection → User selects "Product Documentation"
12. Ask for context parameters → User customizes or keeps defaults
13. Ask for GenAI model → User selects specific model
14. Ask for output format and settings → User configures
15. Submit prompt configuration with all customizations
16. Verify buddy created successfully
17. Report full configuration details and buddy ID
```

### Example 3: Clone Existing Buddy

```
User: Clone the existing "HR Assistant" buddy

Claude will:
1. Query existing buddies (D_OriginalBuddyList)
2. Find "HR Assistant" buddy
3. Ask for new buddy name
4. Create case with IsCloned=true and OriginalBuddyId set
5. Pega copies all settings from source buddy
6. Verify and report new buddy ID
```

## Workflow Details

### Step 0: Verify Connection
- Authenticate with `mcp__pega-dx-mcp__authenticate_pega`
- Verify server connectivity

### Step 1: Query Available Data
Query Pega data pages to present options:
- **D_IndexList**: Available collections
- **D_bxGenAIModelsForBuddy**: Available GenAI models
- **D_BuddyAccessRoleList**: Available access roles
- **D_OriginalBuddyList**: Existing buddies (for cloning)

### Step 2: Gather User Input
Use `AskUserQuestion` tool to collect:
- Settings mode (Simple/Advanced)
- Buddy name and description
- Clone option and source buddy (if cloning)
- Advanced settings (if Advanced mode selected)

### Step 3: Create Case
- Call `create_case` with `PegaFW-QnA-Work-KnowledgeQnA`
- Capture case ID and Create assignment ID
- Case enters Create stage (PRIM0)

### Step 4: Configure Security (Secure Stage)
**Submit Create Assignment:**
- Name, Description, IsCloned flag
- Case auto-transitions to Secure stage (PRIM3)

**Submit Configure Security Assignment:**
- Simple: Keep default access configs using empty UPDATE pageInstructions
- Advanced: Apply custom access roles using UPDATE pageInstructions
- Case auto-transitions to Prompt stage (PRIM1)

### Step 5: Configure Prompts (Prompt Stage)
**Simple Mode:**
- Use default SystemMessage and UserMessage
- Keep default ContextDataConfigs with empty Collection
- Use default GenAI settings (empty ModelId, Custom format)
- Submit with minimal changes

**Advanced Mode:**
- Step 5a: Review and edit prompts
- Step 5b (Step 6): Configure context data (collection, parameters)
- Step 5c (Step 7): Configure GenAI model and settings
- Submit with full customizations using UPDATE pageInstructions

Case auto-transitions to Resolve stage (PRIM2) and resolves to Resolved-Completed

### Step 6: Verify Success
- Call `get_case` to retrieve final status
- Verify status is "Resolved-Completed"
- Check all configurations applied correctly
- Report buddy ID (BUDDY-XXXX) and URL to user

## Technical Details

### Case Type
- **Case Type ID**: `PegaFW-QnA-Work-KnowledgeQnA`
- **Business ID Format**: `BUDDY-XXXX`
- **Full Case ID Format**: `PEGAFW-QNA-WORK BUDDY-XXXX`

### Assignment Actions
- **Create Stage**: `Create` action
- **Secure Stage**: `ConfigureSecurity` action
- **Prompt Stage**: `ConfigurePrompt` action

### Key Data Structures

**BuddyAccessConfigurations** (Page List):
```typescript
{
  classID: "PegaFW-QnA-Data-AccessConfig",
  BuddyAccessType: "manage" | "use" | "view",
  AccessRoleName: string,
  pyID: string
}
```

**ContextDataConfigs** (Page List):
```typescript
{
  classID: "PegaFW-QnA-Data-ContextDataConfig",
  Name: string,
  Collection: {
    pyID: string,
    CollectionName: string,
    pzInsKey: string
  },
  MustReturnResults: boolean,
  MaxChunk: number,
  MaxChunkTotalSize: number,
  MinSimilarityScore: number,
  DataSourcesCSV: string,
  ResponseAttributesCSV: string
}
```

### PageInstructions Pattern

This skill uses Pega's `pageInstructions` API for complex data manipulation:
- **Target paths** must start with dot: `.BuddyAccessConfigurations`
- **listIndex** is 1-based (first item = 1, not 0)
- **UPDATE** instruction modifies existing items
- **APPEND** instruction adds new items
- **Empty content** keeps existing values: `content: {}`

Example:
```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: { pyID: "DC-123" },
      MaxChunk: 5
    }
  }
]
```

## Troubleshooting

### Authentication Issues

**Problem**: MCP server connection fails

**Solutions**:
- Verify `@marco-looy/pega-dx-mcp` is installed
- Check Pega URL, username, password in MCP config
- Test connection with: `mcp__pega-dx-mcp__authenticate_pega`

### Case Creation Issues

**Problem**: 404 Not Found for case type

**Solution**: Verify Knowledge Buddy application is installed in Pega instance

**Problem**: 403 Forbidden

**Solution**: Ensure user has `KnowledgeBuddy:Author` role or higher

### Configuration Issues

**Problem**: pageInstructions not applied

**Solutions**:
- Verify target path starts with dot: `.ContextDataConfigs`
- Check listIndex is 1-based (not 0-based)
- Ensure instruction type is correct (UPDATE, APPEND)

**Problem**: Empty collection reference

**Solution**: This is expected in Simple mode. Link collection later using "Edit configuration" action, or use Advanced mode to configure during creation.

**Problem**: Stage transition failed

**Solution**: Check assignment was submitted correctly with all required fields. Review error messages in response.

### Performance Issues

**Problem**: Slow buddy responses

**Solutions**:
- Reduce MaxChunk parameter (default: 5)
- Reduce MaxChunkTotalSize parameter (default: 5000)
- Adjust MinSimilarityScore threshold (default: 80)
- Optimize collection indexing

## Best Practices

1. **Start Simple**: Use Simple mode for initial setup, switch to Advanced for specific requirements
2. **Descriptive Names**: Use clear, descriptive names for buddies
3. **Appropriate Access**: Configure access control based on security requirements
4. **Test Prompts**: Test custom prompts with sample questions
5. **Collection Setup**: Link to appropriate collections with relevant content
6. **Similarity Tuning**: Adjust MinSimilarityScore based on collection quality (70-90 recommended)
7. **Context Limits**: Set appropriate MaxChunk (3-10) and MaxChunkTotalSize (3000-10000)
8. **Documentation**: Add clear descriptions for buddy purpose
9. **Clone When Possible**: Clone existing buddies to maintain consistency
10. **Model Selection**: Choose appropriate GenAI model based on use case and budget

## Related Skills

- **create-knowledge-content**: Create knowledge base articles and content for buddy context
- **query-knowledge-base**: Query existing knowledge base content

## Resources

- [SKILL.md](./SKILL.md) - Complete skill definition and workflow
- [Technical Details](./references/technical-details.md) - API patterns and data structures
- [Default Prompts](./references/default-prompts.md) - Default SystemMessage and UserMessage templates
- [Examples](./references/examples.md) - Complete working examples
- [Error Handling](./references/error-handling.md) - Common issues and solutions
- [Dictionary](./dictionary.md) - Tools, data pages, and classes reference

## Support

For issues or questions:

1. Check the troubleshooting section above
2. Review reference documentation in `references/` directory
3. Open an issue on the GitHub repository
4. Consult Pega Knowledge Buddy documentation

## License

MIT License - See repository LICENSE file for details.

## Contributing

Contributions welcome! Please:
1. Follow the existing skill structure
2. Update documentation for changes
3. Test with real Pega instances
4. Submit pull requests with clear descriptions

## Changelog

### v1.0.0
- Initial release
- Support for Simple and Advanced configuration modes
- Clone existing buddies functionality
- Access control configuration
- GenAI prompt setup with defaults
- Context data configuration with collection selection
- GenAI model selection (including Pega-Default)
- Complete verification workflow
- Comprehensive error handling
