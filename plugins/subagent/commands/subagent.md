---
description: Find and launch a subagent matching your needs. Usage - /subagent:subagent <description of what you need>
---

# Subagent Picker

Take the user's query from `$ARGUMENTS` and use it to find the most relevant subagent(s) to run.

## Process

1. **Discover**: Look at the Agent tool's definition in your system prompt to extract all available `subagent_type` values along with their descriptions. This is your source of truth for what subagents exist.

2. **Match**: Compare the user's query (`$ARGUMENTS`) against the subagent names and descriptions. Select all subagents that are relevant to the query. If the query is empty or very vague, list all available subagents.

3. **Present**: Show the matching subagents as a numbered list with their type and a one-line description. Group them by category prefix (e.g. `feature-dev:*`, `pr-review-toolkit:*`, `voltagent-qa-sec:*`, and everything else as general).

4. **Ask**: Ask the user two things:
   - Which subagent number they want to run
   - What prompt/task to give the subagent (suggest a reasonable default based on their original query)

5. **Launch**: Use the Agent tool with the selected `subagent_type` and the user's prompt. Set a short `description` summarizing the task.
