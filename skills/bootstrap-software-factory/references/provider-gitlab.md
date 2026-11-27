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
