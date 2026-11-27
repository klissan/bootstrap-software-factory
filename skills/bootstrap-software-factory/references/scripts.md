# Mechanical steps as scripts

Read at design time (SKILL.md §5) and before writing any script (§9).

## Rule

Classify every workflow step:

- Judgement: understanding requirements, planning, implementing, reviewing,
  resolving ambiguity, deciding between alternatives.
- Mechanical: everything whose correct output follows from its inputs.

If a step is repeated and does not need judgement, an LLM must not perform
it by hand. It becomes a tested script; the model supplies arguments and
reads the result. Record the classification in the design, one line per step.
Generated skills and workflows call the scripts by name. They do not
describe the commands for the model to re-derive, and they do not restate
what a script already enforces.

A step that mixes both is split: the script does the mechanical part and
returns structured data; the model decides; another script applies it.

## Script contract

- Written for the shell and version actually detected. Do not assume bash
  or PowerShell. Prefer an interpreter already required by the project.
- One cross-runtime layer: the same scripts serve every runtime (Claude
  Code, Codex, Cursor). Runtime-specific files only wrap or reference them.
- Structured input (named, validated parameters; files for multi-line or
  external content) and JSON output on stdout. Human text goes to stderr.
- Fail closed: any error, missing configuration, or failed child command
  yields a non-zero exit and no recorded success. Document exit codes;
  distinguish "blocked, needs the user" from "failed".
- Idempotent where it touches external systems: look up by task identity or
  a stable operation key first, reuse what exists, never blind-retry.
- Validate every external value (URLs against the allowlist, identifiers
  against a strict pattern) before use. Never interpolate ticket text,
  branch names, or log content into a command line.
- Never print a secret; never pass one as a command-line argument.
- No network or write side effects beyond the documented ones.
- Bounded output: return counts and paths, write bulk output to a file.
  Tool output is also context cost.

## Candidate checklist

Adapt per project; create only what the workflows use.

| Script | Does |
|---|---|
| doctor | Checks tools, versions, paths, credential references, reviewer CLI; see `onboarding.md` |
| issue-fetch | Validates the reference against the allowlist, fetches, writes the snapshot |
| issue-create | Duplicate search, create from a body file, reconcile uncertain results |
| new-worktree | Base preflight (unpushed commits, dirty base), branch, checkout, prepare steps |
| hygiene-scan | Secrets (file and key name only), size, build output, local settings, nested repos |
| verify | Runs configured commands on a clean tree, records evidence bound to the commit |
| baseline | Extracts passed, failed, skipped counts from verification output |
| state | Atomic stage transitions with event history; status summary for resume |
| reconcile | Finds existing branch and change request by task identity |
| change-request | Creates the draft from a body file, marks ready, records links |
| review | Runs the verified reviewer command read-only, stores raw output |
| review-post | Extracts the verdict, sanitizes paths, refuses on secret-like content, posts idempotently |
| gate | Compares evidence and review with the current head; pass or fail only |
| cleanup | Removes merged or cancelled worktrees and branches, never the main checkout |
| report | Assembles the task or readiness report from state and evidence |
| context-budget | Measures always-loaded instruction files against the budget; see `validation.md` |
| decoupling-check | Fails if run-time files reference the bootstrap skill; see `validation.md` |

## Tests

Every script has fixture tests against an isolated temporary remote plus
clone, never a real repository. Cover, where applicable:
block (precondition refused), pass, fail (child command fails), limit
(attempt or size limit reached), invalid input, and invalidate (a new commit,
amend, or config change makes earlier evidence stale).
External writes are tested against a disposable resource or a stub; say
which. Rerun the tests after every script change and record the result.

## Shell pitfalls

List the pitfalls of the detected shell and version in the factory's docs
and add a test for each one that applies. Known categories for every shell:
error and exit-code propagation, stderr handling, empty pipelines, reserved
names, quoting and argument arrays, text encoding and byte-order marks,
missing native parsers, path separators, line endings.

Windows PowerShell 5.1 (observed in a real run):
- No YAML parser: keep machine-readable configuration in JSON.
- Without `$ErrorActionPreference = 'Stop'`, non-terminating errors leave
  exit code 0, so failures look like success.
- With it, native tools that write warnings to stderr can raise terminating
  errors; handle native stderr deliberately.
- Check `$LASTEXITCODE` after every native command; `$?` is not enough.
- `$null | ForEach-Object` runs the block once.
- `$args` is reserved; do not use it as a variable or parameter name.
- `Set-Content -Encoding UTF8` writes a byte-order mark, which corrupts
  commit messages and body files; write BOM-free bytes explicitly.
- Concatenating a string array into a string joins lines with spaces.
- `&&`, `||`, and the ternary operator are not available.

POSIX shells: use `set -euo pipefail` where the shell supports it and know
its gaps (command substitution, `local x=$(...)`); quote every expansion;
do not parse JSON with text tools when a JSON tool is available; check the
actual shell, since `sh` may not be bash.

Add newly observed pitfalls to the factory docs and to
`docs/skill-feedback.md`.
