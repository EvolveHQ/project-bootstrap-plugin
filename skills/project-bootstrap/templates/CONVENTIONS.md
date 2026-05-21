# Conventions

## Project

Project name: <name>.

<!-- Q1 language: if a language mandate is set, state it here.
Example: "Language: en-GB throughout. Use forms such as organisation,
behaviour, prioritise, catalogue, authorisation consistently across all
files." -->

## ADR Files

ADR filenames use `NNNN-kebab-case-slug.md`, zero-padded to 4 digits,
with contiguous numbering and no reserved gaps.

Each ADR describes one decision. If a decision splits, supersede the
original ADR and create new ADRs rather than expanding scope inside a
single document.

Status lifecycle: `<from Q3>`.

| Status | Meaning |
|---|---|
| Proposed | Draft. Decision authored but not yet approved. |
| Accepted | Decision approved; implementation authorised. Work item lives in `plan/todo/`. |
| Implemented | Code shipped per Q4 completion event. Work item moved to `plan/done/`. ADR is the authoritative spec the running system matches. |
| Superseded | Replaced by another ADR. The successor is named in `superseded-by:` metadata. |
| Deprecated | Was real; the world moved on; no successor. Capability is not being rebuilt. |

Terminal states (Superseded / Deprecated) are reachable from any prior
state.

Cross-references link by relative path to `adr/NNNN-*.md`.

## ADR Shapes

<!-- Single shape (Q2): -->
This project uses a single ADR shape. ADRs use `adr/0000-template.md`
and contain these sections in order: Context, Capability statement,
User stories / scenarios, Acceptance criteria, Out of scope, Open
questions, References, Revision History, Approvals.

<!-- Split (Q2): -->
<!--
Capability ADRs (`0001`-`NNNN`) describe what the system must do. They
use `adr/0000-template.md` with sections: Context, Capability
statement, User stories / scenarios, Acceptance criteria, Out of
scope, Open questions, References, Revision History, Approvals.

Technology ADRs (`NNNN+1` onwards) describe how the system is built.
They use `adr/NNNN-template.md` with sections: Context, Decision,
Rationale, Consequences, Acceptance criteria, Out of scope, Open
questions, References, Revision History, Approvals.

Technology ADR Rationale must name alternatives considered and give
specific reasons they were rejected. Generic rationale such as
"simpler" or "more idiomatic" is not sufficient.
-->

<!-- Q10 domain-specific rules go here, each as its own section.
Examples:
## Vendor Name Rule
## Audit-Plane Distinction
## Regulated Evidence
## Mandatory user-story personas
-->

## ADR Privacy

ADRs are internal artefacts. ADR numbers, ADR titles, and the
existence of the ADR catalogue must never appear in any string the
product emits to users: UI copy, API response bodies, error messages,
customer-visible log lines, public documentation, release notes,
marketing copy, or support communications.

Allowed references:
- Inline code comments tying a non-obvious choice to its ADR
  (`// see adr/0042-foo.md`).
- Commit messages and PR descriptions.
- Internal documents: `AGENTS.md`, `INDEX.md`, the `plan/` queue,
  `_agent/` files, internal runbooks.

Rule of thumb: if a non-builder could ever read the string, the ADR
reference comes out. Refer to the behaviour by its product-level name
instead.

## Multi-Agent Rules

<!-- Single agent: drop this section, or leave a one-liner. -->
<!-- Multi-agent:
Claim a file in `_agent/LOCKS.md` before editing. Append to
`_agent/WORKLOG.md` on commit. Regenerate `INDEX.md` after any ADR
status change or new ADR.
-->

## Plan Folder

Pending and shipped work live in `plan/` at the repository root:

- `plan/todo/NNNN-<slug>.md` — pending work, lower numbers run first.
  Each file names the owning ADR(s), scope, and exit criteria.
- `plan/done/<YYYY-MM-DD>-<slug>.md` — shipped work, chronological. A
  `git mv` from `todo/` to `done/` is the completion event.

The completion event is: `<from Q4>`.

When a `plan/todo/` item ships, the file moves to `plan/done/` AND the
owning ADR(s)' `status:` advances from `Accepted` to `Implemented`.
`INDEX.md` is regenerated to match.

## Audit Trail Policy

Every substantive change to an Accepted ADR appends a new row to its
Revision History table.

Editorial changes — typos, formatting, link fixes — are excluded from
Revision History but must be flagged "editorial" in the commit message.

The Approvals table is populated when an ADR transitions to Accepted
and updated on every subsequent substantive revision.

## Git Contract

Commit messages follow Conventional Commits with a mandatory
`Rationale:` footer for any commit that touches an ADR.

<!-- Per Q6:
- Signed commits: <yes/no>
- ADR-revision tags `adr-NNNN-rN`: <yes/no>
- Co-Authored-By trailer on agent commits: <yes/no>
-->
