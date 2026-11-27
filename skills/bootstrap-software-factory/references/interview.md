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
