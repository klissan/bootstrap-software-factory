# Epics and multi-repository tasks

Read only when the input is an epic or a task spans more than one
repository, and when generating that path. Host specifics are in the
"Parent and child items" note of the chosen `provider-*.md`.

## Strategy (a user decision, `epics.strategy`)

- `children`: one tracked item per affected repository (and per component
  where a repository is large), each linked to a parent. Default: each
  child maps to one change request, one gate, one owner, and can be
  delivered, blocked, or reverted alone. Cost: more tracker items.
- `umbrella`: one item with a checklist. Fewer items, but progress, review,
  and blocking are not separable; fits small two-repository changes.
- `ask`: decide per task. Most flexible, one more question each time.

Every child has observable acceptance criteria and names its repository,
component, dependencies, and contract entries.

## Decomposition

Analysis first, mostly mechanical:

1. Scripts collect, per repository: the map from the repository profile,
   components and owners, the dependency graph between repositories and
   components, shared contracts and where each side implements them.
2. Read-only analysis runs in parallel per repository, each worker
   returning a bounded structured result: affected components, needed
   changes, risks, contract impact, rough size.
3. The model decides only the split: children, boundaries, order.
4. Contract first: agree and record each changed contract as one shared
   decision entry before any dependent child starts.
5. Make the dependency order explicit (a graph without cycles), plus the
   merge and deployment order. There is no atomic cross-repository
   release: each child must be backward compatible with the others'
   previous state, or the order and the compatibility window are stated.
6. Size limit: each child must fit the delivery workflow's attempt and
   cost limits; split a child that does not. Cap children per epic
   (`epics.max_children`); above it, propose phases.

The `epic-plan` script validates the plan file: every child has a
repository, criteria, a stable key, and a size estimate; the graph has no
cycles; every contract change has a decision entry; limits hold.

## Approval gate

By default the plan is shown to the user and approved BEFORE any item is
created. Creating and linking items are external writes (`issue.create`,
`issue.link`), each authorized and verified separately. The user may
waive the gate in configuration (`epics.plan_approval: false`); log that
as a user decision. A change to an approved plan (new child, changed
order or contract) is a scope change and returns to the gate.

## Creating children

- Stable key per child (epic identity plus repository plus component plus
  slug), stored in a hidden marker in the item and in the epic state.
- Reconcile first: search by parent link and by key; reuse what exists.
- The `child-create` script creates from a body file, links to the
  parent, records the created identifier, and after an uncertain result
  searches again instead of retrying.
- Where trackers are per repository, each child lives in its repository's
  tracker and the parent links across; where the host cannot link across,
  put the links in the bodies and record that.
- Follow the tracker's item types, labels, and components as configured;
  never guess a field the project requires: discover it.

## Execution

- Each child runs the normal delivery workflow in its own worktree, with
  its own evidence, review, and gate.
- A child starts only when its dependencies have reached the configured
  point (default: change request open and gate passed; stricter: merged).
- Concurrency: the existing limits across repositories and one writer
  per repository unless separate worktrees and resources are allocated.
- Epic state is a persistent file with a single writer: approved plan and
  its decision entry, children (key, identifier, repository, stage,
  links), dependency order, blockers, roll-up status. `epic-status`
  computes the roll-up from child state and real tracker and host state.
- Resume from the state file plus real state; never recreate a child
  because the conversation is gone.
- Partial completion is reported as such: which children are delivered,
  which are blocked and why. A blocked child holds its dependents;
  independent children continue. The parent is not closed by the factory
  unless transitions are enabled and every child is done.
- Integration: verify the revision tuple across the children that share
  a contract before calling the epic integrated.

## Decision log

The strategy, the approved plan, each contract change, any waiver of the
gate, and later plan changes are entries (decided-by user for approvals)
in the shared log, with a pointer entry in each affected repository's log.
