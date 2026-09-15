---
name: writing-skill-files
description: How to write a skill file that an AI agent can actually follow - structure, decision rules, named anti-patterns, code contracts, freshness and testing. Use when creating or editing a SKILL.md, llms.txt, AGENTS.md or agent-facing product doc, when reviewing one for quality, or when diagnosing why agents keep using a product, API or codebase the wrong way.
---

A skill file is the document that tells an AI agent how to use your product, API or codebase. Documenting the interface is the easy part and the part that matters least. What separates a skill file that works from one that reads well is everything wrapped around the interface: decision rules, recommended defaults, named anti-patterns, escalation paths, rationale, and worked examples.

The test to hold every edit against: **write for a new hire who cannot ask you a single question**. Not schema documentation for a human reader, but an executable spec for somebody who will act on it immediately and without checking back.

The 28 rules below are numbered so that a review can cite them. Rules 15 to 17 apply only if your skill file instructs agents to write code.

## Structure and ordering

**1 - Page order is priority order.** An agent that stops reading after the first subsection must still land on the right default. Labels like "recommended" are weak signals; physical position is a strong one. If you prefer flow A to flow B, A goes above B in the file, not below it with a nicer heading.

**2 - Happy path first, everything else behind progressive disclosure.** Open with the one call or one command that covers 90% of cases. Push manual flows, advanced configuration, hand-rolled fallbacks and edge-case handling into collapsible `<details>` blocks or separate reference files. Agents scanning the top converge on the working path; only an agent with a reason to reject it pays the cost of drilling down.

**3 - The quickstart covers the full end-to-end, not the easy 80%.** If there is a surprise mid-flow, it belongs in the quickstart. A three-step flow that omits the authentication challenge at step three is worse than no quickstart, because the agent commits to the path before discovering the step you left out. Walk your own flow and add a step wherever you would have had to explain something.

**4 - Front-load prerequisites that need a human.** Terms acceptance, legal consent, account funding, an owner granting access: these are step 1 of a numbered list with the URL inline, never a footnote or a closing note. An agent that meets a human-gated prerequisite at the end will already have done the thing on the human's behalf, and you find out later.

## Writing rules

**5 - Every rule carries its rationale.** A rule without a reason gets argued with, reinterpreted or quietly bypassed. Follow each rule with either its consequence or a worked example.

* "Fees are 10% of the price, minimum $0.005" is a fact. "So a $0.01 call pays a $0.005 fee, which is 50%" is a rule an agent can reason with.
* "There is no rollback to a previous version" is a fact. "So test before shipping, because unpublishing is final" tells the agent what to do differently.

**6 - Name anti-patterns together with their failure mode.** Agents reach for the most generic primitive they recognise. Naming the specific way that primitive fails kills the instinct before it runs. "Use the SDK" loses to a familiar generic approach; "the generic approach will not produce a valid request, because this endpoint needs a two-call sequence the generic client does not issue" wins.

**7 - Pair every "don't" with a "do".** A prohibition on its own leaves the agent with a gap and it will fill the gap with something. Give the drop-in replacement in the same sentence, so the agent substitutes rather than improvises: "never hand the user raw CLI or RPC commands; give them the dashboard URL and let them act there."

**8 - Name the path you are rejecting, not only the one you chose.** Agents arrive with priors from training data. If the popular library for your problem space is the wrong one here, say so by name and say why. Left unnamed, the prior wins: an agent told to "use our client" will still reach for the library it has seen ten thousand times. Name it, reject it, and give the reason that makes the rejection stick.

**9 - Disambiguate one word at a time, and collapse duplicates.** "Total number of published apps" costs one word over "total number of apps" and stops the agent building a mental model that silently counts drafts. The opposite failure is restating the same rule in two slightly different sentences: agents notice the near-duplicate and ask whether the two are different things. One statement per rule, as precise as you can make it.

## Visual hierarchy

**10 - Blockquotes are for hazards only.** Prose says what to do; a blockquote says what not to miss. Reserve it for the handful of facts that cause silent failure when skipped, and use nothing else for emphasis. Blockquote everything and you have blockquoted nothing.

**11 - Fields are required, recommended or optional.** Binary required/optional throws away the most useful signal you have. A field that is technically optional but that almost every real use needs is marked "optional (highly recommended)" in the field table itself, so an agent skimming the table gets the nuance without reading the prose around it.

## Decision rules

**12 - Every error code ends with a verb.** Not "402: payment required" but "402: complete the payment challenge and retry with the `Authorization: Payment` header; if funds are short, show the user the top-up URL." Every row of a retry or error table ends with an action the agent can execute, plus what to do when that action fails.

**13 - Say when to defer to the human, and how. Thresholds beat adjectives.** "Use judgement" and "if appropriate" are not instructions. "Do not recommend an integration after one successful call; wait for a repeated pattern. A single transient error is not grounds for dropping one" is.

**14 - Mirror server-side rejections into the doc.** Every constraint the backend enforces is worth stating before the agent hits it: reserved names, rate limits, immutable fields, format rules. State the rule and the rejection behaviour together, so no deploy cycle is spent discovering it.

## Code

These three rules apply when the skill file tells agents to write code.

**15 - Every code block is a contract.** Sample code must be valid and must run, because an agent will try it either way and fail confidently when it does not. A function that does not exist, a renamed parameter, a stale import: these produce hours of plausible wrong output. Treat a stale example as a bug, fix it in its own commit, and write the commit message so you can find it later.

**16 - Show persistence patterns as code, not prose.** "Persist this" survives one read. A copy-pasteable block survives the project. Show the actual read-if-exists, otherwise-create-and-write check, and follow it with the failure mode it prevents: "every call to `generateWallet()` returns a new address; calling it per run orphans any funded balance and forces re-registration."

**17 - Paste raw specs inline for last-mile tasks.** Exact format strings, full ABIs, verbatim header names, complete enum values. *Correct in spirit, wrong in exact string* is the most common silent failure in technical documentation, and it is the one an agent cannot debug from first principles. An agent that has to bypass your SDK should find a self-contained blob it can feed straight to a lower-level client.

## Identifiers and terminology

**18 - Identifier consistency is non-negotiable.** When a slug format, endpoint shape or parameter name changes, every occurrence changes with it: request paths, update and delete paths, tool names, manifest examples, and the prose that describes them. One stale reference sends an agent down an hour-long wrong path. A half-finished rename is worse than no rename, because now the file contradicts itself and the agent has to guess which half is current.

**19 - Scrub internal jargon.** If an agent searches your term and finds nothing, that is a bug in the doc. Internal names leak from architecture discussions into external docs and leave agents unable to connect the name to the thing. Use the name a stranger would find, and keep the internal name internal.

## Freshness

**20 - Tell the agent to re-fetch.** One line - "re-fetch this document at least daily to make sure you have the current contract, endpoints and instructions" - defends against drift in long-running agents, where a cached copy of the file gets reused for weeks.

**21 - Carry version metadata in the document.** A version tag in the frontmatter with a matching banner in the body gives both you and the agent a cheap diff check. If the banner in context does not match what a re-fetch returns, the agent knows to reload without re-reading the whole file.

## Design nudges

**22 - Steer toward better patterns without mandating them.** Not every line has to be a rule. Framing alone biases average output: "consider whether persistent storage adds value here - something that accumulates data across calls compounds in value, unlike a stateless proxy that forwards one request." Nothing is enforced, and you get better designs for free.

## Testing the skill file

**23 - Hand it to a fresh agent and watch what it does.** The only real test is a clean-context agent, a realistic task, your skill file and nothing else. Not what it reports the file says, which always sounds fine, but what it actually does when pointed at a sandbox. Watch for where it stalls, back-tracks or invents.

**24 - Vary the model.** Models differ in their priors, their instruction-following and their effective context. A file that passes on one can fall apart on another that has a stronger prior about a neighbouring library. If you can only afford one, test the model with the strongest competing priors, because it is the one most likely to trust its training data over your document.

**25 - Hunt specifically for confident-wrong behaviour.** An agent that throws an error or asks for clarification is the easy case. The dangerous case is the agent that proceeds down a wrong path and returns plausible output - reporting success on a request that was never actually made. That class of failure is almost always one of three things: a missing rationale, an unnamed anti-pattern, or a stale example. Check those three first.

## Maintenance

**26 - Every agent failure produces a change somewhere.** Either the doc changes or the code changes, never nothing. Recording "the agent got confused here" without a follow-up commit is exactly how a skill file rots. Agents inventing a function that does not exist is a doc commit pointing at the real one; agents omitting a required field is a doc commit adding that field to every example.

**27 - Prefer the doc fix to a new guardrail.** Patching a failure at the API layer - reject harder, add a validation message - is tempting. Do it when safety demands it. Otherwise the doc fix is cheaper, composes with every other agent that reads the file, and does not add surface area you maintain forever.

**28 - The skill-file changelog is the real product changelog.** When an agent-facing change ships, the skill-file commit is often not a side effect of the release, it is the release. A bug fix that only changes an example is still a bug fix. Write those commit messages so that future you can grep them when an agent does something surprising.

## The trajectory

Every good skill file follows the same arc: it starts as an API reference and ends as an operations manual. The interface stays roughly the same size; what grows around it is the decision rules, defaults, anti-patterns, persistence patterns, escalation paths, rationale and worked examples.

## Checklist

Walk this as a pass over any skill file. Each line maps to the rule of the same number.

* 1 - Does the first subsection of each section hold the preferred option?
* 2 - Is the happy path at the top, with fallbacks folded away?
* 3 - Does the quickstart reach a finished result, surprises included?
* 4 - Is every human-gated prerequisite step 1?
* 5 - Does every rule state why, or show what happens?
* 6 - Is each anti-pattern named with the way it fails?
* 7 - Does every "don't" have a "do" beside it?
* 8 - Are the tempting wrong paths named and rejected explicitly?
* 9 - Is each term precise, and each rule stated exactly once?
* 10 - Are blockquotes reserved for genuine hazards?
* 11 - Are optional-but-expected fields marked as recommended?
* 12 - Does every error row end in an executable verb, with a fallback?
* 13 - Are the escalate-to-human points stated as thresholds?
* 14 - Is every server-side rejection documented before it is hit?
* 15 - Does every code block run against the current API?
* 16 - Is persistence shown as code, with its failure mode?
* 17 - Are exact strings, headers, ABIs and enums given verbatim?
* 18 - Does every identifier match everywhere it appears?
* 19 - Can a stranger search every term and find it?
* 20 - Does the file tell the agent to re-fetch it?
* 21 - Is there a version tag, mirrored in the body?
* 22 - Are better patterns suggested where mandating them would be wrong?
* 23 - Has a fresh agent been watched attempting a real task with this file alone?
* 24 - Has it been tried on more than one model?
* 25 - Has it been checked for confident-wrong output, not just loud failures?
* 26 - Did the last agent failure produce a commit?
* 27 - Was the last guardrail added because safety demanded it, not convenience?
* 28 - Do the doc commit messages describe the agent behaviour they change?

## Applying this to a Claude Code skill

The rules above are about agent-facing documentation in general. A Claude Code `SKILL.md` adds a few specifics, all of which follow from the way skills load - the frontmatter `description` is preloaded into context, and the body is only read once Claude decides to invoke the skill.

* The `description` is the whole trigger surface, and it is the one place rule 9 cannot be recovered from. `description: Explores a codebase.` does not fail in the body; it fails before the body is ever read, because the skill never fires. Name the artefacts, the tasks and the symptoms that should pull it in, and put the main use case first. `description` and `when_to_use` share a budget of roughly 1536 characters, so there is room to be specific.
* `name` defaults to the directory name. In a plugin the skill is invoked as `/plugin-name:skill-name`, so keep the two identical and in kebab-case.
* Rule 2 has a concrete size limit here: keep `SKILL.md` under 500 lines and move long reference material, schemas and scripts into sibling files. Supporting files are not loaded automatically - link them from the body by relative path, or Claude never reaches them.
* Rules 15 and 17 bind hardest in this format, because the agent runs what you write. Every command and snippet in the body executes against a real machine, and an exact-string error surfaces as a failed command rather than a confusing response.
* `allowed-tools` (hyphenated) pre-approves tools for the skill, which is rule 7 in mechanical form: pair the "don't reach for X" with the tool the agent is meant to use instead.
* Rules 20 and 21 are the two that do not carry over. They exist for a document an agent fetches over the network and caches; a skill installed from a plugin or a repository is read from disk on each invocation, so the freshness problem is the installed version, not a stale copy in context. Version the plugin rather than the file, and skip the re-fetch line - an instruction the agent cannot act on is noise under rule 5.

Field reference and loading behaviour: <https://code.claude.com/docs/en/skills>

## Source

The 28 rules are from Ramesh Nair's "The definitive guide to writing great skill files for AI agents":
<https://hiddentao.com/archives/2026/04/26/the-definitive-guide-to-writing-great-skill-files-for-ai-agents>
