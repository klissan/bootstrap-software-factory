# Recommended tooling to propose

Read after the interview, together with `onboarding.md`. Propose the items
below that fit the chosen runtimes and project; the user accepts or
declines each one separately. Keep the list short: a tool earns a place
only by a capability the factory lacks or by measurable token savings.

Facts are from the linked official sources, read 2026-11-27, unverified on
any machine. Re-check commands against the docs for the installed runtime
version; where a command is missing here, read the tool's own docs. Never
invent a package name, command, or URL.

## Rules

- Install nothing without the user's explicit consent for that item.
- Prefer official sources. A third-party MCP server or plugin is code
  that runs with the user's credentials and file access: say so plainly,
  and recommend reviewing the source or choosing the vendor's own.
- Pin a version where the mechanism allows; `@latest` is not a pin.
- Least privilege: read-only mode and read-only credentials by default;
  enable only the toolsets the factory uses.
- No secret in an install command or in a committed configuration file.
  Reference an environment variable; if the runtime's syntax for that is
  unconfirmed, start without a key or have the user edit the local file.
- State the cost: every MCP server adds its tool descriptions to each
  session's context; say how many tools and prefer a narrow toolset.
- State what leaves the machine. A hosted lookup or search service
  receives the query text: never put secrets, personal data, or
  proprietary code in queries.
- Record every accepted item in the AI inventory and as a `user` decision;
  record every declined item as declined, and do not offer it again on
  rerun unless something changed.
- An accepted tool becomes a capability with status verified or
  configured_unverified and a fallback. The factory never depends on an
  optional tool the user did not accept, and still works when one is
  removed (`validation.md`).
- Accepted items get `docs/setup/<tool>.md` (purpose, cost, what leaves
  the machine, install, configuration file touched, verify, remove) and a
  doctor check.

## 1. Context7: current library and CLI documentation

- Why: version-specific documentation on demand instead of stale model
  knowledge or broad web searches; fewer wrong-API retries.
- Source: https://github.com/upstash/context7 (MCP server
  `@upstash/context7-mcp`; hosted endpoint `https://mcp.context7.com/mcp`).
  The backend is a hosted, closed service.
- Cost: two tools (`resolve-library-id`, `query-docs`); answers can be
  long, so scope each query to one concept.
- Leaves the machine: library names and query text. The README gives no
  data-handling statement: tell the user, and keep queries generic.
- API key: optional, for higher rate limits; created at
  context7.com/dashboard. Start without one.
- Install, as documented:
  - Claude Code: `claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp`
    (the docs add `--header "Authorization: Bearer <key>"`; omit it when
    keyless; drop `--scope user` for project scope).
  - Cursor: an `mcpServers.context7` entry with `"url": "https://mcp.context7.com/mcp"`
    in `~/.cursor/mcp.json` or `.cursor/mcp.json`; a key goes in `headers`
    as `${env:NAME}`.
  - Codex: `codex mcp add context7 -- npx -y @upstash/context7-mcp`
    (the docs append `--api-key <key>`; do not put a key on the command
    line; pin the package version).
  - Without MCP: `npx ctx7 setup` installs a skill that uses the `ctx7`
    CLI; `npx ctx7 remove` undoes it. Zero always-loaded tool descriptions.
- Verify: resolve one library the project uses and fetch one page.
- Remove: delete the server entry through the runtime's MCP command or
  configuration file (exact remove command: read the runtime's docs).
- Fallback: official documentation through the runtime's web tools.

## 2. The code host's and tracker's own tool

- Why: issue and change-request operations without a hand-written adapter.
- What: the official CLI or MCP server named in the chosen `provider-*.md`
  ("Ready-made tool or own adapter"). Propose exactly one transport per
  capability; do not install an MCP server next to a CLI that already
  covers the same operations.
- Cost and trust: as stated in the provider file.

## 3. Playwright MCP: only for projects with a web UI

- Why: lets verification drive a real browser through page structure
  rather than screenshots. Do not propose it otherwise.
- Source: https://github.com/microsoft/playwright-mcp (Microsoft; needs
  Node.js 20 or newer).
- Cost: a large tool set in context; enable it only in sessions or roles
  that test the UI where the runtime allows scoping.
- Leaves the machine: whatever the browser visits. Point it at local or
  test environments only, never at production with real credentials.
- Install, as documented: Claude Code
  `claude mcp add playwright npx @playwright/mcp@<version>`; Codex
  `codex mcp add playwright npx "@playwright/mcp@<version>"`; Cursor: an
  MCP entry with command `npx @playwright/mcp@<version>`. The docs show
  `@latest`; pin a version instead.
- Verify: open a local page and read its title. Remove: delete the entry.
- Fallback: the project's existing end-to-end tests run by `verify`.

## 4. A dedicated secret scanner

- Why: the hygiene scan (SKILL.md §2) is only as good as its patterns; a
  maintained scanner covers far more formats than a home-made regex list
  and runs locally with no model tokens.
- What: whichever scanner the project or organization already uses; if
  none, propose a well-known open-source one and take install and usage
  from its official documentation (not confirmed in this pass).
- Leaves the machine: nothing, if run locally without a hosted service.
- The `hygiene-scan` script wraps it and still reports file and key name
  only. Fallback: the built-in pattern scan, labelled as limited.

## Not on the list by default

Anything else: memory servers, generic filesystem or shell servers
(runtimes already have these), database servers (incident workflow only,
with enforced read-only credentials), and large plugin bundles. Propose
one only for a stated project need, under the same rules.
