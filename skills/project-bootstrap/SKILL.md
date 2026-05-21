---
name: project-bootstrap
description: Scaffold or retrofit documentation-led conventions (AGENTS.md, CLAUDE.md, CONVENTIONS.md, ADR catalogue, plan/ queue, _agent/ coordination) into a repo. Use when the user asks to "set up conventions", "bootstrap ADRs", "scaffold the documentation-led layout", "add AGENTS.md and a plan queue", or invokes /project-bootstrap. Works on fresh repos and existing ones — preserves existing content and merges rather than overwrites.
---

# project-bootstrap

You are installing (or retrofitting) a documentation-led convention set
in the current repo. The end state is a repo that can be driven by
both humans and coding agents off a small set of canonical files. Carry
over the *mechanism* described here — nothing about any other project.

## Step 1 — Detect the situation

Inspect the repo before asking anything.

- **Fresh repo** (no source, no docs): you are scaffolding from zero.
- **Existing repo**: you are retrofitting.
  - Read any current `README.md`, `CONTRIBUTING.md`, `AGENTS.md`,
    `CLAUDE.md`, `docs/`, `adr/`, `.github/` before proposing changes.
  - Preserve existing content. Merge, don't overwrite. If existing
    conventions conflict with the ones below, surface the conflict in
    your assessment summary.
  - If ADRs already exist in another format, propose a migration plan
    (renumber, keep, translate) rather than creating a parallel tree.

State which situation applies in one line before asking the assessment
questions.

## Step 2 — Target layout

```
<repo>/
  AGENTS.md              # hard rules for coding agents — entry point
  CLAUDE.md              # one-liner: @AGENTS.md
  README.md              # human-facing project summary (preserve if exists)
  CONVENTIONS.md         # authoring rules: ADRs, naming, status, audit, git
  INDEX.md               # generated table of all ADRs
  GLOSSARY.md            # shared terms (optional — see Q7)
  adr/
    0000-template.md     # capability-ADR template (always)
    NNNN-template.md     # technology-ADR template (only if split — see Q2)
    NNNN-<kebab-slug>.md # one ADR per decision, contiguous numbering
  domains/<slug>/README.md   # optional (see Q7)
  plan/
    README.md
    todo/NNNN-<slug>.md
    done/<YYYY-MM-DD>-<slug>.md
  _agent/
    ROLES.md             # named agents and what each owns
    LOCKS.md             # file-claim ledger
    WORKLOG.md           # append-only ship log
    CURRENT_FOCUS.md     # slim live snapshot
    HANDOFF.md           # fresh-agent entry point
    prompts/autonomous.md  # only if a verify gate exists (see Q8)
```

## Step 3 — Conventions to install

1. **ADRs are the source of truth.** One decision per ADR. Splits become
   new ADRs that supersede; never expand scope inside an existing one.
2. **Up to two ADR shapes:**
   - **Capability ADR** (what the system must do). Section order:
     metadata → Context → Capability statement → User stories / scenarios
     → Acceptance criteria → Out of scope → Open questions → References →
     Revision History → Approvals.
   - **Technology ADR** (how it's built). Section order:
     metadata → Context → Decision → Rationale → Consequences →
     Acceptance criteria → Out of scope → Open questions → References →
     Revision History → Approvals. Rationale must name alternatives
     considered and give specific rejection reasons (not "simpler" /
     "more idiomatic").
3. **Status lifecycle:** `Proposed → Accepted → Implemented →
   (Superseded | Deprecated)`. Terminal states reachable from any prior
   state. Status drives plan-folder placement: `Accepted` →
   `plan/todo/`, `Implemented` → `plan/done/`.
4. **Filenames:** `adr/NNNN-kebab-slug.md`, zero-padded 4 digits,
   contiguous, no reserved gaps. Cross-references use relative paths.
5. **Acceptance criteria are testable and numbered.** Tests map back to
   them where practical.
6. **Audit discipline.** Substantive ADR changes append a Revision
   History row. Editorial changes (typos, formatting, link fixes) are
   excluded but flagged `editorial` in the commit message. Approvals
   table populates when an ADR is Accepted and updates on each later
   substantive revision.
7. **INDEX.md is regenerated** from ADR metadata after any ADR change.
   Treat as derived, not hand-edited.
8. **`plan/` is the work queue.** `git mv plan/todo/X plan/done/<date>-X`
   is the completion event; the moved file gets a footer naming the HEAD
   SHA (and deploy artefact id if applicable). Owning ADR(s) advance
   `Accepted → Implemented` on the same commit.
9. **Multi-agent coordination.** Before editing, an agent appends a row
   to `_agent/LOCKS.md` (`<agent-id> | <path> | <ISO-8601 timestamp>`)
   and removes it on commit. On commit, append one line to
   `_agent/WORKLOG.md`. `CURRENT_FOCUS.md` is the live snapshot; if it
   disagrees with git, git wins and `CURRENT_FOCUS.md` is corrected.
10. **Git contract.** Conventional Commits. Mandatory `Rationale:`
    footer on commits touching an ADR. Signed commits unless the user
    opts out. No `Co-Authored-By` trailer for agent work unless the user
    asks for one.
11. **AGENTS.md is the hard-rules entry point;** **CLAUDE.md** is the
    one-liner `@AGENTS.md` so the Claude Code CLI picks it up.
12. **ADRs are internal artefacts — never user-visible.** ADR numbers,
    ADR titles, and the existence of the ADR catalogue must NEVER
    appear in product code paths that reach a user: UI strings, API
    response bodies, error messages, log lines emitted to customers,
    public documentation, release notes, marketing copy, or support
    communications. ADRs are for builders, not users. References ARE
    allowed in: code comments (`// see adr/0042-foo.md`), commit
    messages, PR descriptions, internal docs, `AGENTS.md`,
    `CONVENTIONS.md`, `INDEX.md`, and the `plan/` queue. When in
    doubt, ask whether a non-builder would ever see this string — if
    yes, the ADR reference comes out.

## Step 4 — Assessment (10 questions)

**Ask the questions one at a time, not in a batch.** For each question,
state a **recommended** option (label it "Recommended") with one short
sentence on why; the user picks it, picks an alternative, or types a
custom answer. Wait for the answer before moving to the next question.

If the host CLI exposes a structured single-select question tool (e.g.
Claude Code's `AskUserQuestion`), use it and mark the recommended
option with the literal "(Recommended)" suffix in its label. Otherwise
ask in plain text, listing options as A/B/C and naming the recommended
one.

After all 10 answers are in, summarise the resulting plan in 5–10
lines and ask for sign-off before writing any files.

1. **Project identity.** Name, one-line description, doc language
   (en-GB / en-US / other), and — if existing repo — what current files
   (README, CONTRIBUTING, docs/, adr/, etc.) must be preserved or
   merged. *No recommendation — project-specific.*
2. **ADR shape.** Single shape, or capability-vs-technology split?
   **Recommended: single shape** — start simple, split later if
   long-lived product requirements clearly outlive their
   implementations.
3. **Status lifecycle.** Full `Proposed → Accepted → Implemented →
   (Superseded | Deprecated)`, or shorter (drop `Implemented`)?
   **Recommended: full lifecycle** — the `Implemented` rung is cheap
   and gives a clear "what's shipped" signal.
4. **Plan folder.** Use `plan/todo/` + `plan/done/`, or skip it because
   work is tracked elsewhere? If kept, what event counts as "shipped"?
   **Recommended: use it** — the queue is what makes the convention
   set actionable for agents; default completion event is "merged to
   main + CI green".
5. **Multi-agent setup.** Pick one — the three options have different
   coordination-file shapes, and switching later is not free:
   - **(Recommended) Single agent.** `default-agent` in ROLES.
     LOCKS skipped. WORKLOG / CURRENT_FOCUS as standard single-file
     snapshots. Right for small projects and the "one human + one
     agent" case.
   - **Multi-agent, shared checkout.** Named agents in ROLES. LOCKS
     ON as a filesystem mutex (prevents simultaneous writes to the
     same file). WORKLOG append-on-commit, single file.
     CURRENT_FOCUS as the single in-flight snapshot. Right when
     several agents serialise through one working tree.
   - **Multi-agent, separate worktrees / PR branches.** Named agents
     in ROLES. LOCKS *advisory only* — GitHub draft PRs / branch
     assignment are the real lock; pick one signal, not two.
     `_agent/WORKLOG.md` gets `merge=union` via `.gitattributes` so
     concurrent appends concatenate instead of conflicting (or split
     to `_agent/worklog/<agent-id>.md` if agent set is fixed).
     `_agent/CURRENT_FOCUS.md` becomes local-only (added to
     `.gitignore`); a committed `_agent/IN_FLIGHT.md` dashboard
     aggregates per-worktree state.

   Note: option 2 → option 3 is not a free upgrade later; it means
   splitting WORKLOG (or adding the merge driver) and rethinking
   CURRENT_FOCUS. Choose deliberately.
6. **Git contract.** Confirm or override each — Conventional Commits;
   mandatory `Rationale:` footer on ADR-touching commits; signed
   commits; ADR-revision tags `adr-NNNN-rN`; whether agent commits
   carry a `Co-Authored-By` trailer. **Recommended: Conventional
   Commits ON, `Rationale:` footer ON, signed commits ON, ADR-revision
   tags OFF, `Co-Authored-By` trailer OFF.**
7. **Optional artefacts.** Which now vs. defer: `GLOSSARY.md`,
   `domains/<slug>/README.md` groupings, technology-ADR template?
   **Recommended: defer all three** — add when scale demands
   (terminology drift, >20 ADRs, or technology decisions splitting
   from product decisions).
8. **Verify gate.** What command(s) decide a change is shippable
   (`npm test`, CI workflow, deploy + smoke, manual)? *No
   recommendation — project-specific.* If the user has no real gate,
   the skill will refuse to write `_agent/prompts/autonomous.md`.
9. **Existing-content conflicts** (existing repos only). Any
   conventions already in place (commit format, branch policy, ADR
   style, status names) the new layout must defer to or merge with?
   *No recommendation — project-specific.* Skip this question on a
   fresh repo.
10. **Domain-specific hard rules to bake in.** Any project-specific
    constraints to enforce in `AGENTS.md` / `CONVENTIONS.md` from day
    one — e.g. vendor-naming restriction, regulated-evidence posture
    (attribution, retention, e-signatures), language mandate,
    mandatory user-story personas, separated audit streams?
    **Recommended: none from day one** — add later when a concrete
    requirement appears; pre-emptive hard rules accumulate as cruft.

## Step 5 — Output sequence (after sign-off)

Templates live in this skill's `templates/` directory. Read each
template, fill its placeholders from the assessment answers, then
write it into the repo.

1. `CONVENTIONS.md` — from `templates/CONVENTIONS.md`. Spec other files
   reference.
2. `AGENTS.md` — from `templates/AGENTS.md`.
3. `CLAUDE.md` — from `templates/CLAUDE.md` (single line `@AGENTS.md`).
4. `adr/0000-template.md` — from `templates/adr-capability.md`.
5. `adr/NNNN-template.md` — from `templates/adr-technology.md`, only if
   Q2 said split. `NNNN` is the number where capability ADRs end
   (project-defined, e.g. 0091; default 0100 if unspecified).
6. `plan/README.md` — from `templates/plan-README.md`. Create empty
   `plan/todo/.gitkeep` and `plan/done/.gitkeep`.
7. `_agent/ROLES.md` — from `templates/_agent-ROLES.md`. Mode 1
   keeps the `default-agent` block; modes 2 and 3 expand to named
   agents per Q5.
8. `_agent/LOCKS.md` — from `templates/_agent-LOCKS.md`. **Skip in
   mode 1.** **Mode 3** writes it with an advisory header noting
   PRs are the authoritative lock.
9. `_agent/WORKLOG.md` — from `templates/_agent-WORKLOG.md`. In
   **mode 3**, also write a `.gitattributes` entry:
   `_agent/WORKLOG.md merge=union`.
10. `_agent/CURRENT_FOCUS.md` — from
    `templates/_agent-CURRENT_FOCUS.md`. In **mode 3**, add
    `_agent/CURRENT_FOCUS.md` to `.gitignore` (the file stays
    local-only per worktree) and write
    `_agent/IN_FLIGHT.md` from `templates/_agent-IN_FLIGHT.md` as
    the committed cross-worktree dashboard.
11. `_agent/HANDOFF.md` — from `templates/_agent-HANDOFF.md`.
12. Stub `INDEX.md` — header + empty table.
13. `_agent/prompts/autonomous.md` — from
    `templates/_agent-prompts-autonomous.md`, **only** if Q8 confirmed a
    verify gate.

Commit each file (or logical group) with a Conventional Commit message;
no `Co-Authored-By` trailer unless Q6 asked for one.

For an existing repo, prefer Edit over Write where files exist, and
call out every merge decision in the commit message.

## Step 6 — Offer backfill (existing repos only)

Once the scaffolding commit has landed, **offer to backfill** the ADR
catalogue, the plan folder, and `CONVENTIONS.md` from the existing
code and git history. Skip this step entirely on a fresh repo.

Phrase the offer like this:

> The scaffolding is in. Want me to backfill ADRs, plan/done entries,
> and CONVENTIONS additions from the existing code and commit history?
> I'll propose drafts; you approve in batches before anything lands.

If the user accepts, run the backfill in four passes. Each pass
produces drafts the user reviews before they are committed.

1. **Scan inputs (read-only).**
   - `git log --oneline --reverse main` (or the default branch) — the
     decision and shipped-work trail.
   - Major modules / packages / top-level source directories — the
     surface of what exists.
   - Any existing docs (`README`, `docs/`, design notes, RFCs) — prior
     decision records, even informal ones.
   - `package.json` / `pyproject.toml` / `go.mod` etc. — declared
     dependencies often map to technology decisions.

2. **Propose ADRs.** For each distinguishable decision or capability
   evident from the scan, draft an ADR using the appropriate template.
   - Capabilities (what the system does) → capability ADR, status
     `Implemented` (the code already exists), Revision History row
     citing the commit(s) that introduced the behaviour.
   - Technology choices (framework, persistence, deployment target) →
     technology ADR, status `Accepted` or `Implemented` as appropriate,
     Rationale section reconstructed from commit messages and code
     comments — flag any speculative rationale clearly so the human
     can correct it.
   - Number contiguously from `0001`. Show the user the proposed list
     (number + title + status + one-line scope) before writing files.

3. **Propose plan/done entries.** For each ADR drafted as
   `Implemented`, generate a corresponding `plan/done/<date>-<slug>.md`
   file using the commit date of the implementing commit (or the
   merge commit) as the date prefix. Body: short summary, owning ADR,
   "Shipped at HEAD `<sha>`" footer. Group these into a single
   approval prompt — they are mechanical once the ADRs are agreed.

4. **Propose CONVENTIONS additions.** Identify patterns in the existing
   repo that should be promoted to written conventions: commit-message
   style, branch naming, test layout, file-naming rules, language /
   tooling choices that are de-facto standards. Draft additions to
   `CONVENTIONS.md` (and corresponding bullets in `AGENTS.md` §Hard
   rules if they should be enforced). Show the diff before applying.

After each pass, commit the approved drafts with a Conventional Commit
(`docs(adr): backfill ADRs 0001-00NN from code and history`,
`docs(plan): backfill plan/done from shipped commits`,
`docs: backfill conventions from de-facto patterns`). Regenerate
`INDEX.md` after the ADR pass.

**Important guardrails.**
- The backfill produces *drafts*. Every commit is reviewable; the user
  is the authority on whether an inferred decision is real.
- If commit history is sparse or unclear, say so and stop — do not
  invent rationale to fill gaps.
- If the user declines the backfill, the scaffolding is still
  complete; the queue is just empty until the first hand-authored ADR
  lands.
