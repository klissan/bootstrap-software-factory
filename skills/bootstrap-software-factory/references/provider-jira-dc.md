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

## Ready-made tool or own adapter

Selection rule: official ready-made tool > well-maintained community tool
(only after the user's consent and a trust note: it runs with the user's
credentials) > thin adapter written by the factory. Whatever is chosen,
the glue around it (snapshots, reconciliation, idempotent posting,
evidence) stays in the factory's own scripts: it is repeated and needs no
model. Facts read 2026-11-27; anything not from the linked official page
is "verify at bootstrap".

- Official: none found. Atlassian's hosted MCP server and `acli` are
  documented for Cloud; nothing in the pages read says they support Data
  Center, and a public feature request for an official Jira Data Center
  MCP server is open. Do not assume one exists; verify at bootstrap.
- Community: `mcp-atlassian` (https://github.com/sooperset/mcp-atlassian)
  states support for Server and Data Center 8.14+ with a personal access
  token. Unaffiliated with Atlassian, and it receives a token that
  inherits all of the user's rights: propose only with consent, a pinned
  version, and a dedicated low-privilege account. Many similarly named
  packages exist; confirm the exact source.
- Thin adapter (the default): REST v2 for read issue, search, comment,
  transition, and create, with the Bearer token.

## Ticket transitions

- Real workflow transitions, different per project and issue type:
  list the transitions available for the issue in its current status,
  then perform one by its identifier. The REST v2 issue resource provides
  both; the exact paths and bodies were seen only in secondary sources:
  verify in the REST reference for the deployed version.
- Transitions may require fields or be hidden by conditions and
  permissions of the account in use: discover per issue.
- Integrated code hosts and workflow triggers may move tickets on their
  own: ask, and do not duplicate.
