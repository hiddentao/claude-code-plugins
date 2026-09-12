---
description: Adversarial multi-agent review of a codebase and its recent commits. Usage - /code-review:adversarial [path] [last N commits] [with N agents]
---

# Adversarial Code Review

Review a codebase and its recent commit history using two rounds of subagents - independent reviewers, then adversarial verifiers whose job is to disprove what the reviewers found.

Follow the `code-review:review-rubric` skill for the review dimensions, the finding record schema, the severity scale and the verification protocol. Do not restate any of that here.

## Process

1. **Parse arguments**: Extract five things from `$ARGUMENTS`. Accept flags and plain English equally - `last 5 commits with 4 agents` and `--commits 5 --agents 4` mean the same thing. Extract in this order, removing each match before looking for the next, so that `with 4 agents` is never mistaken for a commit count.
  * Agent budget - `--agents N`, or `with N agents` / `use N subagents` / `max N agents`. Default 10.
  * History as a count - `--commits N` or `last N commits` / `past N commits`. Default 5.
  * History as a range - `--range A..B`, or `since <tag>` meaning `<tag>..HEAD`, or `against <branch>` meaning the merge base of `<branch>` and `HEAD`.
  * Scope - any leftover token that looks like a path. Several are allowed. Default the repository root.
  * Working tree - uncommitted changes are included by default. `--no-worktree` or `committed only` excludes them.
  * Working directory - `--out <dir>`. Default a fresh temporary directory from `mktemp -d`.

2. **Fill the gaps**: If `$ARGUMENTS` is empty, ask the user three questions in one message - what to review, how much history, and the maximum number of subagents - then proceed. If `$ARGUMENTS` is non-empty, do not ask anything; fill the missing axes with the defaults above.

3. **Set the agent budget**: The budget is the maximum number of subagents for the whole review, across both rounds. Clamp it to 10 and say so in one line if the user asked for more. Treat a non-numeric or below-1 value as 10. Split it into R reviewers and V verifiers, where R plus V never exceeds the budget.
  * 10 agents - 7 reviewers, 3 verifiers. Lanes A correctness, B security, C performance and scalability, D tests, E architecture and design patterns, F maintainability duplication and idioms, G documentation.
  * 9 agents - 6 and 3. 8 agents - 6 and 2. 7 agents - 5 and 2. 6 agents - 4 and 2. 5 agents - 3 and 2. 4 agents - 3 and 1. 3 agents - 2 and 1. 2 agents - 1 and 1.
  * As the budget shrinks, merge lanes in this order - E with F, then C into that group, then D into A, then B into A, then all code lanes together. Documentation stays a lane of its own down to a budget of 2, because it is the dimension a general reviewer most reliably skips.
  * If the user asks for 1 agent, warn that adversarial review needs at least 2, run a single reviewer across all lanes, and do the verification pass yourself - stating in the report that verification was not independent.

4. **Resolve the git range**: Run these read-only commands and keep the output small. Never read the full diff into your own context - record the file list in the brief and let each agent run its own git commands.
  * Confirm the repository and find its root with `git rev-parse --is-inside-work-tree` and `git rev-parse --show-toplevel`.
  * If `git rev-parse --verify -q HEAD` is empty there are no commits - review the working tree only.
  * Warn if `git rev-parse --is-shallow-repository` is true, since `HEAD~N` may not resolve.
  * To turn a count N into a base commit, compare N against `git rev-list --count HEAD`. If N is greater than or equal to the total, use the empty tree from `git hash-object -t tree /dev/null` - compute it rather than hardcoding a hash, so that SHA-256 repositories work. Otherwise use `git rev-parse "HEAD~$N"`.
  * For a branch comparison use `git merge-base <branch> HEAD` as the base. For an explicit `A..B` use A as the base; for `A...B` use `git merge-base A B`.
  * List the commits with `git log --first-parent --oneline --decorate -n <N>`. `HEAD~N` already walks first parents, so this matches the range. Check for merges with `git log --first-parent --merges --oneline -n <N>` and, if any are present, note in the brief that `git show` on a merge needs `--first-parent` or `-m`.
  * Size the change with `git diff --shortstat <base>..HEAD -- <scope>` and `git diff --stat <base>..HEAD -- <scope>`.
  * Get the file list with `git diff --name-status <base>..HEAD -- <scope>` plus pathspec exclusions for `*.lock`, `**/dist/**`, `**/build/**`, `**/node_modules/**`, `**/vendor/**` and `*.min.*`.
  * Unless the working tree was excluded, also capture `git status --porcelain`, `git diff HEAD --stat` and `git ls-files --others --exclude-standard`, and label those changes WORKTREE.
  * If the range exceeds roughly 20000 changed lines or the scope holds more than roughly 2000 files, narrow the review to changed files only and record the narrowing under coverage.

5. **Prepare the working directory**: Create the working directory with `round1/` and `round2/` subdirectories. Invoke the `code-review:review-rubric` skill and write its content to `rubric.md` inside the working directory, so that every agent can reach the rubric regardless of how skills resolve in its context. Write `context.md` containing the resolved scope, the base and head revisions, the commit list, the name-status file list, the working tree status, the agent budget and lane assignment, and any coverage caveats.

6. **Echo the plan**: Print one line confirming scope, range, working tree inclusion, budget split and working directory path. Do not wait for approval.

7. **Run Round 1**: Launch every reviewer with the Agent tool using `subagent_type` `general-purpose`. Issue all of the Agent tool calls in a single message so that they run concurrently - do not send them one at a time and do not wait for one to return before creating the next. Each prompt must contain all of the following.
  * An instruction to invoke the `code-review:review-rubric` skill and follow it exactly, falling back to reading `<workdir>/rubric.md` if the skill is unavailable.
  * The absolute path to `context.md`, and an instruction to read it before anything else.
  * The agent's lane letter and dimension, and a statement that this lane is its responsibility. Findings outside the lane should still be recorded with their real dimension, since duplicates are resolved later.
  * The scope rule - review the current state of the whole codebase in scope, using the commit range as the entry point and the focus. Run your own git commands rather than expecting a diff in the prompt.
  * The exact output path, `<workdir>/round1/<lane>-<dimension>.md`, and finding ids of the form `<LANE>-001`.
  * A requirement that every finding cite a real `file:line` the agent has actually read, and that speculation be omitted rather than filed at low confidence.
  * The read-only rule - you are reviewing, not fixing. Do not create, edit, move or delete any file in the repository under review. The only file you may write is your own findings file.
  * An instruction to return only a one-line severity tally and its output path, not the findings themselves.

8. **Run Round 2**: When every Round 1 agent has returned, launch the verifiers the same way - `subagent_type` `general-purpose`, all Agent tool calls in a single message. Assign lanes round-robin, so that verifier j takes lanes j, j+V, j+2V and so on. Each verifier then gets a spread of unrelated dimensions and every lane is verified exactly once. With 7 lanes and 3 verifiers that is V1 taking A, D and G; V2 taking B and E; V3 taking C and F. Each prompt must contain all of the following.
  * The same rubric instruction and `context.md` path as Round 1.
  * The absolute paths of the Round 1 files it must verify, and an explicit statement that it did not write them and must not edit them.
  * The adversarial posture, stated plainly - assume each finding is wrong until you can reproduce it from the source. Read the cited lines, grep for guards and call sites, check reachability, and run the relevant test if it is cheap and safe. Reading the reviewer's prose and agreeing is a failed verification. A pass that confirms everything is a failed pass.
  * The requirement to give every finding a verdict of CONFIRMED, REJECTED or UNCERTAIN, where CONFIRMED needs a `file:line` the verifier found itself, REJECTED needs counter-evidence, and UNCERTAIN must name the one fact that would settle it. Severity may be adjusted up or down with a stated reason.
  * The requirement to also sweep its lanes for issues Round 1 missed, filed as new findings with ids of the form `<LANE>-N01`. Reporting none is acceptable only with an explicit note of what was swept.
  * The output path `<workdir>/round2/V<n>-verification.md`, the verification record schema, the same read-only rule, and the same instruction to return only a tally and a path.

9. **Check coverage**: Confirm that every Round 1 finding id appears in exactly one Round 2 file. Re-dispatch any unverified id to the least loaded verifier, or verify it yourself if the agent budget is exhausted.

10. **Synthesise**: Build the report yourself from the files on disk - do not delegate this.
  * Drop REJECTED findings from the body and collect them in a section of their own with their counter-evidence. Keeping them visible is what shows the adversarial pass did its job. Keep UNCERTAIN findings in a separate section.
  * Apply the verifiers' severity adjustments.
  * Merge duplicates - same file, overlapping line ranges, same claim - keeping the highest severity, the union of the evidence and both ids.
  * Rank by severity, then confidence, then blast radius, then lane order.
  * Put the top 25 findings in the body and the remainder in a compact index table.

11. **Write the report**: Write `code-review-report.md` to the repository root with these sections in order - Scope, Verdict, Summary, Confirmed findings, Uncertain findings, Rejected by Round 2, Coverage and gaps, Suggested next steps, Appendix. If that file already exists, ask whether to overwrite it or write a timestamped name instead. The report must be self-contained, since the working directory is temporary - carry the evidence into it rather than pointing at files that will be cleaned up. Record the working directory path in the appendix.

12. **Report back**: Print the verdict, the summary table, the top five findings and the path to the report. Do not apply any fixes and do not commit anything - this command reviews only.
