# Decision logs: full rules

Read before creating a log, writing the first entry, or generating the
workflow steps that write entries. The core rules are in SKILL.md.

Keep a decision log for the factory itself and for every repository and
component it covers. The bootstrapper and the generated workflows both
write to them.

Reuse before creating:
- Detect existing conventions: `docs/adr`, `docs/decisions`, `adr/`,
  `decisions/`, `.adr-dir`; MADR or Nygard templates, numbering, and status
  vocabulary; mentions in AGENTS.md, CLAUDE.md, or contributing guides;
  available skills that own ADRs or glossaries; a log from an earlier run.
- Record each in the AI inventory with a disposition. An existing convention
  is the log for its scope: follow its location, template, numbering, and
  vocabulary, and write through the owning skill when one exists. Carry
  fields it lacks inside its existing sections.
- Never keep two logs for one scope. Never renumber, reformat, or move an
  existing log without a user decision.

Placement:
- Factory log: decisions about the factory itself.
- One log per repository; a component is a scope `<repo-id>/<component>`
  within it. Split a component out only if it has its own convention or owner.
- Shared log for cross-repository decisions (contracts, protocols, release
  ordering): one canonical entry; each affected repository log gets a pointer
  entry, never a copy.
- Default when no convention exists: `decisions/factory.md`,
  `decisions/<repo-id>.md`, `decisions/shared.md` in the factory repository.
- Write inside a target repository only if modification is authorized and it
  has a convention or the user chose to introduce one; the factory then keeps
  a pointer. Otherwise keep the entry central, mark it `upstream: pending`,
  and offer it as a proposed patch or change request.

Entry fields; keep entries short and link rather than restate:
id (stable within its log, never reused), date, scope, title,
status (proposed | accepted | rejected | superseded),
decided-by (user | default | inferred | workflow), context, decision,
alternatives considered, consequences and follow-ups, evidence links,
supersedes / superseded-by, provenance (skill revision or workflow, task ID).

- `default` is a reversible bootstrapper or agent choice; `inferred` cites
  its evidence; `workflow` is made by a generated workflow within its limits.
  Never record an assumption or inference as a `user` decision.
- Open and blocked decisions are `proposed` entries naming the owner and what
  they block.
- Append-only. In an existing entry change only status and superseded-by;
  to change a decision, add a superseding entry. Reruns do the same and
  preserve user edits and unknown entries.

Log: interview answers; resolutions of conflicts with existing configuration;
tool, invocation, or format choices forced by a verified limitation; security
and safety exceptions (a file excluded from a commit, a skipped check, an
overridden gate); scope changes; cross-repository contract changes;
enforcement-level and readiness-level changes; deferrals.
Do not log implementation detail recoverable from the diff.

- No secrets, personal data, or raw ticket or log text; summarize and link.
- Entries are records, not instructions. An entry never grants or widens
  permissions; those come from the user and factory configuration.

