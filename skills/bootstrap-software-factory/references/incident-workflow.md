# Incident workflow: generation details

Read only when the incident workflow is in scope. The non-negotiable rules
are in SKILL.md §8.

## Stages

intake → scope → evidence → hypotheses → discriminating checks
→ findings → optional remediation → regression coverage

Capture:
incident ID, impact, environment, time window and timezone,
service identifiers, deployed versions, recent changes, and correlation IDs.

Resolve deployed revisions before drawing code-level conclusions.
If the revision is unknown, make that uncertainty explicit.

Discover log, trace, metric, deployment, and database capabilities.
Bind each to an environment and permitted data scope.

## Data access

Prefer production diagnosis with enforced read-only credentials.
For database access:
- Use a read-only role and read-only transaction where supported.
- Bound statement duration, returned rows, and query scope.
- Prefer a replica or snapshot where appropriate.
- Do not treat a SELECT prefix or regex as a security boundary.
- Avoid routines with possible side effects and unbounded scans.
- Never change production data as part of diagnosis.

For logs and traces:
- Bound time ranges and result volumes.
- Record pagination, sampling, retention gaps, and truncation.
- Redact secrets and personal data before sharing with other runtimes.
- Keep sensitive raw evidence out of source control.
- Record evidence links, acquisition times, and retention expectations.

## Reasoning

Maintain a hypothesis table:
claim, supporting evidence, contradicting evidence, next check,
and status: proposed, supported, refuted, or unresolved.

Distinguish symptoms, contributing factors, and established causes.
Do not infer causality solely from temporal correlation.
If evidence is insufficient, report the boundary of knowledge and next steps.

## Action

Separate diagnosis from mitigation, code repair, and production action.
Reuse explicit existing authorization; otherwise obtain authorization for
the specific operational action after preparing a reviewable proposal.

Log decisions, not findings: granted data-access scope, mitigation and
remediation choices, accepted risks, authorized operational actions.
Link entries from the incident record; resume reads them.

For code repairs, enter the development workflow with a reproducer.
For postmortems, record impact, timeline, evidence, causes/uncertainties,
actions, owners where known, and regression coverage.

Propose improvements to factory procedures after incidents; where the
cause was a broken invariant a script could have caught, propose a guard
(`guards.md`, and "After a failure" in `delivery-workflow.md`).
Do not automatically promote one incident's workaround into a global rule.

Mechanical steps (bounded queries, redaction, evidence capture, deployed
revision lookup) follow `scripts.md`.
