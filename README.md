# @hiddentao/claude-code-plugins

A Claude Code plugin marketplace for better dev.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add hiddentao/claude-code-plugins
```

## Available Plugins

### docs

A plugin to generate and manage documentation for a codebase.

**Install:**
```
/plugin install docs@hiddentao-plugins
```

#### Commands

##### generate-markdown-retype

Generate Markdown documentation for your codebase with [Retype](https://retype.com) compatibility.

```
/docs:generate-markdown-retype
```

**Features:**
- Interactive documentation generation with user approval at each stage
- Thorough code analysis using AI sub-agents
- Hierarchical documentation structure with index files
- Retype-compatible output with YAML frontmatter
- Links back to source code in your repository

**Process:**
1. Prompts for output location (default: `docs/`) and source location (default: `src/`)
2. Analyzes the codebase thoroughly using sub-agents
3. Proposes a documentation hierarchy for your approval
4. Generates documentation files with your approval on the writing style
5. Validates output against the codebase for accuracy

**Viewing docs:**

Install Retype globally:
```
npm install retypeapp --global
```

Then run from the documentation output folder:
```
retype start
```

### subagent

Interactive subagent picker - find and launch the right subagent for any task.

**Install:**
```
/plugin install subagent@hiddentao-plugins
```

#### Commands

##### subagent

Find and launch a subagent matching your needs.

```
/subagent:subagent <description of what you need>
```

**Examples:**
```
/subagent:subagent review my code
/subagent:subagent security audit
/subagent:subagent explore the codebase
```

Dynamically discovers all available subagent types, matches them against your query, and lets you pick which one to launch with a custom prompt.

## License

MIT
