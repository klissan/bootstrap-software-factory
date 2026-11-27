# Rerun, audit, and upgrade

Read in audit, upgrade, or resume mode, and whenever generated files
already exist at the destination.

Before proposing changes, read the previous run's provenance (skill
revision, generated files, content hashes), bootstrap progress, and
decision logs. Compare with the current skill revision and say what the
newer revision adds.

For generated files, retain provenance (skill revision, run date) and
content hashes.

On rerun:
- Reuse matching configuration.
- Detect user edits by comparing with the recorded hashes.
- Update only owned sections or perform a reviewable three-way merge.
- Preserve unknown fields and unrelated files.
- Show conflicts rather than overwriting them.
- Never reset, clean, or stash user changes automatically.
- Expect concurrent edits: re-read each file immediately before writing.

Decision logs: append only; a log from an earlier run is the existing
convention for its scope, even if a newer revision would place it
elsewhere. Moving or splitting it needs a user decision. Reconcile changed
facts through superseding entries.

Scripts: rerun the fixture tests before and after changing a script. Do not
edit a control script while a job that will start it is queued or running.

Re-verify rather than inherit: integration capabilities, runtime profiles,
reviewer invocation, and baselines are re-checked when tool versions
changed; otherwise keep their recorded verification time and say so.

Rerun the doctor and the context-budget check and report the differences
from the previous run.

When changes to target repositories are not authorized, generate proposed
patches and keep central profiles usable without applying those patches.

Log the upgrade itself: from which revision to which, what changed, what
was kept because of user edits.
