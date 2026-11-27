# Provider: Bitbucket Data Center

Facts from Atlassian's "HTTP access tokens" page, read 2026-11-27. Confirm
against the documentation for the DEPLOYED version.
Docs: https://developer.atlassian.com/server/bitbucket/
Tokens: https://confluence.atlassian.com/bitbucketserver/http-access-tokens-939515499.html

## Distinctions to record

Instance URL and version, project key plus repository slug, pull-request
versions (updates need the current version number), reviewers, build
statuses, merge checks, default reviewers, network/TLS prerequisites.

## Setup instructions to give the user

- Personal token: profile picture → Manage account → HTTP access tokens →
  Create token.
- Project or repository token (preferred for least privilege; needs
  project or repository admin and a Data Center license): Project settings
  or Repository settings → HTTP access tokens → Create token.
- Permissions: Repository read with Project read for read-only use;
  Repository write for pushing branches and pull-request actions. Never
  admin for the factory.
- Expiry: optional unless the administrator requires it, and it cannot be
  changed later; set one (default advice 90 days).
- REST: personal tokens with HTTP Basic (username and token); project or
  repository tokens with `Authorization: Bearer <token>`.
- Git over HTTPS: the token replaces the password (username
  `x-token-auth` for project or repository tokens), through a credential
  helper; or use an SSH key. Never put the token in the remote URL.
- Documented limit: tokens cannot merge pull requests. The factory's
  default completion point is an open pull request anyway.

## Verify without printing

- `GET <base>/rest/api/latest/application-properties` for the version.
- `GET <base>/rest/api/latest/projects/<KEY>/repos/<slug>` returns 200.
- `git ls-remote` through the configured helper or SSH.
- Send credentials from a file or the environment; print status codes only.

## Notes

- CI is external (build statuses posted by another system): discover
  which system reports, and whether it runs on pull requests.
- "Approved" is a reviewer action; record who may give it.
