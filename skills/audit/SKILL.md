---
name: audit
description: Audit a documentation-led repo against its own conventions — contiguous ADR numbering, INDEX sync, plan/ coverage, required sections, status validity, cross-reference resolution, language mandate, and ADR-privacy leaks into user-visible code. Reports a punch list and offers to fix the mechanical issues. Use when the user says "audit the ADRs", "lint the conventions", "check repo consistency", "are the ADRs in sync", or invokes /audit.
---

# audit

Check a documentation-led repo against the conventions it declares.
This is the enforcement `AGENTS.md` cannot guarantee on its own.

## Step 0 — Preconditions and context

1. Confirm the repo is bootstrapped.
2. Read `CONVENTIONS.md` to learn what to enforce: ADR shape and
   cutoff, status lifecycle, integration model, multi-agent mode,
   language mandate, optional artefacts present (GLOSSARY, domains/),
   and any Q10 domain hard rules.

## Step 1 — Run the checks (read-only)

Report each as PASS / FAIL / N/A with specifics (file + line where
relevant):

1. **Numbering.** ADR filenames contiguous, zero-padded, no gaps, no
   duplicates. Split repos: capability below cutoff, technology at/above.
2. **INDEX sync.** Every ADR appears in `INDEX.md`; every INDEX row has
   a matching file; metadata fields (status, title, date) agree.
3. **Plan coverage.** Every `Accepted` ADR has a `plan/todo/` item;
   every `Implemented` ADR has a `plan/done/` entry. Flag orphans both
   ways.
4. **Section completeness.** Each ADR has the required sections in the
   order its shape mandates. Acceptance criteria are numbered.
5. **Status validity.** Every `status:` is in the declared lifecycle.
   `Superseded` ADRs name a successor in `superseded-by:`; the successor
   names them in `supersedes:` (symmetry).
6. **Revision/Approvals.** Revision History present; Approvals populated
   for ADRs at `Accepted` or beyond.
7. **Cross-references.** Relative `adr/NNNN-*.md` links resolve to real
   files. Glossary anchors (if used) resolve.
8. **Language mandate.** If set, spot-check user-facing docs for the
   required spellings.
9. **ADR-privacy leaks.** Grep source / product directories for ADR
   identifiers in user-visible strings — patterns like `ADR 0042`,
   `adr-0042`, `see ADR`, ADR titles — in UI copy, API responses,
   error messages, customer-facing logs, public docs, release notes.
   Report each suspect; this rule is easy to violate by reflex.
10. **Coordination hygiene.** `_agent/LOCKS.md` has no stale claims
    (mode 2); `_agent/IN_FLIGHT.md` rows match live worktrees (mode 3).

## Step 2 — Report

Lead with a one-line verdict (clean / N issues). Then the punch list,
grouped by severity: **blocking** (privacy leaks, status/lifecycle
violations, broken cross-refs), **drift** (INDEX out of sync, missing
plan files), **hygiene** (stale locks, formatting).

## Step 3 — Offer fixes

Offer to fix the **mechanical** issues automatically: regenerate
`INDEX.md`, create missing `plan/todo` stubs, clear stale locks, fix
broken relative links. **Do not** auto-edit ADR content, rewrite
acceptance criteria, or remove suspected privacy leaks without the
user confirming each — those need judgement. Commit fixes as
`fix(adr): ...` / `docs: ...` with a `Rationale:` footer where an ADR
is touched.
