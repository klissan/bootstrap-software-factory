# Validation checklist

Read at SKILL.md §9, after generation. Report fixture validation separately
from live integration validation. An item that was not run is "not run".

## Static

- Paths, references, configuration syntax, and runtime discovery.
- Existing instructions remain intact.
- Repeated bootstrap does not duplicate or overwrite user configuration.
- Every covered scope has exactly one decision log; existing conventions
  keep their place, format, and numbering.
- This run's answers, defaults, and open decisions have entries; links
  from state, change requests, and pointers resolve; no secrets in entries.
- Every chosen connection and runtime has a `docs/setup/` file and a
  checklist line; no setup file contains a credential value.
- Every mechanical step in the design maps to a script; no workflow or
  skill text re-describes a scripted step.

## Scripts (fixture)

- Each script has fixture tests for block, pass, fail, limit, invalid input,
  and invalidate paths, as applicable (see `scripts.md`).
- Missing configuration or credentials fail clearly and non-zero.
- Failed checks prevent completion.
- Candidate changes invalidate prior evidence; the gate compares with the
  current head.
- Interrupted runs resume without duplicate external writes.
- Pagination and partial failures are surfaced.
- Doctor reports pass or fail per item and prints no values.

## Runtimes

- Primary runtime discovers the generated entrypoints (start a fresh
  session and check, do not infer from file placement).
- Reviewer runtime: the recorded command ran against a fixture, the chosen
  model is in its catalog, and a write attempt in review mode failed.
- A skill or rule is not discovered twice through compatibility paths.

## Context budget

Always-loaded files are paid for in every session and every subagent:
AGENTS.md, CLAUDE.md, always-applied rules, and the descriptions of all
generated skills, agents, and commands.

- Keep them to identity, hard rules, and pointers. Procedures live in
  on-demand files (skills, workflows, references) loaded by the step that
  needs them.
- State a budget in configuration. Default, to be adjusted with the user:
  each always-loaded file at most 60 lines or 4 KB; each skill description
  at most 300 characters; all always-loaded content together at most 12 KB.
- The `context-budget` script measures bytes and lines of every
  always-loaded file per runtime, prints a JSON table, and exits non-zero
  over budget. Run it in validation and after every edit to those files.
- On-demand files have a soft budget too: flag any single file over 20 KB
  and split it by condition of use.
- Do not duplicate one text across runtimes' instruction files; point each
  at the shared file.

## Decoupling from the bootstrap skill

The factory must run without this skill installed.

- The `decoupling-check` script scans always-loaded instruction files,
  workflows, roles, scripts, generated skills, and machine-readable
  configuration used at run time for the bootstrap skill's name or path,
  and exits non-zero on any operational reference.
- Allowed: provenance fields and history in docs, decision logs, the
  feedback file, and the README note on when the skill is needed again.
  Keep the allowlist explicit in the script's configuration.
- Confirm no generated file links into the skill's `references/`; copy
  what the factory needs at run time into the factory's own files.
- If the skill lives inside the factory repository, run the check with
  the skill directory excluded, then confirm the first-run prompt works in
  a fresh session that does not invoke the skill.

## Conventions

- The configured branch pattern, transition allowlist, and review
  publication mode are exercised by fixture tests: `branch-name` (valid,
  too long, illegal characters, empty slug, reuse of an existing branch
  or change request, collision); `ticket-transition` (already in state,
  allowed move, move outside the allowlist refused, uncertain result);
  `review-publish` in each mode and with no change request.
- What happened matches the configured mode: no comments were posted in
  `local` or `summary`, exactly one summary exists, a draft was opened
  early only in `comments`, and the report names where the record lives.
- No transition ran with transitions disabled, and none outside the
  allowlist.

## Inline review threads and epics (only if configured)

- `review-publish` in `inline` mode, against a stub or disposable change
  request: new finding creates one thread; rerun creates none (duplicate
  suppression by marker); stale anchor after a new commit is matched and
  answered in the same thread; verified fix resolves; regression reopens;
  host without line anchors, or a rejected anchor, falls back to a flat
  `file:line` comment; missing resolve authorization leaves threads open
  and says so; a partial thread listing blocks posting.
- `epic-plan`: valid plan, cycle, missing criteria, missing contract
  entry, over the child limit. `child-create`: create, rerun reuses,
  uncertain result reconciles, link failure reported. `epic-status`:
  all done, one blocked holding its dependents, partial completion.
- No child item was created before plan approval unless the waiver is
  logged.

## Guards

- Every guard has fixture tests: a violating diff, a compliant diff, and
  its known false-positive cases; all pass.
- The verification stage runs all accepted guards and a violation fails it.
- Guards protect guards: changing a guard script, its tests, the hook
  configuration, or the gate trips the protection guard.
- Hook configuration parses and is valid for the installed runtime
  version; a deliberate violation in a fixture session is fed back to the
  agent. Where that was not exercised, the hook layer is unverified.
- Each guard is labelled with the layers it runs in (`guards.md`): a
  pre-action block counts only where a fixture session showed the call
  blocked, including the same change attempted through the shell tool;
  server-side only where it is a required check the agent cannot change.
- The publish-gate hook, where registered, blocks a publish command with a
  failing gate and allows it with a passing one; hook configuration and
  guard paths are protected from edits.
- No hook was registered without a decision entry; no target repository
  was modified without authorization.
- False-positive review: run the guards over recent history of the
  default branch and report how many past commits would have been blocked.

## Optional tools

- Every installed optional tool (MCP server, plugin, skill, CLI) proposed
  or found in this run is in the AI inventory and has a decision entry
  (accepted or declined, decided-by user). Nothing was installed without
  one.
- No install command or committed configuration contains a secret;
  versions are pinned where the mechanism allows.
- Removing or disabling each optional tool does not break the factory:
  no script, gate, or mandatory stage requires it, and the documented
  fallback works. Test by running the affected step with the tool
  disabled, or mark the fallback unverified.

## Workflows

- Cross-repository contracts are checked together.
- Incident reports preserve uncertainty and data boundaries.
- Run a small real delivery pilot if authorized and feasible.
- Optionally replay a historical incident without revealing its known
  cause to the investigator.
- Use isolated temporary fixtures when live operations are not authorized.
