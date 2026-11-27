# Provider: GitLab self-managed

Facts from docs.gitlab.com (personal access tokens), read 2026-11-27. Menu
labels and features vary by version and tier: confirm against the
documentation for the DEPLOYED version.
Docs: https://docs.gitlab.com/api/rest/
Tokens: https://docs.gitlab.com/user/profile/personal_access_tokens/

## Distinctions to record

Instance URL, version and tier, project ID or URL-encoded path, merge
requests, pipelines and their triggers, approval rules (tier-dependent),
protected branches, supported features. Do not assume the newest features.

## Setup instructions to give the user

- Personal access token: avatar → Edit profile → Access → Personal access
  tokens → Generate token (labels differ in older versions).
- Prefer a project or group access token where the tier and version offer
  one, so access is limited to the factory's projects; otherwise a
  dedicated account. See the access-token scopes page linked from the docs.
- Scopes, only for authorized capabilities: `read_api` and
  `read_repository` for reads; `api` when the factory creates or comments
  on merge requests or issues (it also grants Git read and write);
  `write_repository` to push over HTTPS when `api` is not granted.
- Expiry: mandatory in current versions, default and default maximum 365
  days; set 90 days unless policy says otherwise.
- Use: header `PRIVATE-TOKEN: <token>`.
- CLI option: `glab` with `glab auth login --hostname <host>`; confirm the
  installed version supports the instance.
- Git transport: SSH key or credential helper.

## Verify without printing

- `GET <base>/api/v4/version` for the version.
- `GET <base>/api/v4/user` returns 200 and the expected account.
- `GET <base>/api/v4/projects/<id-or-encoded-path>` for each project.
- Send the header from a file or the environment; print status codes only.

## Notes

- "Approved" depends on approval rules and tier; record who may approve
  and whether the author can.
- Pipelines for merge requests run only if the CI configuration defines
  them; check before promising CI results.

## Ready-made tool or own adapter

Selection rule: official ready-made tool > well-maintained community tool
(only after the user's consent and a trust note: it runs with the user's
credentials) > thin adapter written by the factory. Whatever is chosen,
the glue around it (snapshots, reconciliation, idempotent posting,
evidence) stays in the factory's own scripts: it is repeated and needs no
model. Facts read 2026-11-27; anything not from the linked official page
is "verify at bootstrap".

- Official: `glab`, GitLab's open-source CLI; works with GitLab.com,
  Dedicated, and Self-Managed.
  https://docs.gitlab.com/editor_extensions/gitlab_cli/
- Official: GitLab MCP server at `https://<host>/api/v4/mcp`, offered for
  Self-Managed too; experiment in 18.3, beta in 18.6, generally available
  in 19.5. An administrator must allow access for the instance. OAuth with
  the `mcp` scope or a fine-grained token; legacy personal, project, and
  group tokens are rejected. Check the deployed version first.
  https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server/
- Community: not needed.
- Thin adapter: REST v4 for anything the deployed version lacks.
