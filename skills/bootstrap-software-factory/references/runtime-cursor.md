# Runtime profile: Cursor

Status: every fact below is `documented` (official docs read 2026-11-27).
None was verified on a machine. Verify each one you use with a real
fixture call before generating scripts (see `runtime-profile.md`).

Sources: cursor.com/docs/cli/headless, /cli/using,
/cli/reference/parameters, /cli/reference/permissions,
/cli/reference/authentication, /context/rules, /context/skills,
/context/mcp.

## Instructions

- Project rules: `.cursor/rules/*.mdc` (subfolders allowed; plain `.md`
  there is ignored). Frontmatter: `description`, `globs`, `alwaysApply`.
  Types: always, agent-decided by description, by file glob, manual.
- AGENTS.md is read at the project root and in subdirectories; the more
  specific file wins. The CLI also reads a root CLAUDE.md as a rule.
- Precedence: team rules, then project rules, then user rules. Team rules
  can be enforced by an admin and are invisible in the repository: ask.
- Docs advise keeping a rule under 500 lines; apply the factory's stricter
  context budget to `alwaysApply: true` rules.

## Skills, commands, hooks, permissions

- Skills: `.cursor/skills/` and `.agents/skills/` (project), the same under
  the home directory, and compatibility paths `.claude/skills/` and
  `.codex/skills/`. Each is a folder with `SKILL.md`; frontmatter `name`
  (must match the folder), `description`, optional `paths`,
  `disable-model-invocation`. Consequence: skills generated for another
  runtime may already be visible to Cursor; do not generate a second copy.
- Invocation: automatically by description, or `/` in the agent chat.
- CLI permissions: `~/.cursor/cli-config.json` (global) and
  `<project>/.cursor/cli.json`; `permissions.allow` and `permissions.deny`
  with tokens `Shell(...)`, `Read(...)`, `Write(...)`, `WebFetch(...)`,
  `Mcp(server:tool)`. Deny wins.
- Hooks and subagents exist; their locations and schemas were not
  confirmed in this pass: unknown, read the docs for the installed version.

## MCP

- `.cursor/mcp.json` (project) and `~/.cursor/mcp.json` (global), top-level
  `mcpServers`. Stdio: `command`, `args`, `env`, `envFile`. Remote: `url`,
  `headers`, `auth`.
- Reference secrets as `${env:NAME}`; never write a value into the file.
- The CLI uses the same configuration; `agent mcp list`,
  `agent mcp list-tools <id>`. `--approve-mcps` auto-approves servers: do
  not use it for a reviewer.

## Headless CLI

- Executable: `agent`. Version: `agent --version`; account: `agent status`.
- Non-interactive: `agent -p "<prompt>"`; `--output-format text|json|stream-json`;
  `--model <model>`; `--workspace <path>`; `--trust` (headless only).
- Models: `agent models` or `--list-models`. No separate effort flag is
  documented; effort may be part of the model name. Unknown: discover
  from the catalog and record what `reviewer.effort` maps to, or mark it
  unsupported.
- Modes: `--mode ask` (answers without editing files) and `--mode plan`.
- Sandbox: `--sandbox enabled|disabled`.

## Reviewer safety

Print mode can write: without `--force` it still edits workspace files and
runs allowlisted commands; `--force` / `--yolo` allow everything not denied.
For a reviewer:
- never pass `--force`, `--yolo`, or `--approve-mcps`;
- use `--mode ask`, and verify that it combines with `-p` (unknown);
- add `permissions.deny` for `Write(**)` and shell in the review
  workspace's `.cursor/cli.json`, and review an exported copy or a
  disposable checkout rather than the working checkout;
- prove it: the fixture call must fail to create a file.
If read-only cannot be demonstrated, label the reviewer's isolation
advisory and log it.
An admin can disable headless mode or "run everything"; `agent -p` then
exits with an error. Detect and report as blocked.

## Authentication

- Interactive: `agent login` (browser; `NO_OPEN_BROWSER=1` prints the URL).
- Automation: `CURSOR_API_KEY` in the environment (preferred over
  `--api-key`, which exposes the key in the process list). Keys are created
  in the Cursor dashboard under API Keys: https://cursor.com/dashboard/api
- Verify without printing: `agent status`.

## As primary

The coordinating session runs in the Cursor editor's agent or in the
interactive `agent` CLI. Entry points: a short `alwaysApply` rule or
AGENTS.md pointing at the shared workflows, plus skills.

## Limits and unknowns

- Windows support of the `agent` CLI outside WSL: unknown, check first.
- No documented way to pass the prompt by standard input or file:
  unknown; if absent, write the prompt to a file and instruct the agent
  to read it, never interpolate external text into the argument.
- Exit-code behavior on refusal or failure: unknown; test it.
- Cloud agents do not read `~/.cursor/mcp.json`.
