# Runtime profile: Claude Code

Status: `documented` unless marked (run), which was observed in a real
bootstrap run on another machine. Re-verify what you use on this machine
(see `runtime-profile.md`).

Sources: code.claude.com/docs/en/headless (read 2026-11-27) and the CLI,
settings, skills, sub-agents, hooks, MCP, and memory pages linked from it.

## Instructions

- CLAUDE.md: project root, nested directories, and user level under
  `~/.claude/`. Further rule files may live under `.claude/rules/`.
- AGENTS.md is not read natively: keep shared text in AGENTS.md or a
  shared file and have a short CLAUDE.md import or point to it. Confirm
  the import syntax for the installed version.

## Skills, agents, commands, hooks, permissions

- Skills: `.claude/skills/<name>/SKILL.md` (project, discovered in a real
  run) and `~/.claude/skills/` (user). Only the description is always
  loaded; the body loads on invocation.
- Subagents: `.claude/agents/`. Commands: `.claude/commands/`.
- Hooks and permission rules: `.claude/settings.json`; machine-local
  overrides in `.claude/settings.local.json`, which must not be committed.
- A Markdown link does not activate a skill; start a fresh session and
  check discovery.
- File-read helpers may report a file as modified outside the session:
  re-read, do not revert (run).

## MCP

- Project servers: `.mcp.json` at the repository root (`mcpServers`);
  `claude mcp` manages them. Reference secrets by environment variable.

## Headless CLI

- `claude -p "<prompt>"`; standard input is read (10 MB cap).
- `--output-format text|json|stream-json`; `--json-schema` for a
  structured result in `structured_output`.
- `--model <name>`; `--allowedTools "<rules>"`;
  `--permission-mode <mode>` (for example `dontAsk` denies anything that
  would prompt); `--append-system-prompt`; `--mcp-config`; `--settings`.
- `--bare` skips discovery of hooks, skills, commands, subagents, plugins,
  MCP servers, and CLAUDE.md; it needs `ANTHROPIC_API_KEY` or an
  `apiKeyHelper` and does not use the subscription login.
- Exit code 0 on success, non-zero on failure; some in-run failures are
  printed as the result on standard output: check both.
- Reasoning effort selection per call: unknown here; confirm in the CLI
  reference for the installed version.

## Reviewer safety

- Without `--bare`, a `-p` run executes the workspace's project hooks and
  connects its `.mcp.json` servers with no trust prompt. For a reviewer,
  use `--bare` and pass the rules it needs explicitly.
- Read-only: allow only read tools through `--allowedTools` with
  `--permission-mode dontAsk`, and supply the diff on standard input so no
  shell access is needed.
- Prove it: the fixture call must fail to create a file.

## Authentication

- Interactive: subscription login inside the CLI.
- Automation: `ANTHROPIC_API_KEY` in the environment, key created in the
  Claude Console (https://platform.claude.com), or `apiKeyHelper` in
  settings. Other cloud providers use their own credentials.
- Verify without printing: a trivial `claude -p` fixture call.

## As primary

The coordinating session runs in the factory directory. Entry points: a
short CLAUDE.md and project skills (for example deliver, resume).

## Limits

- Flags and minimum versions change often; the headless page states
  version requirements per flag. Check the installed version first.

## Hooks

Documented (code.claude.com/docs/en/hooks, read 2026-11-27); verify on the
installed version before registering a guard (see `guards.md`).
- Configuration: `hooks` in `.claude/settings.json` (project, shared),
  `.claude/settings.local.json` (project, ignored), `~/.claude/settings.json`
  (user); also managed policy, plugins, and skill or subagent frontmatter.
- Shape: event → matcher group (`matcher` on the tool name) → handlers
  (`type: "command"`, `command`). `${CLAUDE_PROJECT_DIR}` is the project root.
- Events used by guards: `PreToolUse`, `PostToolUse`, `Stop`.
- Input: JSON on standard input with `cwd`, `hook_event_name`, `tool_name`,
  `tool_input` (file tools carry `tool_input.file_path`).
- Exit 2 is the blocking code and feeds standard error to the model:
  `PreToolUse` blocks the call; `PostToolUse` reports after the tool
  already ran; `Stop` prevents stopping. Any other non-zero code is a
  non-blocking error. JSON alternative: `decision: "block"` with `reason`.
- Blocks before the action: yes. A `PreToolUse` hook exiting 2 (or JSON
  `hookSpecificOutput.permissionDecision: "deny"`) stops the call before
  it runs and is evaluated before permission rules, so an allow rule does
  not bypass it.
- Organization level: hooks in managed policy settings cannot be disabled
  from user, project, or local settings (`disableAllHooks` respects the
  hierarchy; `allowManagedHooksOnly` restricts hooks to managed ones).
  Project, user, and local hooks are ordinary editable files: protect them
  with permission deny rules (syntax: permissions page).
  Sources: /hooks, /hooks-guide, /permissions, /settings-reference.
- Default command timeout is 600 seconds: set a short one.
- Unconfirmed: the loop-protection field for `Stop` hooks.
