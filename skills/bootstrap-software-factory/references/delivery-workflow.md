# Development workflow: generation details

Read before generating the delivery workflow, its skills, and its scripts.
The non-negotiable rules are in SKILL.md §7; this file adds the detail.
Mechanical steps below are scripts (see `scripts.md`); the workflow text
calls them and does not re-describe them.

## Stages

intake → discovery → plan → implementation → verification
→ review → integration → delivery

Allow explicit blocked, failed, cancelled, and retry states.

## Intake

- Fetch tracker references only from allowlisted instances and projects;
  ask about anything else.
- For free text: analyse, clarify, search for duplicates, then create the
  tracker item as a separately authorized write; where trackers are
  per-repository, one cross-linked item per affected repository.
- Snapshot issue identity and relevant content as data.
- Log clarifications; post them externally only if authorized.

Treat later requirement changes as scope changes requiring replanning.

## Planning

- Define observable acceptance criteria.
- Identify affected repositories and dependency ordering.
- Agree cross-repository contracts before dependent implementation.
- Separate editable working tests from protected acceptance checks.
- Log plan decisions that choose between alternatives, and every contract
  change as one shared entry.

## Workers and checkouts

For each worker, provide:
task ID, scope, repository/worktree, base revision, applicable instructions,
contracts, acceptance criteria, allowed operations, and required output.

Use one isolated checkout per concurrent writer.
Allocate separate ports, test databases, and temporary resources.
A worktree does not isolate processes, credentials, or network access.

Refuse a base with unpushed commits unless the user accepts it.
Never work in the user's main checkout.
A fresh checkout lacks ignored files, submodules, local tools, and caches:
define per-repository prepare steps that copy only example or template
files, never real secrets.
Verify the untouched base first; a failure there is pre-existing.
A task may change repository layout or verification commands: update the
repository profile within the task, log it, and keep both baselines.
Define cleanup of branches and checkouts after merge or cancellation.

Require worker output:
changed revision, summary, tests and evidence, unresolved issues,
contract changes, decisions made, and next dependency.

Execute mandatory stages explicitly.
Do not let a natural-language "done" skip required verification or review.

## Independent review

Provide requirements, diff, applicable rules, and test evidence in a fresh
context. Do not supply the author's self-assessment as proof of correctness.
Use only the verified reviewer invocation from the reviewer's runtime
profile, with read-only access. The reviewer runtime and model are
configured independently of the primary runtime.
A verdict parsed from reviewer output is advisory; treat an unparseable or
failed review as blocked.
If the required reviewer is unavailable, mark the stage blocked; do not
silently substitute self-review.
Define "approved" per host: who may approve and how it is recorded. Where
the host forbids approving one's own change, use a recorded gate verdict
plus whatever record the publication mode allows, and do not call it a
host approval.
Review rounds are limited: check every factual claim in generated
documentation against the code before the first round.

## Review publication

The user chooses how much of the review is published
(`review.publication`, logged as a user decision). In every mode the
reviewer call is local and headless, the stage is mandatory, evidence is
bound to the commit, and the full transcript of every round stays in the
factory's local state. A review is never a black box: there is always a
durable record in a place the user chose, and the task report says where.

- `comments`: open the change request as a draft after the first passing
  verification; publish every round (verdict, findings, revision, reviewer
  model) as one comment each; mark it ready only after the gate passes on
  the final head.
- `summary`: rounds run locally. Create the change request only after the
  gate passes on the final head, with ONE concise summary in its
  description (or a single final comment, the user's choice): verdict,
  number of rounds, what was found and fixed, reviewed revision, reviewer
  model.
- `local`: rounds run locally. Create the change request already reviewed,
  only after the gate passes on the final head, never as an early draft.
  Publish nothing except, if the user wants it, a one-line verdict or a
  link in the description. The durable record is the review report file.

Whatever is published is sanitized (no local paths), refused on
secret-like content, and idempotent (a hidden marker per revision and
round, or per summary). A new commit after publication invalidates the
review; the summary or verdict is then replaced, not duplicated.

No change request (completion point is local changes, or the host has
none): write the review report to the configured location in the factory
state (`review.report_path`, one file per task and revision), reference
it from task state and the final report to the user, and where authorized
and wanted add a one-line reference in the tracker item. Modes
`comments` and `summary` fall back to this file and say so.

## Evidence and integration

The verification stage runs every accepted guard against the candidate
(base to head) and records the result as evidence; a guard failure fails
the stage. Hooks give earlier feedback but never replace this run.


Bind evidence to the exact candidate:
repository revisions, test-suite revision, command, environment,
timestamp, exit status, and artifact location.
Do not claim commit-level verification with untracked or dirty changes;
capture the complete tree identity or first make a candidate commit.

Integrate a specific revision tuple across all affected repositories.
Invalidate relevant evidence whenever candidate code, contracts, tests,
or configuration changes. Amends and rebases change the revision: gates
compare evidence with the current head immediately before publishing.

Distinguish local verification, CI success, review completion,
PR creation, merge, deployment, and production verification.
Do not wait for or report CI that the repository's triggers never run.
Cross-repository PRs do not imply an atomic release.
Document merge/deployment ordering and backward compatibility when needed.

## Publishing

Before publishing, reconcile existing branches and PRs by task identity.
Use a stable external-operation key and inspect uncertain prior outcomes.
Store delivery links and partial successes.
Link relevant decision entries from the change-request body.

## Naming

Branch, commit message, and change-request title follow the configured
patterns (`naming.*`), inferred from the repository's existing convention
or chosen in the interview.

- The `branch-name` script builds and validates every name: placeholders
  (ticket key or number, type, slug), per-repository override, maximum
  length, allowed characters. The slug is derived mechanically: ASCII,
  lowercase, dashes, length cap. Raw ticket text never reaches a shell
  command or a ref name unvalidated.
- An existing branch or change request for the same task is reused, not
  duplicated; a name collision with a different task is a blocked state.
- Never push to the default or a protected branch.
- Cross-repository work uses each repository's own ticket number.
- Where the user wants a hard rule, enforce the patterns with a guard
  (`guards.md`); otherwise the script is the only producer of names.

## Ticket transitions

Off unless the user turned them on. When on, the configuration lists,
per tracker project, the events (work started, change request opened,
review passed, merged or closed, blocked), the target state for each, and
who moves the ticket (the factory's account or the user).

- Each transition is an external write under the separately authorized
  and separately verified `issue.transition` capability, limited to the
  configured allowlist.
- The `ticket-transition` script reads the current state first, discovers
  the transitions available for that issue (workflows differ per project;
  never hard-code identifiers), moves only along an allowed one, is a
  no-op when the ticket is already there, and after an uncertain result
  re-reads the state instead of retrying. It never interpolates ticket
  text into a command.
- A transition the host performs by itself (a closing keyword on merge,
  tracker-side automation) is not repeated by the factory: record which
  side owns each event. See "Ticket transitions" in the provider file.
- A failed or unavailable transition never blocks delivery; report it.

## After a failure

When a mistake reached review or the user (a forbidden edit, a broken
invariant, a regression): state the invariant in one sentence, propose a
guard for it (`guards.md`), and on the user's consent write it, add the
failing diff as a fixture test plus a compliant one, run the tests, log
the decision, and link the failure evidence. Keep it scoped to the
repository where it happened; do not promote one incident's workaround
into a global rule without the user's consent. If no mechanical check is
possible, record that and add the case to the review checklist instead.

## State and resume

Persist state outside conversation history:
task identity, scope snapshot, stage, attempts, worker ownership,
base/candidate revisions, evidence, blockers, decision-entry IDs,
and external operations.

Use a single state writer initially. A background or chained job that writes
state is a writer: while one runs, defer your own state writes, and do not edit
a control script that a queued job will still start.
Before enabling concurrent coordinators, implement locking or atomic
compare-and-swap updates and worker lease recovery.
Store state atomically and retain an event history.

Enforce configured retry limits.
A blocked stage creates a `proposed` entry; its resolution updates the entry.
On resume, run the status-summary script, read the linked decision entries,
then inspect actual repository and external state before continuing.
Never rerun completed external writes solely because chat history is missing.
