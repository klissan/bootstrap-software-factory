# Provider: GitHub

Facts from docs.github.com, read 2026-11-27. For GitHub Enterprise Server,
open the documentation for the deployed version and correct differences.

## Distinctions to record

Host (github.com or an Enterprise host), owner/repository, issues, PRs,
checks, reviews, repository visibility, branch protection, CI triggers,
and what the authenticated CLI or API can actually do.
"Approve" cannot be given by the account that authored the PR.

## Setup instructions to give the user

Preferred: the GitHub CLI with its own login.
- Install `gh` from https://cli.github.com and check `gh --version`.
- `gh auth login` (add `--hostname <host>` for an Enterprise host). The
  token is kept in the OS credential store.
- Git transport: let `gh auth setup-git` configure the credential helper,
  or use an SSH key.

Alternative: a fine-grained personal access token.
- Profile picture → Settings → Developer settings → Personal access
  tokens → Fine-grained tokens → Generate new token.
  Docs: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
- Resource owner: the account or organization that owns the repositories.
  Repository access: "Only select repositories", exactly the factory's.
- Repository permissions, only for authorized capabilities:
  Metadata read (always); Contents read, or read and write to push
  branches; Pull requests read and write to create and comment on PRs;
  Issues read, or read and write to create and comment on issues;
  Commit statuses and Actions read for CI results.
- Expiration: set one (default advice 90 days); the organization may cap
  it or require approval, and a pending token reads only public data.
- Store it as `GH_TOKEN` (github.com) or `GH_ENTERPRISE_TOKEN` (Enterprise
  host) in the user environment or a secret store; record only the name.
- Documented limits of fine-grained tokens: one resource owner per token,
  no Checks API, no contribution as an outside collaborator. If one
  applies, prefer `gh auth login`; a classic token is the last resort and
  needs a logged decision.

## Verify without printing

- `gh auth status` (never add `--show-token`).
- `gh api user --jq .login`
- `gh repo view <owner>/<repo> --json viewerPermission,visibility`
- Writes stay `configured_unverified` until an authorized write succeeds.

## Notes

- Pass issue and PR bodies with `--body-file`; search before creating.
- Branch protection and required checks need admin rights and are a
  separate, logged user decision, not part of the factory token.
- Check workflow triggers: push-only CI never reports on PRs.

## Ready-made tool or own adapter

Selection rule: official ready-made tool > well-maintained community tool
(only after the user's consent and a trust note: it runs with the user's
credentials) > thin adapter written by the factory. Whatever is chosen,
the glue around it (snapshots, reconciliation, idempotent posting,
evidence) stays in the factory's own scripts: it is repeated and needs no
model. Facts read 2026-11-27; anything not from the linked official page
is "verify at bootstrap".

- Official: the `gh` CLI (github.com and Enterprise Server). Default choice;
  scripts call it directly, which costs no context.
- Official: GitHub MCP server, https://github.com/github/github-mcp-server.
  Remote endpoint `https://api.githubcopilot.com/mcp/` (github.com only);
  the local server supports Enterprise Server through `--gh-host` or
  `GITHUB_HOST`. Has `--read-only` and `--toolsets`: use both. Propose it
  only if `gh` does not cover a needed capability.
- Community: not needed.
- Thin adapter: only for an endpoint `gh` lacks; use `gh api`.
