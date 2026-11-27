# Runtime profile: Codex

Status: items marked (run) were observed in a real bootstrap run on another
machine; the rest are `documented` or `unknown`. Re-verify everything you
use on this machine (see `runtime-profile.md`).

Sources: official Codex documentation (configuration and non-interactive
mode pages), read 2026-11-27.

## Instructions and configuration

- AGENTS.md is the instruction file (repository root and nested; a
  user-level one under the Codex home directory). Documented from general
  knowledge, not re-read in this pass: confirm discovery and precedence.
- Configuration: `~/.codex/config.toml` (user) and `.codex/config.toml`
  (project; loaded only for trusted projects). Keys: `model`,
  `model_reasoning_effort`, `sandbox_mode`, `approval_policy`. Named
  profiles are selected with `--profile`.
- MCP servers are configured in the same TOML configuration; the table
  name and schema: unknown, read the MCP page for the installed version.
- Skills and custom prompts: locations unknown; discover them. Do not
  assume Claude Code subagent or hook files work here unchanged.

## Headless CLI

- `codex exec` is the non-interactive mode (run).
- Flags observed working (run): `-m <model>`, `-s read-only`,
  `--ephemeral`, `-C <dir>`, `--color never`; the prompt can be supplied
  on standard input.
- Model catalog: `codex debug models` (run). Choose `reviewer.model` and
  `reviewer.effort` only from what the catalog lists.
- Reasoning effort: the configuration key is `model_reasoning_effort`;
  how to override it per call is unknown here: confirm with `--help`.
- `codex exec review --base <branch>` rejects a custom prompt (run), so the
  reviewer cannot be given requirements that way. Use plain `codex exec`
  in the read-only sandbox with the requirements, diff, and rules supplied
  in the prompt.
- Exit code and final-message format: test with the fixture; a verdict
  parsed from the last line is advisory.

## Reviewer safety

- `-s read-only` plus `--ephemeral`, working directory set with `-C` to
  the checkout under review.
- Prove it: the fixture call must fail to create a file.
- A project `.codex/config.toml` in a trusted checkout can change sandbox
  or approval settings: pass the read-only flag explicitly on every call.

## Authentication

- Interactive login through the CLI, or an API key in the environment for
  automation. Exact command and variable names: confirm in the official
  authentication page for the installed version before writing the setup
  file. Store the reference name only.
- Verify without printing: a trivial read-only `codex exec` fixture call.

## As primary

The coordinating session runs in the interactive Codex CLI in the factory
directory. Entry point: AGENTS.md pointing at shared workflows and scripts.

## Limits and unknowns

- Skill and subagent support differs from Claude Code and by version.
- Hook support on Windows: not stated; verify.

## Hooks

Documented (official Codex hooks page, read 2026-11-27); verify on the
installed version before registering a guard (see `guards.md`).
- Configuration: `.codex/hooks.json` (project; loaded only from trusted
  project layers), `$CODEX_HOME/hooks.json` (user), a system location, or
  inline `[hooks]` in `config.toml`. On by default.
- Non-managed hooks must be reviewed and trusted by the user before they
  run, and again after any change (`/hooks` in the CLI): a newly written
  guard is inactive until then.
- Shape: event → matcher group → handlers (`type: "command"`, `command`,
  `timeout`); a `commandWindows` override is documented.
- Events used by guards: `PreToolUse`, `PostToolUse`, `Stop`. File edits
  arrive as `apply_patch` (matchable as `apply_patch`, `Edit`, or `Write`);
  shell commands as `Bash`.
- Input: JSON on standard input with `cwd`, `hook_event_name`, `tool_name`,
  `tool_input`.
- Exit 2 is the blocking code with the reason on standard error. A blocked
  `PostToolUse` replaces the tool result with the feedback (the edit is not
  undone); a blocked `Stop` becomes a continuation prompt. On exit 0,
  `Stop` output must be JSON. JSON alternative: `decision: "block"`, `reason`.
- Blocks before the action: documented yes. `PreToolUse` exiting 2, or
  `hookSpecificOutput.permissionDecision: "deny"`, blocks with the reason
  fed back. Not verified on a machine.
- Organization level: a system-wide hooks file and "managed" hooks exempt
  from the trust review are documented; whether a user or project can
  disable them (`[features] hooks = false` turns hooks off) is
  unconfirmed: verify at bootstrap.
- Matching hooks for one event run concurrently: guards must not depend
  on each other's order. Default timeout 600 seconds: set a short one.
- Unconfirmed: behavior of other exit codes; support on Windows.
