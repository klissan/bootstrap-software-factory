# Generated artifacts: layout and configuration

Read at SKILL.md §6, before writing files.

Prefer this logical layout; adapt to existing conventions:

factory.<format>
factory.local.<format> (ignored) and factory.local.example.<format>
README.md
AGENTS.md, plus CLAUDE.md or Cursor rules only for runtimes in use
decisions/factory.md, decisions/<repo-id>.md, decisions/shared.md
docs/discovery.md, docs/ai-inventory.md, docs/integration-capabilities.md
docs/setup/README.md, docs/setup/<connection-or-runtime>.md
docs/validation.md, docs/skill-feedback.md
repos/<repo-id>.md
workflows/deliver.md, workflows/investigate.md
roles/<needed-role>.md
scripts/<mechanical steps and controls>, with their fixture tests
<runtime-specific skills and agent configuration>

Create only applicable files.
Place runtime-specific entrypoints where the installed runtime discovers them.
Do not assume that a Markdown link automatically activates a skill.

Always-loaded files (AGENTS.md, CLAUDE.md, always-applied rules, skill and
agent descriptions) hold identity, hard rules, and pointers only, within
the stated budget (`validation.md`). Procedures go in on-demand files.

Generate project skills for delivery, investigation, and resume when useful.
Each skill must define its trigger, required inputs, procedure, outputs,
failure behavior, and completion conditions. Skills and workflows call the
scripts; they do not describe scripted steps for the model to re-derive.
Keep shared content in one place rather than duplicating it across roles
or runtimes.

No generated instruction file, workflow, role, script, or skill may depend
on this skill's name or path to operate. Provenance mentions are fine.
The factory README states the first-run prompt per runtime, where state
lives, and when the bootstrap skill is needed again.

Include configuration for:
schema version, generating skill revision, factory identity, runtimes
(primary and `reviewer.runtime`, `model`, `effort`, verified command),
repositories, connections, commands, workflows, permissions, limits,
context budget, decision-log locations, evidence storage, and readiness.
Choose a format the generated scripts' runtime parses natively.

Use structured command arguments and explicit working directories where
possible. Never interpolate ticket text, branch names, or log contents into
shell commands without appropriate validation and quoting.
Pass multi-line external content through files, not command arguments.

Keep machine-local checkout paths and tool locations in the ignored local
file, with a committed example. Verify each path and tool exists when
generating it. Keep shared configuration portable.
Ignore worktrees, local state, and evidence that may hold sensitive data.

Retain provenance (skill revision, run date) and content hashes for
generated files. On rerun never overwrite user edits, reset, clean, or
stash; follow `upgrade.md`.

When changes to target repositories are not authorized, generate proposed
patches and keep central profiles usable without applying those patches.

