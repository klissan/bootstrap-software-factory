# Guards: mechanical checks of project invariants

Read when designing the factory (SKILL.md §5), when discovery or a failure
yields a candidate invariant, and before writing any guard or hook.

## What a guard is

A small deterministic script that checks ONE invariant on the changed
files or on the diff against the base, and:

- exits non-zero with a short message that tells the agent exactly what
  is wrong and what to do next (the message is the fix instruction);
- is fast (seconds), read-only, idempotent, and needs no network;
- takes its inputs as validated arguments, never from ticket or diff text;
- has fixture tests: a violating diff, a compliant diff, and the known
  false-positive cases (renames, deletions, generated files, empty diff).

A guard checks; it never repairs. One invariant per guard keeps messages
precise and lets a guard be retired alone.

## Where guards run, and how strong each layer is

1. Harness-enforced BEFORE the action: a pre-action hook that blocks the
   tool call and feeds the reason back to the agent, or a permission deny
   rule. The harness runs it; the model cannot skip it. A hard block for
   the tool calls it covers, bounded by three limits:
   - Coverage. A matcher on file-edit tools does not see a file changed
     through a shell command (in-place edit, redirect, a script). Cover
     the shell tool too and inspect the command defensively, or restrict
     by permissions or sandbox. Pattern matching on shell commands is
     inherently incomplete: say so. Claude Code (docs: code.claude.com
     permissions, errors, sandboxing, read 2026-11-27): an `Edit(<path>)`
     deny rule blocks the file tools and shell writes such as `tee` and
     `>` redirection, but not a script that opens the file itself; only
     the sandbox enforces that at the OS level. Write the rule as
     `Edit(...)`: a deny rule on `Write(...)` never matches. `//path` is an
     absolute path, `/path` is relative to the working directory.
   - Who can change the configuration. Project, user, and local hook
     configuration and the guard scripts are files the agent could edit.
     Protect those paths (permission deny rules on them, in the syntax
     above or the runtime's own, plus a guard that blocks edits to guards
     and hook configuration), or place hooks in the runtime's
     organization-managed configuration where the machine allows it.
   - Scope. It constrains this runtime's tool calls, not the human, not
     another tool, not another runtime.
2. After-the-fact detection with feedback: post-action and stop-type hooks
   (which can make the agent continue), the factory verification stage and
   gate, git pre-commit and pre-push hooks. Catches a violation and gets
   it fixed; does not prevent the action. The verification stage is
   runtime-neutral, binds evidence to the commit, and is the default home
   of every guard.
3. Server-side: CI required checks, branch protection, code owners.
   Enforced regardless of any local agent; the only layer that also binds
   humans and other tools, provided the agent cannot change the pipeline
   or the protection.

Label every guard with the layers it actually runs in, per runtime: do
not assume parity. Whether a runtime can block before an action, and
whether it has a non-overridable organization level, is in the Hooks
section of each `runtime-*.md`; anything marked unconfirmed there is
layer 2 until a fixture session proves the block. Offer layer 3 as the
upgrade path and log the choice.

Guards protect guards: the first guard blocks any change to the guard
scripts, their tests, the hook configuration, and the gate, unless the
task is explicitly about them.

## Publish gate as a pre-action hook

"No publish without passing verification and review" need not be
advisory where the runtime supports blocking pre-action hooks:

- Register a pre-action hook on the shell tool. Its adapter reads the
  command from the hook input and, when the command publishes (push,
  create or mark ready a change request, merge), resolves the task from
  the working directory and calls the factory's gate script for the
  current head.
- Gate fails: block, with the gate's message (which check is missing or
  stale, and the script to run). Gate passes: allow. Gate cannot run: the
  failure policy above applies; by default the publish gate fails open
  with a loud warning, and the user may opt in to fail closed.
- Make the factory's publish scripts the only allowlisted way to publish
  and have them call the gate themselves, so the hook is the second
  barrier, not the only one. Deny direct publish commands by permission
  rule where the runtime has one.
- Command detection is pattern matching and can be evaded (aliases,
  wrappers, another tool); a publish by a human or from outside the agent
  is not covered at all. Only layer 3 closes that. State both in the
  gate's documentation and test the hook with allowed and blocked
  commands in a fixture session.

## One script, thin adapters

Write each guard once, runtime-neutral. Every layer calls the same script:

- Verification stage and git or CI: `guard <base> <head>`.
- Runtime hook: a thin adapter reads the hook's JSON from standard input,
  takes only the working directory and file path, validates them, calls
  the guard against the working tree, and translates the result into what
  that runtime feeds back to the agent (see the `Hooks` section of the
  chosen `runtime-*.md`). No logic lives in the hook configuration.
- Failure policy. A guard that found a violation always blocks. A guard
  that could not run (crash, timeout, missing interpreter or tool, invalid
  output) follows a policy the user sets per guard and the decision log
  records. Default: fail closed only for guards that protect secrets,
  because a leaked credential cannot be taken back; every other guard fails
  open with a loud message (to the agent and to a log line), never
  silently. Runtimes treat unexpected exit codes as non-blocking, so the
  adapter maps a failure to the blocking form, with the message "guard
  could not run", only for fail-closed guards. A failure is a defect to
  fix, not to ignore: the doctor script runs each hook on a sample input,
  and the fixture tests cover the failure paths (missing tool, bad input,
  timeout).
- Stop-type hooks can loop: use the runtime's loop limit or a counter, and
  stop blocking after the limit with a visible failure instead.
- Keep hook guards cheap and narrow (matcher on edit tools, path filter
  first). Run the full set in the verification stage.

## Authorization and safety

- A hook is executable configuration that runs with the user's privileges.
  Read existing hooks before registering anything; never overwrite them.
- Register or change a hook only with the user's explicit consent for that
  guard, and log it (`decided-by: user`).
- Writing hook configuration or a hooks directory into a target repository
  modifies that repository: only when authorized. Otherwise keep guards in
  the factory, run them in the verification stage and against each
  worktree, and offer the in-repository hook as a proposed patch.
- Never interpolate ticket text, diff content, or file names into a
  command line. Pass paths as arguments, quote them, reject revisions and
  paths that start with `-`, and resolve paths inside the checkout.
- Never print secrets; a guard that finds one reports file and key name.
- Some runtimes require hooks to be reviewed and trusted by the user
  before they run, and again after a change: tell the user when to do it.

## Shell

Write guards in the detected system shell: bash or zsh on Linux and macOS,
PowerShell on Windows. Detect it and its version; never assume. Apply the
pitfalls in `scripts.md`. A guard that needs real parsing (a syntax tree,
a schema, structured files) may use an interpreter only if the project
already requires it and it is present everywhere the guard runs; record
that dependency. Otherwise keep the check coarse and say what it misses.

## Contract

- Input: base revision and optional head revision (omitted: compare the
  working tree, including untracked files), or an explicit file list.
- Output: on pass, one JSON line on standard output; on violation, the
  message on standard error.
- Exit codes: 0 pass; 2 violation; 3 or more: the guard could not run.
  The verification stage treats every non-zero code as failure.

## Minimal example

The invariant "paths matching a pattern must not change". An illustration
of the contract, untested here: it must pass its fixture tests before use.

```bash
#!/usr/bin/env bash
# guard protected-paths: <base> [<head>]
set -euo pipefail
base="${1:?base revision required}"; head="${2:-}"
pattern='^(guards/|generated/)'
fail() { code="$1"; shift; printf '%s\n' "$@" >&2; exit "$code"; }
for rev in "$base" "$head"; do
  case "$rev" in -*|*[[:space:]]*) fail 3 "guard protected-paths: bad revision" ;; esac
done
git rev-parse --verify --quiet "$base^{commit}" >/dev/null \
  || fail 3 "guard protected-paths: unknown base revision"
if [ -n "$head" ]; then
  changed="$(git diff --name-only "$base" "$head" --)"
else
  changed="$(git diff --name-only "$base" --)"
  changed="$changed
$(git ls-files --others --exclude-standard)"
fi
hits="$(printf '%s\n' "$changed" | grep -E -- "$pattern" || true)"
[ -z "$hits" ] || fail 2 "guard protected-paths: FAIL" "$hits" \
  "These paths are protected. Undo your own changes to them; if the task needs them changed, stop and ask the user."
echo '{"guard":"protected-paths","status":"pass"}'
```

```powershell
# guard protected-paths: -Base <rev> [-Head <rev>]
param([Parameter(Mandatory = $true)][string]$Base, [string]$Head)
$ErrorActionPreference = 'Stop'
$pattern = '^(guards/|generated/)'
function Stop-Guard([int]$Code, [string[]]$Lines) {
    foreach ($l in $Lines) { [Console]::Error.WriteLine($l) }
    exit $Code
}
foreach ($rev in @($Base, $Head)) {
    if ($rev -match '^-|\s') { Stop-Guard 3 @('guard protected-paths: bad revision') }
}
& git rev-parse --verify --quiet "$Base^{commit}" | Out-Null
if ($LASTEXITCODE -ne 0) { Stop-Guard 3 @('guard protected-paths: unknown base revision') }
if ($Head) {
    $changed = @(& git diff --name-only $Base $Head --)
    if ($LASTEXITCODE -ne 0) { Stop-Guard 3 @('guard protected-paths: git failed') }
} else {
    $changed = @(& git diff --name-only $Base --)
    if ($LASTEXITCODE -ne 0) { Stop-Guard 3 @('guard protected-paths: git failed') }
    $changed += @(& git ls-files --others --exclude-standard)
    if ($LASTEXITCODE -ne 0) { Stop-Guard 3 @('guard protected-paths: git failed') }
}
$hits = @($changed | Where-Object { $_ -match $pattern })
if ($hits.Count -gt 0) {
    Stop-Guard 2 (@('guard protected-paths: FAIL') + $hits + @('These paths are protected. Undo your own changes to them; if the task needs them changed, stop and ask the user.'))
}
'{"guard":"protected-paths","status":"pass"}'
exit 0
```

## Where guards come from

- Discovery proposes candidates with evidence (`discovery.md`, "Candidate
  invariants"); the user accepts or declines each.
- A failure produces one (`delivery-workflow.md`, "After a failure").
- Start with the few highest-value, lowest-false-positive guards. Every
  blocking guard costs time on each change; too many slow work and teach
  the agent to route around them. Prefer warn-only for a new, uncertain
  guard and promote it after it has proved accurate.

## Override and retirement

- Override: only an explicit human approval for a named guard, task, and
  revision, recorded as a decision entry that the gate reads. "The agent
  judged it fine" is never an override. An override does not edit the guard.
- A guard that produces false positives is fixed or retired, not ignored:
  add the case as a fixture, or supersede the decision entry and remove
  the guard from every layer where it was registered.
- Record per guard: invariant, evidence or originating failure, layers,
  strength label, owner, and the decision entry.
