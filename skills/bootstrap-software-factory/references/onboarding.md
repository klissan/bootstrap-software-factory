# Onboarding the chosen tools

Read after the interview has fixed tracker, host, primary runtime, and
reviewer. Then read only the matching `provider-*.md` and `runtime-*.md`.

## Rules

- Give the user concrete steps for every chosen connection and tool; do not
  say "configure access".
- The reference files hold facts confirmed in official documentation on the
  date stated in each file. Self-hosted products differ by version: open the
  official documentation for the DEPLOYED version and correct any menu
  path, scope name, or endpoint before writing it into a setup file. If you
  cannot confirm a detail, link the official page and say "confirm there";
  never invent a URL, menu path, scope, or flag.
- Ask for the minimum permissions the authorized operations need. Derive
  them from the capability list (SKILL.md §3): read-only capabilities get a
  read-only credential; each write capability is added only if authorized.
- Recommend an expiry (default 90 days unless policy or the product says
  otherwise) and record the renewal date, not the secret.
- The user creates the credential and stores it. Never ask for a secret in
  the chat; if one is pasted, say it must be revoked and rotated, and do
  not write it anywhere.
- Store references only: environment-variable names, credential-store or
  keychain entry names, CLI login state. No secret in generated files,
  configuration, remote URLs, prompts, logs, decision logs, or task state.
- Prefer, in order: the tool's own login into the OS credential store; a
  secret manager or credential helper; a user-level environment variable.
  A plain-text file is a last resort and must be ignored by git.
- Prefer a dedicated least-privilege account or a project-scoped token
  where the product lets a personal token inherit all of the user's rights.
- Git transport: SSH key or credential helper. Never a token in a remote URL.

## Recommended tooling

In the same step, propose the items from `recommended-tooling.md` that fit
the chosen runtimes, one by one, with purpose, cost, and what leaves the
machine. Accepted tools get a `docs/setup/<tool>.md`, a checklist line,
and a doctor check; declined ones are only logged as declined.

## Generated output

`docs/setup/<connection-or-runtime>.md`, one per chosen item:

1. What it is for and which capabilities depend on it.
2. Install: tool, minimum version, official install link, version check.
3. Create the credential: exact location (menu path and official URL),
   permissions to select, expiry.
4. Store it: where, under which reference name.
5. Verify: the doctor command and what a pass looks like.
6. Rotation and revocation: where, renewal date, what to rerun afterwards.
7. Unresolved or version-dependent details, with documentation links.
8. Source documentation and the date it was checked.

`docs/setup/README.md`: a checklist with one line per item and a status:
`done-verified`, `done-unverified`, `todo-user`, `blocked`, `not-needed`.
Statuses come from the doctor, not from the user's say-so.

## Doctor

A script (see `scripts.md`) that reads the factory configuration and local
overrides and checks, per item:

- tool present and version at least the required one;
- configured paths exist; shell and interpreter versions match;
- credential reference resolves (variable set and non-empty, CLI logged
  in, store entry present) without reading the value into output;
- a scoped live read succeeds (identity endpoint, one allowed repository
  or project) and, for self-hosted products, the server version;
- reviewer CLI present, logged in, and the configured model in its catalog;
- each accepted optional tool is configured and answers one read call.

Output: JSON list of `{id, status: pass|fail|skip, hint}` where the hint
names the setup file to open. Print only pass, fail, or skip and
non-secret identifiers; never a credential, header, or authenticated URL.
Pass secrets to child tools by environment or standard input, never as
arguments. Performs no writes. Exits non-zero if a required item fails.
Write capabilities stay `configured_unverified` until an authorized write.

Run it: after onboarding, at the start of validation, on resume when a
call fails with an authentication error, and after rotation.
