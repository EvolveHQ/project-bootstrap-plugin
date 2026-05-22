# Plan

This folder holds the project's implementation queue — one file per
unit of work. The queue mirrors the ADR catalogue (`INDEX.md`) but
tracks the human ordering of work, not the ADR catalogue ordering.

## Layout

- `plan/todo/NNNN-<slug>.md` — pending work, ordered by priority
  (lower numbers run first). Each file names the owning ADR(s), the
  scope, the exit criteria, and any dependencies.
- `plan/done/<YYYY-MM-DD>-<slug>.md` — shipped work, ordered
  chronologically. The `git mv` from `todo/` to `done/` is the
  completion event; the file's body is amended with a "Shipped"
  footer naming the HEAD SHA and any artefact id.

## Convention

- A pending item gets a `plan/todo/` file BEFORE work starts.
- When work ships, the file is `git mv`'d to `plan/done/` with a new
  date prefix and the body amended with the shipped footer.
- A small fix that doesn't justify a plan file (a typo, a one-line
  tweak, a dependency bump) skips the ceremony. Use judgement.
- The status of the owning ADR(s) advances when the work ships:
  `Accepted` → `Implemented`.

## Status semantics on the owning ADRs

| ADR status | Meaning |
|---|---|
| Proposed | Draft; decision authored but not yet approved. |
| Accepted | Decision approved; implementation authorised. Sits in `plan/todo/`. |
| Implemented | Shipped per the project's completion event. Sits in `plan/done/`. |
| Superseded | Replaced by another ADR (named in `superseded-by:`). |
| Deprecated | Was real; the world moved on; no successor. |

See `CONVENTIONS.md` §Status lifecycle for the canonical definition.
