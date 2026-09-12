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

### code-review

Adversarial multi-agent review of a codebase and its recent commits.

**Install:**
```
/plugin install code-review@hiddentao-plugins
```

#### Commands

##### adversarial

Review a codebase and its recent commit history using up to 10 adversarial subagents.

```
/code-review:adversarial [path] [last N commits] [with N agents]
```

**Examples:**
```
/code-review:adversarial
/code-review:adversarial src/ last 5 commits
/code-review:adversarial packages/api --commits 3 --agents 4
/code-review:adversarial --range v1.2.0..HEAD --agents 6
```

**Features:**
- Two rounds of subagents - independent reviewers, then adversarial verifiers who try to disprove every finding
- Covers correctness, security, performance, scalability, tests, architecture, maintainability, duplication, language idioms and documentation
- Configurable subagent budget, `--agents N`, defaulting to 10 and capped at 10, with review dimensions merged automatically at smaller budgets
- Every finding carries file:line evidence, a verdict and, where rejected, the counter-evidence that ruled it out
- Reviews the working tree and the commit range together, and never modifies the code under review

**Process:**
1. Resolves the scope, commit range and subagent budget from your arguments, asking only when you give none
2. Writes a review brief and launches Round 1 reviewers in parallel, one per review dimension
3. Launches Round 2 verifiers that must reproduce or reject each Round 1 finding from source, and hunt for what Round 1 missed
4. Deduplicates findings, drops the rejected ones and ranks what survives by severity
5. Writes a ranked, self-contained report to `code-review-report.md` and prints the verdict

#### Skills

##### review-rubric

The review rubric used by the subagents - dimensions, finding schema, severity scale and verification protocol. Invoke it on its own to apply the same standard during a manual review.

```
/code-review:review-rubric
```

## License

MIT
