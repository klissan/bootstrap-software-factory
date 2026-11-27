# Decision interview: question list

Read at SKILL.md §4. Skip every question that discovery or the decision
logs already answer.

Resolve only decisions not already established:

- Which repositories and modules belong to this factory?
- Which existing AI rules must remain authoritative?
- May target repositories be modified, or only the factory repository?
- What happens to uncommitted or unpushed work found in the checkouts?
- Which task inputs are accepted (tracker reference, free text), and may the
  factory create tracker items?
- What is the desired completion point: local changes, PRs, merge, deployment?
- Which external writes are already authorized?
- What acceptance criteria and compatibility guarantees matter?
- Which commands require special runners, services, or licenses?
- Which runtime is primary; which runtime, model, and effort review?
- What counts as approval on this host, and who can give it?
- Advisory gates, or host-enforced branch protection and required CI?
- Where do decision logs live where no convention exists?
- What may run in parallel, and what resources are shared?
- What incident data may be accessed, and in which environments?
- Who owns unresolved product and operational decisions?
- What limits apply to time, attempts, concurrency, and cost?

## Workflow conventions (cheap; ask together, one batch)

Offer the default and its tradeoff; skip what discovery already settled.

- Review publication: `comments`, `summary`, or `local`?
  Default `summary`: one readable record on the change request without a
  thread of agent rounds. `comments` is the most transparent but noisy;
  `local` keeps the change request clean but reviewers must open a file
  or trust the one-line verdict. Where does the durable record live?
- Ticket transitions: should the factory move tickets at all?
  Default no: nothing changes in the tracker without the user, at the
  cost of manual moves. If yes: on which events, to which states, in
  which projects, and under whose account? Does the host or tracker
  automation already move tickets on merge?
- Branch naming: confirm the convention inferred from existing branches
  and merged change requests, or choose one. Default when none exists:
  `<type>/<ticket>-<slug>`, at most 60 characters, lowercase ASCII and
  dashes. A stricter pattern is easier to enforce; a looser one fits
  mixed teams. Any per-repository override?
- Commit messages and change-request titles: ticket-key prefix, a
  conventional-commits style, a closing keyword? Default: follow what the
  merged history shows; otherwise ticket reference in the title and a
  closing keyword in the description only if the host should close the
  ticket on merge.
- Should naming be a hard rule (a guard) or only what the scripts produce?
