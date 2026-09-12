---
name: review-rubric
description: The rubric, finding schema, severity scale and verification protocol for adversarial code and documentation review. Use when reviewing code or docs for correctness, security, performance, scalability, maintainability, test coverage, architecture or documentation quality, and when recording or verifying review findings.
---

Use this rubric whenever you are reviewing code or documentation and recording what you find. Every finding must be falsifiable, cited to a real `file:line` you have read, and written so that somebody else can try to prove you wrong.

## Review dimensions

* Correctness
  * Logic errors, off-by-one, null and undefined handling, wrong boundary conditions
  * Error handling - swallowed errors, wrong error types, failures that leave state half-written
  * Concurrency - races, deadlocks, non-atomic read-modify-write, unawaited work
  * Resource handling - leaked handles, connections, listeners, timers
  * API contract violations, and regressions introduced by the commits under review
* Security
  * Injection of every kind, path traversal, SSRF, unsafe deserialization
  * Authentication and authorization gaps, missing ownership checks, privilege escalation
  * Secrets in source, logs or error messages
  * Missing or bypassable input validation at trust boundaries
  * Crypto misuse, weak randomness, and dependency and supply-chain risk
* Performance
  * Algorithmic complexity on paths that see real volume
  * N+1 queries, repeated work that could be hoisted, hot-path allocations
  * Synchronous or blocking I/O where it costs throughput
  * Caching that is missing, unbounded, or incorrect
* Scalability
  * Unbounded growth - queries without limits, lists without pagination, buffers without caps
  * Shared mutable state or local state that prevents running more than one instance
  * Missing backpressure, retry storms, absent timeouts
* Tests
  * Coverage - changed lines and new branches with no test exercising them
  * Testability - hidden globals, unmockable dependencies, constructors doing I/O, missing seams
  * Test quality - assertions too weak to fail, over-mocking that tests the mock, time and order dependence
* Architecture and design patterns
  * Layering violations, inappropriate coupling, low cohesion, responsibility in the wrong place
  * Abstractions that do not fit the problem, and leaky interfaces
  * Pattern misuse and over-engineering - indirection that buys nothing
  * Elegance - whether a simpler structure would express the same behaviour
* Maintainability, duplication and idioms
  * Redundancy - copy-pasted and near-duplicate logic that must now be changed in several places
  * Dead code, unreachable branches, unused exports, stale configuration
  * Naming, function length, nesting depth, comments that no longer match the code
  * Language and framework best practices - idiomatic constructs, correct API usage, type system and linter features left unused, dependency and version conventions
* Documentation
  * Clarity - can a competent newcomer act on this without guessing
  * Readability - structure, ordering, headings, examples that actually run
  * Succinctness - padding, repetition, ceremony that could be cut without losing meaning
  * Alignment with the codebase - stale signatures, renamed concepts, examples that no longer work, undocumented public surface, documentation that contradicts the code it describes

## Severity scale

* CRITICAL - exploitable security hole, data loss or corruption, or a defect that breaks the primary path in production. Fix before merge.
* HIGH - wrong behaviour on a realistic input or state, a scaling cliff that will be hit, or no test around a critical path. Fix before merge.
* MEDIUM - a real defect with a workaround, meaningful duplication or maintainability debt, or documentation that will actively mislead a reader. Schedule it.
* LOW - narrow edge case, idiom deviation, minor redundancy, unnecessary verbosity. Fix opportunistically.
* NIT - subjective preference. No action required. Record at most five per dimension.

Two calibration rules:

* A security finding is never below MEDIUM unless the path is shown to be unreachable.
* A documentation finding caps at MEDIUM, unless the documentation misdescribes security or data-loss behaviour, in which case it is HIGH.

## Finding record schema

One record per finding. Ids are `<LANE>-001`, `<LANE>-002` and so on, where the lane letter identifies the dimension. Findings raised during verification use `<LANE>-N01`.

    ### A-003 | Severity: HIGH | Dimension: correctness
    - Location: src/queue/worker.ts:118-134
    - Claim: <one falsifiable sentence>
    - Evidence: <what in the source proves it - quoted lines, call sites, the commit that introduced it>
    - Impact: <what breaks, for whom, under which input or state>
    - Suggested fix: <the smallest change that resolves it>
    - Confidence: high | medium | low
    - Introduced by: <commit sha | pre-existing | worktree>

Begin each findings file with the lane, the agent id, and a count of findings by severity.

## Verification protocol

When verifying somebody else's findings, your default position is that each one is wrong.

* Reproduce from the source, never from the prose. Read the cited lines, grep for guards and call sites, check whether the branch is reachable, and run the relevant test if it is cheap and safe to do so. Agreeing with a well-written paragraph is not verification.
* Give every finding one of three verdicts.
  * CONFIRMED - you reproduced it, and you cite a `file:line` you found yourself.
  * REJECTED - you can say what makes it wrong. The guard already exists, the framework already handles it, the path is dead, or the behaviour is the documented contract.
  * UNCERTAIN - you cannot settle it, and you name the single fact that would.
* Adjust severity up or down where the original call was wrong, and state why.
* Also sweep the same dimension for what the first pass missed, and record those as new findings. Finding nothing is acceptable only with an explicit note of what you swept.
* Never edit the file you are verifying. Write your own, so the original findings stay auditable.
* A verification pass that confirms everything is a failed verification pass.

Verification record schema:

    ### A-003 | Verdict: REJECTED
    - Verified by: V1
    - Method: <what you actually did - files read, greps run, tests executed>
    - Counter-evidence: src/queue/worker.ts:97 already guards this with `if (!job) return`
    - Severity adjustment: HIGH -> none
    - Notes: <residual doubt, and what would change the verdict>

## Quality bar

* Cite a real `file:line` you have read. A finding without one is not a finding.
* One issue per record. Do not bundle.
* Omit speculation rather than filing it at low confidence. "This might be slow" is noise; "this runs a query per row over an unbounded list, see `x.ts:44`" is a finding.
* Style preferences belong at NIT or nowhere. Do not pad a report with them.
* Say what is fine as well as what is not, where it is load-bearing - silence about a well-covered area is indistinguishable from not having looked.
