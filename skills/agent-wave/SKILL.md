---
name: agent-wave
description: Orchestrate a wave of parallel agents over the plan/todo queue in a documentation-led repo — asks how many agents, the budget (items/waves, with hours as a soft cap), and whether to checkpoint after each wave or run continuously. Spawns isolated worktree subagents, assigns one queue item each, collects results. Use when the user says "spawn a wave of agents", "run the queue in parallel", "fan out the work", "agent wave", or invokes /agent-wave.
---

# agent-wave

Drive the implementation queue with parallel subagents, in waves.

## Honest scope (read first)

In-session subagents are bounded by **this session**, not by wall-clock
hours. So this skill measures budget reliably in **items and waves**,
with hours as a *soft cap* (stop starting new waves once elapsed time
passes it). For a true multi-hour unsupervised fleet that outlives a
session, use remote agents or a scheduled run of
`_agent/prompts/autonomous.md` (`/schedule`) instead — this skill points
you there rather than pretending to be it.

## Step 0 — Preconditions and context

1. Confirm the repo is bootstrapped with a `plan/` queue and that
   `_agent/prompts/autonomous.md` exists (a real verify gate is
   required — see `/bootstrap` Q8).
2. Read `CONVENTIONS.md` for the **multi-agent mode** and **integration
   model**.
   - **Mode 1 (single agent):** refuse. Parallel agents in one checkout
     clobber each other. Tell the user to re-bootstrap as mode 2/3, or
     run the autonomous prompt sequentially instead.
   - **Mode 2 (shared checkout):** allowed but warn — file contention is
     real; LOCKS must be respected and wave width kept low.
   - **Mode 3 (worktrees):** the intended mode. Each subagent works in
     its own isolated worktree.

## Step 1 — Ask the wave parameters (one at a time, recommend)

1. **Wave width N** — how many agents per wave.
   *Recommended: min(queue depth, 3)* — modest, keeps review tractable.
2. **Budget** — total items (or waves) to attempt this run.
   *Recommended: one wave, then reassess.* Optionally a soft hours cap.
3. **Supervision** — **checkpoint** (review after each wave, then
   continue) or **continuous** (run wave after wave until budget/queue
   exhausted).
   *Recommended: checkpoint* for the first run on a repo.

Cross-check: **continuous + direct-to-main** is risky — nothing gates
each merge. If the repo is direct-to-main, recommend either switching
this run to checkpoint, or that the repo move to PR-based so CI gates
every merge.

## Step 2 — Plan the wave

- Read `plan/todo/`; take the lowest-numbered N items with no unmet
  dependencies. Two agents must never get the same item or items that
  edit the same files — partition by item and, in mode 2, by LOCKS.
- Record the assignment in `_agent/IN_FLIGHT.md` (mode 3) so the wave is
  visible.

## Step 3 — Spawn the wave

Spawn N subagents in parallel, each in an isolated worktree
(`isolation: worktree`). Brief each with: its assigned queue item, the
instruction to follow `_agent/prompts/autonomous.md` end-to-end for that
one item (orient → implement → verify → integrate per the repo's model →
ship), and to report back a structured result (item, pass/fail, HEAD or
PR link, any blocker).

## Step 4 — Collect and checkpoint

When the wave returns, summarise: shipped, failed, blocked, with links.
Update `_agent/IN_FLIGHT.md` (remove finished rows). Then:
- **Checkpoint mode:** present the summary and ask whether to launch the
  next wave.
- **Continuous mode:** launch the next wave with the next N items,
  unless a stop condition fired.

## Stop conditions

Stop the run (do not start another wave) when any holds:
- Queue empty, or budget (items/waves) reached, or soft hours cap passed.
- Failure rate exceeds a sane threshold (e.g. >½ the wave failed) —
  something systemic is wrong; surface it.
- Repeated merge conflicts on the same files — the queue partition is
  wrong; re-plan.
- An item's ADR is not `Accepted`, or its acceptance criteria are
  ambiguous.

On stop, leave every worktree in a committed or cleanly-abandoned state,
record the outcome in the WORKLOG, and report.
