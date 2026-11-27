# Provider: Forgejo

Facts from forgejo.org docs (API usage, token scope), read 2026-11-27.
Confirm against the documentation for the DEPLOYED version.
Docs: https://forgejo.org/docs/latest/user/api/usage/
Scopes: https://forgejo.org/docs/latest/user/token-scope/

## Distinctions to record

Instance URL and version, owner/repository, the instance's own OpenAPI
schema, PRs, issues, and whichever CI or status integration is actually
attached. Do not assume parity with GitHub or Gitea, and do not assume a
GitHub CLI works against it.

## Setup instructions to give the user

- Create an access token in the user settings (Applications section in
  current versions; confirm the menu path in the docs for the deployed
  version before writing it down).
- Repository and organization access: choose "specific repositories" and
  select only the factory's. This mode permits only the repository and
  issue scopes, which is what the factory needs.
- Scopes, only for authorized capabilities: `read:repository` and
  `read:issue` for reads; `write:repository` to push branches and manage
  pull requests; `write:issue` to create or comment on issues and pull
  requests. No `admin`, `organization`, or `user` write scopes.
- Expiry: if the deployed version offers none, record a renewal date
  (default advice 90 days) and rotate manually.
- Use: header `Authorization: token <token>`.
- Git transport: SSH key or credential helper.

## Verify without printing

- `GET <base>/api/v1/version` for the version.
- `GET <base>/api/v1/repos/<owner>/<repo>` returns 200 with the expected
  permissions block.
- Read the instance's OpenAPI document before generating any adapter call.
- Send the header from a file or the environment; print status codes only.

## Notes

- No official first-party CLI is assumed: use an available MCP tool or a
  small adapter over the verified API.
- Check which CI system reports statuses and on which events.

## Ready-made tool or own adapter

Selection rule: official ready-made tool > well-maintained community tool
(only after the user's consent and a trust note: it runs with the user's
credentials) > thin adapter written by the factory. Whatever is chosen,
the glue around it (snapshots, reconciliation, idempotent posting,
evidence) stays in the factory's own scripts: it is repeated and needs no
model. Facts read 2026-11-27; anything not from the linked official page
is "verify at bootstrap".

- Official: none confirmed (no CLI or MCP server from the Forgejo project
  itself). Verify at bootstrap.
- Community: `fj` (forgejo-cli), under the `forgejo-contrib` organization,
  actively developed, pre-1.0: https://codeberg.org/forgejo-contrib/forgejo-cli
  Community MCP servers exist under several similar names and one has
  moved hosts: check the exact source, maintainer, and release history
  before proposing any, and prefer `fj` or the adapter.
- Thin adapter (often the right choice here): a few REST v1 calls taken
  from the instance's OpenAPI document: read repository, read and create
  issue, comment, list, create, and update pull request, read statuses.

## Ticket transitions

- Issues are open or closed. Closing and reopening keywords (close, fix,
  resolve; reopen) in a pull-request description or commit message act
  when the pull request is merged, if the person merging may close the
  issue; administrators can change the keyword list.
  https://forgejo.org/docs/latest/user/linked-references/
- Finer states are labels by the team's convention. Whether closing
  requires the default branch, and the API calls for state and labels:
  verify at bootstrap from the instance's OpenAPI document.
