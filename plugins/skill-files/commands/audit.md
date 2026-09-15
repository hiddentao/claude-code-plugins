---
description: Audit an agent-facing skill file against the rules for writing great skill files. Usage - /skill-files:audit [path] [--test] [--fix]
---

# Audit A Skill File

Review a skill file - a `SKILL.md`, an `llms.txt`, an `AGENTS.md`, or any document whose readers are AI agents - against the 28 rules and report what is wrong, where, and what to change.

Follow the `skill-files:writing-skill-files` skill for the rules, the numbering and the checklist. Do not restate any of that here.

## Process

1. **Parse arguments**: Extract three things from `$ARGUMENTS`. Accept flags and plain English equally.
  * Target - any token that looks like a path. Several are allowed, and each is audited separately.
  * Fresh-agent trial - `--test`, or `test it` / `try it on an agent`. Off by default, because it costs a subagent and only pays off on a file that describes a runnable workflow.
  * Fix mode - `--fix`, or `and fix them`. Off by default. Without it the audit reports only.

2. **Resolve the target**: If no path was given, search the repository for `SKILL.md`, `llms.txt`, `llms-full.txt`, `AGENTS.md` and `CLAUDE.md`. One match is the target. Several matches, and you list them and ask which. No matches, and you say so and stop rather than auditing something that was never meant for agents.

3. **Read the whole document**: Read the target end to end, then every file it links or references by relative path, and audit those as part of the same document. A rule violation hidden in a reference file is still a violation - progressive disclosure under rule 2 moves content, it does not exempt it. Note any referenced path that does not exist; that is a rule 18 finding.

4. **Establish ground truth**: Before judging accuracy, find what the document describes. Locate the SDK, API surface, schema, contract or codebase it documents, and read enough of it to check claims against it. If the subject is not reachable from here - an external API with no local source - say so, and mark every accuracy finding UNVERIFIED rather than asserting it.

5. **Walk the checklist**: One pass per rule group, in the skill's order. Record a finding for each violation, with the rule number, the line or heading where it occurs, one sentence on what is wrong, and the concrete replacement text or structural change that fixes it. "Add rationale here" is not a fix; the sentence you would add is.

6. **Verify every code block** (rule 15): For each snippet, check that the functions, methods, fields, endpoints, imports and parameters it names exist in the ground truth from step 4. A snippet citing something that does not exist is the highest-severity finding this audit produces, because it fails confidently and silently. Where the language has a cheap syntax check available, run it.

7. **Check identifier consistency** (rule 18): Collect every identifier, slug format, path shape, parameter and header name the document uses. Grep the whole document set for each one and for its plausible stale variants. Report any identifier that appears in more than one form, and say which form the ground truth uses.

8. **Check terminology** (rule 19): Flag terms that appear without definition and that a stranger could not resolve by searching. Internal code names are the usual offenders.

9. **Run the fresh-agent trial** (rules 23 to 25, only with `--test`): Launch a `general-purpose` subagent with the Agent tool. Give it the document contents, one realistic task drawn from what the document claims to enable, and nothing else - no repository context and no hints from this conversation. Instruct it to report what it did, what it assumed, and where it was unsure. Then compare what it did against the ground truth and record where it went wrong. Weight confident-wrong behaviour heaviest: an agent that reported success on a path that could not have worked is a finding about the document, not about the agent, and it points at a missing rationale, an unnamed anti-pattern or a stale example.

10. **Rank the findings**: Order them by how badly each one misleads an agent, not by rule number.
  * Blocker - the document will produce confidently wrong output. Non-existent code, stale identifiers, a quickstart that stops before the flow finishes, a human-gated prerequisite buried at the bottom.
  * Major - the document will be argued with or bypassed. Missing rationale, an unnamed anti-pattern, a "don't" with no "do", a vague escalation rule.
  * Minor - the document costs the agent time. Ambiguous wording, duplicated rules, overused blockquotes, binary required/optional.

11. **Report**: Print a table of rule number, severity, location, what is wrong and the fix, worst first. Follow it with the checklist items that passed, so that silence is not mistaken for not having looked, and a one-line verdict. If a fresh-agent trial ran, include what the agent actually did before the findings, because that evidence is the point of the trial.

12. **Apply the fixes**: With `--fix`, apply every Blocker and Major finding whose fix you are confident in, list the ones you left and say why, and print a diff summary. Without `--fix`, list the edits you would make and stop. Never commit.
