# Review publication mode `inline`: threads on the diff

Read only when `review.publication` is or may become `inline`. The other
modes are in `delivery-workflow.md`. Host specifics are in the "Review
threads" note of the chosen `provider-*.md`.

## What it is

Each reviewer finding becomes one thread anchored to a file and line of
the reviewed revision, in the host's native review channel. The author
side replies and fixes; a thread is resolved only when the fix is
verified. May be combined with `summary` (`review.publication: inline`
plus `review.summary: description`): threads carry the detail, the
description carries the verdict.

Recommend it where the host has resolvable threads (confirmed from
official docs: GitLab discussions, GitHub review threads via GraphQL
resolve/unresolve; Bitbucket Data Center and Forgejo: verify at
bootstrap): the noise concern is
answered by resolution, not by silence. A reader sees open threads only,
and the history stays attached to the code. Still ask the user.

The change request must exist before the first round, so it is opened
early as a draft, as in `comments`. The reviewer call stays local and
headless; the full transcript stays local; evidence stays bound to the
commit; the stage stays mandatory.

## Findings

The reviewer returns structured findings, not prose to be parsed loosely:
stable finding id (hash of rule or title, file, and normalized quoted
line, not of the line number), severity (blocking, should-fix, note),
file, line or range, side, the reviewed revision, message, suggested fix
if any. Findings without a valid file and line in the diff become
file-level or general comments. Cap the number published per round and
say how many were withheld; notes may be folded into the summary.

## Rounds

1. Reconcile first: list existing threads, read the hidden marker
   (finding id, revision, round) in each, and build the map finding →
   thread. Never post before reconciling.
2. New finding: create one thread at the anchor on the reviewed revision.
3. Author side: fix, commit, reply in the thread with what changed and
   the fixing revision. The author's agent never resolves its own thread.
4. Next reviewer round on the new head, per existing thread:
   fixed and verified → resolve (or, where the host has no resolve,
   reply "verified at <revision>" and record it); not fixed → reply, keep
   open; regressed after being resolved → reopen the same thread;
   the finding still stands but the anchor moved → reply in the existing
   thread with the new location, or re-anchor if the host allows. Never
   open a second thread for the same finding id.
5. The gate requires: no open blocking thread created by the reviewer,
   verification and review bound to the current head. Threads opened by
   humans are reported, never resolved by the factory.

Anchors go stale: after new commits the host may mark a thread outdated
or hide it. Outdated is not resolved. The script matches by marker, not
by position.

## Posting rules

- One script (`review-publish`, mode `inline`) does all of it; see
  `scripts.md`.
- Idempotent: the hidden marker per finding and revision makes a rerun a
  no-op; after an uncertain result, list threads before retrying.
- Sanitize (no local paths), refuse on secret-like content, never quote
  a suspected secret from the diff in a comment.
- Bodies go through files or request bodies, never a command line.
- Three write kinds, each authorized and verified separately: create a
  thread (`change.comment`), reply (`change.comment`), resolve or reopen
  (`change.resolve`). Without resolve authorization, the script replies
  with the verdict and leaves resolution to a human; say so in the report.
- Host cannot anchor to a line, or the anchor is rejected: fall back to a
  flat comment starting with `file:line`, with the same marker, and
  record that the thread is unanchored.
- Rate limits and pagination when listing threads are handled explicitly;
  a partial list blocks posting.

## Configuration

`review.publication: inline`; `review.summary` (none, description,
comment); severities that open a thread; maximum threads per round;
whether the factory may resolve; fallback behavior. Log the choices.
