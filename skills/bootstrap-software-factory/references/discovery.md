# Discovery: inventory and integrations

Read at SKILL.md §2 and §3. The non-negotiable rules are repeated there.

## Inventory repositories and existing AI configuration

For each repository, record:

- Canonical remote, visibility, local checkout, default branch, inspected
  revision, and commits ahead of or behind the remote default branch.
- Working-tree status and existing changes.
- Submodules, nested repositories, and git-ignored files that builds need.
- Languages, build systems, package managers, runtime requirements.
- Build, test, lint, type-check, and development commands, with their
  working directories.
- CI definitions and their triggers (push, change request, manual),
  required checks, and branch protection.
- Deployment configuration, ownership, architecture docs.
- Decision-log or ADR conventions and glossaries.
- Dependencies on other repositories, published packages, and API contracts.
- Sources of deployment-version information.

When a repository holds several implementations or generations of code,
confirm which is live from recent commits, entry points, build files, and CI.
Do not infer it from the README or the dominant language.

Run each verification command at the inspected revision where safe and
record the baseline: exit status and passed, failed, and skipped counts.
A command that was not run is unverified.

Pre-existing local work:
- Uncommitted work and unpushed default-branch commits need a user decision;
  never commit, push, stash, or discard them on your own. Unpushed base
  commits leak into every change request.
- Before committing or pushing such work when asked, scan changed and
  untracked files and unpushed commits for secrets, oversized files, build
  output, local-only settings, editor directories, and nested repositories.
  Report secrets as file and key name only, never the value. Check
  repository visibility before a push. Also scan tracked source, and match
  credentials embedded in URLs and long literals, not only known token
  formats; a clean scan proves only what its patterns cover, so say so.
- To keep a secret out, commit a sanitized copy and leave the user's working
  file untouched. Log every exclusion.

Inspect applicable ancestor, root, and nested instructions.
Include AGENTS.md, CLAUDE.md, Cursor rules, skills, agent definitions,
rules, hooks, MCP configuration, permission settings, editor-specific AI
configuration, and referenced scripts where present.

Inspect hidden configuration directories deliberately within scope.
Read relevant ignore rules; avoid dependency caches and generated trees.
Do not print secrets from configuration files or authenticated remote URLs.

Create an AI inventory with:
path, scope, owner if known, purpose, consumer/runtime, executable behavior,
dependencies, conflicts, proposed disposition, and evidence.
Classify each item:
reuse, extend, bridge, replace, leave isolated, or unresolved.

Read hooks and scripts before executing them. Existing executable
configuration is not automatically safe or applicable to this environment.

Check:
- Conflicting instructions and different runtime precedence.
- Hooks that call unavailable commands or recursively invoke agents.
- Duplicate formatters, tests, or lifecycle hooks.
- Hard-coded paths, credentials, and global side effects.
- Skills that assume a single repository.
- Skill or command names that generated ones would collide with.
- Reviewer access to write tools.
- Commands that work only in CI or on a particular operating system.

Apply the host runtime's actual instruction precedence.
Do not assume a central factory instruction overrides repository rules.

If existing configuration conflicts with factory behavior, explain the
concrete conflict and request a decision only where needed.
Offer preserving local behavior, adding a scoped bridge, or migrating
specific files. Do not silently replace existing instructions.

For repositories without AI configuration, propose a minimal setup.
Do not assume every repository needs a full copy of factory instructions.
Central profiles may be sufficient.

## Discover and verify integrations

Use available service-specific tools first, then authenticated CLIs,
then a small API adapter where necessary.

Inspect actual tool schemas and installed versions.
Never invent MCP tool names, server packages, CLI flags, or credentials.
Read official documentation for the deployed product/version before
generating integration-specific executable configuration.

Read the `provider-*.md` file of each provider in use for its required
distinctions. Do not substitute Cloud APIs for Data Center APIs. Do not
assume Forgejo is interchangeable with GitHub or Gitea, that self-hosted
instances expose the latest features, or that hosting CI exists merely
because repositories are accessible.

Support multiple instances of the same provider.
Identify resources using provider + instance + native resource ID.

For each connection, record:
provider and instance URL; transport (MCP, CLI, or API); authentication
reference, never its value; allowed projects/repositories and operation
scopes; capability mappings to actual tools or adapter commands; pagination
and retry behavior; read/write authorization; verification time and
outcome; network/VPN/TLS prerequisites and unresolved limitations.

Use these normalized capabilities as internal concepts, not invented tools:
issue.read, issue.search, issue.create, issue.comment, issue.transition;
repo.read, branch.push, change.create, change.read, change.update,
change.comment, change.approve;
ci.read, review.read; logs.query, traces.query, db.query_readonly.

`change.resolve` (resolve or reopen a review thread) and `issue.link`
(parent, child, or dependency links) are separate write capabilities.

Mark each capability:
verified, configured_unverified, unavailable, or not_requested.
Each write capability is authorized separately; one does not imply another.

Verify reads against scoped real resources.
Verify writes only through an authorized action or disposable test resource.
Read access does not prove write access.
If writes cannot be tested, keep them configured_unverified.

Fill a runtime profile for the primary and the reviewer runtime
(`runtime-profile.md`). Verify agent CLIs with a real fixture call: record
the exact working command, the models and effort levels the installation
offers, read-only flags, and limitations such as a mode that rejects a
custom prompt. A runtime that is not installed stays configured_unverified.

Handle pagination, rate limits, timeouts, and partial results explicitly.
Before retrying a write with an uncertain result, look up whether it succeeded.
Never blindly retry PR creation or comments.

If credentials or network access are missing, prepare a usable configuration
with unresolved references and exact setup steps. Continue independent work.
Never call the connection verified.

## Candidate invariants (guards)

While reading each repository, collect invariants a script could check
(`guards.md`). Propose each with one evidence line (file and what it
shows); the user accepts or declines each. Look for:

- Generated or vendored files and "do not edit" headers.
- Lockfiles and files that change only through a tool.
- Frozen identifiers, formats, or files named in the repository's own
  instructions. A rule already written in AGENTS.md, CLAUDE.md, or a
  contributing guide ("never change X") is the strongest candidate: turn
  it into a mechanical check.
- Paired artifacts that must change together: data model and migration
  (detect the migration tool actually in use from build files and
  directories; never assume one), API schema and generated client, public
  interface and its documentation or changelog.
- Wire contracts shared between repositories: a change on one side
  without the paired change or a shared decision entry.
- Secrets and local configuration that must never be committed.
- "Tests change with code" or similar rules the project already states.
- Formatting or build outputs committed by mistake.

Evidence comes from the repository (history of past fixes and reverts,
review comments, CI steps, instructions), not from general habit. Do not
invent an invariant the project does not have. Propose the few with the
highest value and the lowest false-positive risk first; say what each
check would miss. Record accepted and declined candidates in the
decision log.

## Naming conventions

Infer branch, commit-message, and change-request title conventions from
evidence: existing remote branches, merged change requests, contributing
guides, CI branch filters, commit or push hooks, and protected-branch
patterns. Propose the observed convention as the default; note
exceptions. If there is none, propose a simple deterministic one. Never
invent a convention the history contradicts.
