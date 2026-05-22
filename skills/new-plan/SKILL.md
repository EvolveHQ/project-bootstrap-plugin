---
name: new-plan
description: Add a work item to the plan/todo queue in a documentation-led repo — name the owning ADR(s), scope, exit criteria mapped to acceptance criteria, dependencies, and queue position. Use when the user says "add a plan item", "queue this work", "create a todo for ADR X", "new plan", or invokes /new-plan.
---

# new-plan

Add one item to the implementation queue.

## Step 0 — Preconditions and context

1. Confirm the repo is bootstrapped and a `plan/` queue exists. If the
   repo was bootstrapped without a plan folder (Q4a = skip), stop and
   say so — there is no queue to add to.
2. Read `CONVENTIONS.md` for the plan-folder convention and the
   completion event, and `plan/README.md` if present.
3. `ls plan/todo/` to learn existing numbers and priority ordering.

## Step 1 — Identify the owning ADR(s)

- Ask which ADR(s) this work implements. Validate they exist in
  `adr/`.
- Normally a queue item tracks an **Accepted** ADR. If the named ADR is
  still `Proposed`, warn — you can queue ahead of acceptance, but the
  work is not yet authorised. If it has no ADR at all, suggest running
  `/new-adr` first; plan items should trace to a decision.

## Step 2 — Pick number and position

- `plan/todo/NNNN-<slug>.md`, zero-padded. **Lower numbers run first** —
  ask where this sits in priority and renumber neighbours only if the
  user wants it inserted ahead of existing items.

## Step 3 — Write the item

The file names, at minimum:
- Owning ADR(s) by relative path.
- Scope — what is in, what is explicitly out.
- Exit criteria — map directly to the ADR's numbered acceptance
  criteria where possible.
- Dependencies — other plan items or ADRs that must land first.

## Step 4 — Commit

Conventional Commit. If the file references an ADR by number in its
body that is fine (the plan queue is internal — the ADR-privacy rule
only forbids ADR references in **user-visible product** surfaces).
