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
