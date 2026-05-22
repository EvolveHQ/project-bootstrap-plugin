# Handoff

Entry point for a fresh agent picking up this repo. Read these files
in order before any tool calls:

1. `AGENTS.md` — hard rules. Read in full.
2. `CONVENTIONS.md` — authoring rules + ADR status semantics + plan
   folder convention.
3. `plan/README.md` — the queue's own README.
4. `_agent/CURRENT_FOCUS.md` — short live snapshot of any in-flight
   session: active branch, active queue item, blockers, uncommitted
   work.
5. `INDEX.md` — the ADR catalogue. Look up an ADR's filename and
   dependency chain. Sorted by ADR number, not by work order.
6. Tail of `_agent/WORKLOG.md` (last 30 lines) — confirms what landed.
7. The next queue item at `plan/todo/NNNN-*.md` — and the ADR(s) it
   names — read in full before implementing.

## Stop conditions

Stop and surface the issue rather than guessing if:

- The verify gate fails.
- The queue is empty.
- A `plan/todo/` item references an ADR whose status is not Accepted.
- An ADR's acceptance criteria are ambiguous or untestable as written.
- Two `plan/todo/` items at the same priority both target the same
  files (lock contention).
