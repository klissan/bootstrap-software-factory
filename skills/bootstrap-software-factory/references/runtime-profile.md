# Runtime profiles

Read when selecting or verifying a runtime. Then read only the profile
files for the chosen primary and reviewer runtimes:
`runtime-claude-code.md`, `runtime-codex.md`, `runtime-cursor.md`.
For any other runtime, fill the same fields from its official documentation.

## Two independent roles

- Primary: where the coordinating session runs and discovers the factory's
  instructions, skills, and scripts.
- Reviewer: a headless, read-only, fresh-context call. Configure it
  separately: `reviewer.runtime`, `reviewer.model`, `reviewer.effort`, plus
  the verified command template. It may be a different runtime, or the
  same runtime with a different model. A reviewer on the same model as the
  author is weaker independence; record that as a decision.

More than one primary runtime may be supported. Keep procedures and
scripts runtime-neutral; per runtime, generate only thin entrypoints that
point at the shared files.

## Fields to fill per runtime

- Installed version and executable path (local configuration).
- Instruction discovery: files and directories read, nesting, precedence,
  and whether AGENTS.md is read natively.
- Locations and schemas: skills, agents or subagents, commands, hooks,
  permission settings.
- MCP configuration: file location, schema, how secrets are referenced.
- Headless invocation: command, prompt input (argument, stdin, file),
  output formats, working-directory flag, exit-code behavior.
- Model catalog command; how model and reasoning effort are selected.
- Read-only or sandbox mode and what it actually prevents.
- What a headless run auto-loads from the workspace (hooks, MCP servers,
  rules) and how to suppress it for a reviewer.
- Authentication: method and reference name only.
- Limits: what this runtime cannot do that the others can.

## Status of every fact

- `verified`: observed on this machine; record version, date, command.
- `documented`: from official documentation; record the URL and date.
  Usable for planning, not for a readiness claim.
- `unknown`: discover at bootstrap.

The profile files ship with `documented` facts and a few facts verified in
earlier runs on other machines. Treat all of them as `documented` until a
real fixture call on this machine confirms them: `--help` or `--version`,
the model catalog, and one headless read-only call that must return a
parseable answer and must fail to write a file. Never copy a flag into a
generated script before that call has passed. If the runtime is not
installed, generate its entrypoints as `configured_unverified` and say so.

## Cross-runtime layout

Prefer locations several runtimes read (AGENTS.md; a shared skills
directory where the installed versions support one) over per-runtime
copies. Check that compatibility paths do not make one runtime load the
same skill or rule twice.
