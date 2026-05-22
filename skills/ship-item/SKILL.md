---
name: ship-item
description: Ship a plan/todo item in a documentation-led repo — run the verify gate, integrate per the repo's model (fast-forward or PR), git mv todo→done with a shipped footer, advance the owning ADR(s) to Implemented, regenerate INDEX, append WORKLOG, update the live snapshot. Use when the user says "ship this", "complete the plan item", "mark done", "close out the queue item", or invokes /ship-item.
---

# ship-item

Execute the completion event for one queue item. This is the most
order-sensitive operation in the system — follow the steps exactly.

## Step 0 — Preconditions and context

1. Confirm the repo is bootstrapped with a `plan/` queue.
2. Read `CONVENTIONS.md` and `AGENTS.md` for: the **integration model**
   (direct-to-main fast-forward vs. PR-based with required CI), the
   **verify gate** command, the **multi-agent mode**, and the Git
   contract (signed commits, tags, trailers).

## Step 1 — Select the item

Default to the lowest-numbered `plan/todo/` file, or the one the user
names. Read it and the owning ADR(s) in full.

## Step 2 — Verify

Run the repo's verify gate. **Require a pass.** Do not bypass with
`--no-verify` or equivalent. If it fails, stop, surface the failure,
fix the root cause, re-run.

## Step 3 — Integrate (per the repo's model)

- **Direct-to-main, fast-forward:** `git merge --ff-only <branch>` (or
  the work is already on `main`), then `git push origin main`. The
  verify gate ran locally in Step 2.
- **PR-based:** push the branch, `gh pr create --draft --fill`, wait
  for CI green (`gh pr checks --watch`), `gh pr ready`, then
  `gh pr merge` with the repo's strategy. Confirm the merge landed on
  `main` before continuing.

## Step 4 — Move the queue item

Once the change is on `main`:
- `git mv plan/todo/NNNN-<slug>.md plan/done/<YYYY-MM-DD>-<slug>.md`
  (today's date prefix).
- Amend the moved file with a footer: **"Shipped at HEAD `<sha>`"** plus
  any artefact id, image tag, deploy id, or PR link.

## Step 5 — Advance the ADR(s) and regenerate

- Advance each owning ADR's `status:` from `Accepted` to `Implemented`.
- Append a Revision History row if the status change is substantive
  (it is). Regenerate `INDEX.md` to match.

## Step 6 — Record

- Append a one-line `_agent/WORKLOG.md` row: branch, HEAD, verify
  result, any deferral.
- Update the live snapshot: `_agent/CURRENT_FOCUS.md` in single-checkout
  modes; in worktree mode (Q5 mode 3) remove this worktree's row from
  `_agent/IN_FLIGHT.md` instead (CURRENT_FOCUS is local-only there).

## Step 7 — Commit

Conventional Commit, `Rationale:` footer (touches an ADR). Group the
move + status advance + INDEX + WORKLOG into one coherent commit where
possible so the completion event is atomic in history.
