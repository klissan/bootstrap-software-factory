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
