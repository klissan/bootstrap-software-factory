# Provider: Jira Cloud

Facts from support.atlassian.com (API tokens) and the Jira Cloud REST v3
reference, read 2026-11-27.
Docs: https://developer.atlassian.com/cloud/jira/platform/rest/v3/

## Distinctions to record

Site identity (site URL and cloud ID), authentication route, project keys,
issue fields including custom ones, rich-text format (Atlassian Document
Format in v3), pagination, workflow transitions per project.
Do not use Data Center endpoints or assumptions.

## Setup instructions to give the user

- If an Atlassian MCP connector or another verified tool is already
  available in the runtime, use its own sign-in and skip the token.
- Otherwise create an API token at
  https://id.atlassian.com/manage-profile/security/api-tokens
  Docs: https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/
- Prefer "API token with scopes" (recommended by Atlassian). Select only
  Jira scopes matching the authorized capabilities: read scopes for
  issue.read and issue.search; write scopes only for authorized comments,
  transitions, or issue creation. Confirm the exact scope names in the
  token dialog; do not guess them.
- Scoped tokens call the API through
  `https://api.atlassian.com/ex/jira/<cloudId>/...`; unscoped tokens use
  the site URL. Record which route is in use.
- Expiry: 1 to 365 days; default advice 90 days.
- Authentication is HTTP Basic with the account email and the token.
  Store both as references (for example an email variable and a token
  variable in the user environment or a secret store).
- The token acts as the user: prefer a dedicated account limited to the
  factory's projects when writes are authorized.

## Verify without printing

- `GET /rest/api/3/myself` returns 200 and the expected account.
- `GET /rest/api/3/project/<KEY>` for each allowed project.
- Send credentials by a header file or the environment, never on the
  command line; print only the status code and non-secret fields.

## Notes

- Transitions and required fields differ per project and issue type: read
  them from the API, do not assume names.
- Handle pagination and rate limiting explicitly.

## Ready-made tool or own adapter

Selection rule: official ready-made tool > well-maintained community tool
(only after the user's consent and a trust note: it runs with the user's
credentials) > thin adapter written by the factory. Whatever is chosen,
the glue around it (snapshots, reconciliation, idempotent posting,
evidence) stays in the factory's own scripts: it is repeated and needs no
model. Facts read 2026-11-27; anything not from the linked official page
is "verify at bootstrap".

- Official: Atlassian's hosted MCP server, `https://mcp.atlassian.com/v2/mcp`,
  OAuth 2.1 (API token optional), covering Jira among other cloud
  products. Some calls consume Rovo credits; an organization admin may
  control access.
  https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/
- Official: Atlassian CLI (`acli`), documented under the Cloud developer
  docs; which Jira commands it offers and how it authenticates were not
  confirmed: verify at bootstrap.
  https://developer.atlassian.com/cloud/acli/guides/introduction/
- Community: not needed when the official server is permitted.
- Thin adapter: REST v3 for read issue, search, comment, transition, and
  create, when the organization does not allow the MCP server or the
  factory needs scripted, model-free access.

## Ticket transitions

- Real workflow transitions, different per project and issue type. The
  REST v3 Issues group has "Get transitions" (those available for the
  issue in its current status to this user) and "Transition issue" (by
  transition identifier). Paths and required fields were not re-read in
  this pass: take them from the reference linked above.
- A transition may require fields (for example a resolution) or be
  hidden by permissions and conditions: discover per issue, never
  hard-code identifiers or state names across projects.
- A scoped token needs the Jira write scope for transitions.
- Development-tool integrations and automation rules may move tickets on
  their own: ask, and do not duplicate.

## Review threads

- None: Jira has no diff. Reviews live on the code host; at most a link
  or one-line verdict goes to the issue, if authorized.

## Parent and child items

- Hierarchy is expressed through the `parent` field; Atlassian has been
  replacing the older Epic Link and Parent Link fields with it for both
  company-managed and team-managed projects (announcements, not the
  current reference). Item types, hierarchy levels, and required fields
  vary by project: discover them from the project's create metadata.
- Issue links (blocks, relates to) are a separate mechanism; link types
  are instance-specific.
- Exact fields and calls: verify at bootstrap in the REST v3 reference.
