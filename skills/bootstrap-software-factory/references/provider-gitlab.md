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

## Ticket transitions

- Issues are open or closed. Closing patterns (close, fix, resolve,
  implement and their forms) in a commit message or merge-request
  description close the issue when merged into the default branch; a
  project can turn this off. Cross-project form: `group/project#N`.
  https://docs.gitlab.com/user/project/issues/managing_issues/
- Finer states: Status (Premium and Ultimate), or labels and board lists.
  Discover which the project uses. API calls for state, status, and
  labels: verify at bootstrap against the deployed version.

## Review threads

- Supported: diff threads with notes, resolve and unresolve.
  https://docs.gitlab.com/api/discussions/
- Create: `POST /projects/:id/merge_requests/:merge_request_iid/discussions`
  with `body` and `position[base_sha]`, `position[start_sha]`,
  `position[head_sha]`, `position[position_type]` (`text`),
  `position[new_path]`, `position[old_path]`, and `position[new_line]`
  or `position[old_line]` (both for unchanged lines).
- Reply: `POST .../discussions/:discussion_id/notes`. Resolve or reopen:
  `PUT .../discussions/:discussion_id` with `resolved`. List:
  `GET .../discussions`.
- Take the three revisions from the merge request's current diff version;
  how that is read on the deployed version: verify at bootstrap.

## Parent and child items

- Epics (Premium and Ultimate) can have child epics and issues; details
  of adding children and cross-project scope are on the child-items page,
  not confirmed here: verify at bootstrap, including the tier.
  https://docs.gitlab.com/user/group/epics/manage_epics/
- Without epics: linked issues or task items; calls and availability on
  the deployed version: verify at bootstrap.
