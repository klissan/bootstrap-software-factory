# Provider: Jira Data Center

Facts from Atlassian's "Using personal access tokens" page, read
2026-11-27. Menu paths and endpoints vary by version: confirm against the
documentation for the DEPLOYED version.
Docs: https://developer.atlassian.com/server/jira/platform/
Tokens: https://confluence.atlassian.com/enterprise/using-personal-access-tokens-1026032365.html

## Distinctions to record

Base URL including any context path, deployed version, supported
authentication, custom fields, workflows, network/VPN/TLS prerequisites
(internal certificate authority). Do not substitute Cloud APIs.

## Setup instructions to give the user

- Personal access tokens need Jira 8.14 or later. On older versions, or
  where an administrator disabled them, ask which authentication the
  instance supports; do not fall back to a password in a file.
- Create: avatar (top right) → Profile → Personal access tokens →
  Create token.
- The token has no scopes: it inherits all of the user's permissions.
  For least privilege use a dedicated account restricted to the factory's
  projects, read-only unless writes are authorized.
- Expiry: optional; set one (default advice 90 days; the instance maximum
  is 365 days by default and administrators may change it).
- Use: header `Authorization: Bearer <token>`.
- Store the token reference and the base URL; the base URL is not secret.

## Verify without printing

- `GET <base>/rest/api/2/serverInfo` for the version.
- `GET <base>/rest/api/2/myself` returns 200 and the expected user.
- `GET <base>/rest/api/2/project/<KEY>` for each allowed project.
- Send the header from a file or the environment; print only status codes.

## Notes

- TLS failures against an internal authority are a setup item (trust
  store), never a reason to disable certificate verification.
- Rich text is wiki markup, not Atlassian Document Format.
