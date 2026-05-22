# AGENTS.md

This file provides guidance to coding agents working in this repository.

## What this repository is

<One paragraph. Project purpose, current baseline, what the artefacts in
this repo represent.>

## Repository structure

- `adr/0000-template.md` — canonical ADR template.
<!-- If technology-ADR split (Q2): also include `adr/NNNN-template.md` for technology ADRs. -->
- `adr/NNNN-<kebab-slug>.md` — one ADR per decision, contiguous
  numbering, no gaps.
- `INDEX.md` — table regenerated from every ADR's metadata block.
- `CONVENTIONS.md` — authoring rules (read before editing anything).
- `plan/todo/NNNN-<slug>.md` — pending work, lower numbers run first.
- `plan/done/<YYYY-MM-DD>-<slug>.md` — shipped work, chronological.
- `_agent/` — multi-agent coordination: `ROLES.md`, `LOCKS.md`,
  `WORKLOG.md`, `CURRENT_FOCUS.md`, `HANDOFF.md`, `prompts/`.
<!-- If GLOSSARY.md (Q7): also include `GLOSSARY.md` — shared terms. -->
<!-- If domains/ (Q7): also include `domains/<slug>/README.md`. -->

## Hard rules when editing ADRs

These come from `CONVENTIONS.md` and override default behaviour:

- **One decision per ADR.** Splits become new ADRs that supersede;
  never expand scope inside an existing one.
- **Status lifecycle:** `<from Q3>`.
- **Capability ADR section order:** metadata → Context → Capability
  statement → User stories / scenarios → Acceptance criteria → Out of
  scope → Open questions → References → Revision History → Approvals.
<!-- If technology-ADR split (Q2): -->
- **Technology ADR section order:** metadata → Context → Decision →
  Rationale → Consequences → Acceptance criteria → Out of scope → Open
  questions → References → Revision History → Approvals. Rationale must
  name alternatives considered with specific rejection reasons.
- **Acceptance criteria are testable and numbered.**
- **ADRs are internal artefacts — never user-visible.** ADR numbers,
  ADR titles, and the existence of the ADR catalogue must NEVER appear
  in any string the product emits to users: UI copy, API response
  bodies, error messages, customer-visible log lines, public
  documentation, release notes, marketing copy, or support
  communications. The catalogue is a builder's tool, not a user-facing
  surface. References ARE allowed in: code comments
  (`// see adr/0042-foo.md`), commit messages, PR descriptions,
  internal docs, `AGENTS.md`, `CONVENTIONS.md`, `INDEX.md`, and the
  `plan/` queue. Rule of thumb: if a non-builder could ever see the
  string, the ADR reference comes out.
<!-- Insert any Q10 domain-specific hard rules here as additional bullets. -->

## Implementation work

- Start from the ADRs. Identify which ADRs a code change implements or
  affects before changing behaviour.
- If implementation reveals a capability gap or changed decision, update
  the relevant ADR rather than silently diverging.
- Add or update tests for implemented behaviour. Map tests back to ADR
  acceptance criteria where practical.
- **Do not leak ADR identifiers into user-visible surfaces.** When
  writing error messages, UI copy, API responses, log lines that ship
  to customers, public docs, or release notes, refer to the behaviour
  by its product-level name — never by ADR number, ADR title, or
  phrases like "per the ADR catalogue". The ADR link belongs in the
  commit message and (optionally) an inline code comment, not in the
  string the user reads.

## Audit trail and revision discipline

- Substantive ADR changes append a row to the Revision History table.
  Editorial changes (typos, formatting, link fixes) are excluded but
  flagged `editorial` in the commit message.
- Approvals table populates when an ADR is Accepted and updates on each
  later substantive revision.
- Regenerate `INDEX.md` from ADR metadata after any ADR status change
  or new ADR.

## Multi-agent workflow

<!-- Mode 1 — single agent (Q5 default). Keep this block. -->
A single agent owns this repo. The `_agent/` directory tracks live
state and history; LOCKS discipline is not in use.

<!-- Mode 2 — multi-agent, shared checkout. Replace the block above with: -->
<!--
Work is partitioned across named agents (see `_agent/ROLES.md`).
Coordination rules:
- Before editing a file, claim it in `_agent/LOCKS.md` by appending
  `<agent-id> | <path> | <ISO-8601 timestamp>`. Remove the line on
  commit. LOCKS is a filesystem mutex — it prevents simultaneous
  writes to the same file.
- Append a one-line entry to `_agent/WORKLOG.md` on every commit.
- `_agent/CURRENT_FOCUS.md` is the single in-flight snapshot
  (branch, active queue item, blockers, uncommitted work). Update it
  when state changes.
-->

<!-- Mode 3 — multi-agent, separate worktrees / PR branches. Replace
the block above with: -->
<!--
Work is partitioned across named agents (see `_agent/ROLES.md`).
Each agent works in its own git worktree or PR branch.
Coordination rules:
- **GitHub draft PRs / branch assignment are the authoritative lock.**
  `_agent/LOCKS.md` is advisory only — append an intent declaration
  if it helps coordinate before a PR exists; do not rely on it as a
  mutex. The filesystem can't conflict across worktrees, but
  duplicated work and contradictory merges still can.
- Append a one-line entry to `_agent/WORKLOG.md` on every commit.
  The repo's `.gitattributes` sets `_agent/WORKLOG.md merge=union`
  so concurrent appends concatenate instead of conflicting.
- `_agent/CURRENT_FOCUS.md` is local-only per worktree (gitignored).
  Update it freely; it never merges.
- The committed cross-worktree dashboard is `_agent/IN_FLIGHT.md` —
  one row per active worktree (agent, branch, queue item, started).
  Add your row when you start, remove it when the worktree closes.
-->

## Plan folder

- A pending item gets a `plan/todo/NNNN-<slug>.md` file BEFORE work
  starts, naming the owning ADR(s), scope, and exit criteria.
- The completion event is: `<from Q4 — e.g. merge to main, deploy +
  smoke passes, release tag>`. On completion, `git mv` the file to
  `plan/done/<YYYY-MM-DD>-<slug>.md` with a footer naming the HEAD SHA
  and any artefact id.
- The owning ADR(s) advance `Accepted → Implemented` on the same
  commit. Regenerate `INDEX.md`.

## Git contract

- Commit messages follow **Conventional Commits**.
- Mandatory `Rationale:` footer on any commit touching an ADR.
- <Signed commits: yes/no per Q6.>
- <ADR-revision tags `adr-NNNN-rN`: yes/no per Q6.>
- <Co-Authored-By trailer: yes/no per Q6 — default no.>
- Cross-references between ADRs use relative paths (`adr/NNNN-*.md`).

<!-- Integration model per Q4b — keep one block. -->

<!-- Direct-to-main:
- **Integration:** direct-to-main, **fast-forward only**. No merge
  commits on `main`. The verify gate (`<command from Q8>`) runs
  locally and must pass before push. Completion event:
  fast-forwarded to `main` + remote push succeeded.
-->

<!-- PR-based:
- **Integration:** every change ships via a pull request. CI must be
  green before merge — the verify gate (`<command from Q8>`) runs in
  CI on the PR, not (only) locally. Merge strategy:
  <squash | merge | rebase>. Completion event: PR merged to `main`
  with CI green.
-->

