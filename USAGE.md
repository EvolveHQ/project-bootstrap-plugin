# Using project-bootstrap

This document explains how the skill behaves end-to-end, what each of
the 10 assessment questions actually changes in the output, and how to
customise or extend the templates.

For installation, see the [README](README.md).

## 1. Triggering the skill

Once the plugin is installed (or `--plugin-dir`'d) into a Claude Code
session, the skill is available in any repo. Three ways to trigger:

1. **Slash command:** `/project-bootstrap`.
2. **Natural language** matching the skill's description, e.g.
   - "set up documentation-led conventions in this repo"
   - "bootstrap ADRs and a plan queue"
   - "scaffold AGENTS.md and the _agent/ layout"
   - "retrofit this existing repo with the documentation conventions"
3. **From another agent's prompt** — name the skill explicitly:
   *"invoke the `project-bootstrap` skill on this repository."*

## 2. The flow

The skill runs in three phases.

### Phase A — Detect

The skill inspects the repo and states in one line whether it is:

- **Fresh** — no source, no docs. It will scaffold from zero.
- **Existing** — it reads any current `README.md`, `CONTRIBUTING.md`,
  `AGENTS.md`, `CLAUDE.md`, `docs/`, `adr/`, `.github/` first.
  Existing content is preserved; merges, not overwrites.

If existing ADRs use a different format, the skill proposes a
migration plan (renumber, keep, translate) instead of creating a
parallel tree.

### Phase B — Assess

The skill asks the 10 assessment questions **one at a time**, in order.
For each question it states a **recommended** option with one short
sentence on why, plus the alternatives. You pick the recommended
option, pick an alternative, or type a custom answer. After all 10
answers are in, the skill summarises the resulting plan in 5–10 lines
and asks for sign-off. **It does not write files before sign-off.**

If the host CLI supports structured single-select (e.g. Claude Code's
`AskUserQuestion`), the recommended option is labelled "(Recommended)"
in the picker; otherwise the recommendation is named in plain text.

### Phase C — Write

After sign-off, the skill writes files in dependency order
(`CONVENTIONS.md` first, then `AGENTS.md` + `CLAUDE.md`, then ADR
templates, then `plan/`, then `_agent/`, then `INDEX.md` stub,
optionally `_agent/prompts/autonomous.md`). For existing repos it
prefers `Edit` over `Write` where files already exist, and calls out
every merge decision in the commit message.

### Phase D — Backfill (existing repos only)

Once the scaffolding has landed, the skill **offers** to backfill
ADRs, `plan/done/` entries, and `CONVENTIONS.md` additions from the
existing code and git history. Skipped on fresh repos.

If accepted, the backfill runs in four passes, each producing drafts
you review before they commit:

1. **Scan** (read-only): `git log`, top-level source dirs, existing
   docs, dependency manifests.
2. **Propose ADRs.** One per distinguishable decision or capability.
   Capabilities go in as capability ADRs with status `Implemented`
   (code already exists), Revision History citing the commits that
   introduced the behaviour. Technology choices go in as technology
   ADRs with reconstructed Rationale — anything speculative is flagged
   so you can correct it. You see the proposed list (number + title +
   status + scope) before any file is written.
3. **Propose `plan/done/` entries.** One per ADR drafted as
   `Implemented`, dated by the implementing commit's date. Approved as
   a batch.
4. **Propose `CONVENTIONS.md` additions.** De-facto patterns in the
   repo (commit-message style, branch naming, test layout, file-naming
   rules, tooling defaults) promoted to written rules. Diff shown
   before applying.

Each accepted pass commits with a Conventional Commit:
`docs(adr): backfill ADRs 0001-00NN from code and history`,
`docs(plan): backfill plan/done from shipped commits`,
`docs: backfill conventions from de-facto patterns`. `INDEX.md` is
regenerated after the ADR pass.

**Guardrails.** Everything is a draft; you approve each pass. If
history is sparse or unclear, the skill stops rather than invent
rationale. Declining the backfill is fine — the scaffolding is
already complete and the queue is just empty until the first
hand-authored ADR.

## 3. The 10 assessment questions

Each question's answer changes specific files. Knowing the
recommendation and what it changes up front helps you answer quickly.

| # | Question | Recommended | What it changes |
|---|----------|-------------|-----------------|
| 1 | **Project identity** — name, one-liner, doc language, existing files to preserve. | *No default — project-specific.* | Header of `AGENTS.md`, `CONVENTIONS.md`, `README.md`; language mandate (if any) added to `CONVENTIONS.md` §Project. |
| 2 | **ADR shape** — single vs. capability-vs-technology split. | **Single shape.** Start simple; split later if long-lived product requirements clearly outlive their implementations. | Whether `adr/NNNN-template.md` (technology) is created in addition to `adr/0000-template.md`. Section-order rules in `CONVENTIONS.md` §ADR Shapes and `AGENTS.md` §Hard rules. |
| 3 | **Status lifecycle** — full or shorter. | **Full lifecycle.** The `Implemented` rung is cheap and gives a clear "what's shipped" signal. | Status table in `CONVENTIONS.md` §ADR Files and `plan/README.md` §Status semantics. Whether the `Implemented` rung exists at all. |
| 4 | **Plan folder + integration model.** Q4a: use `plan/todo/` + `plan/done/` or skip. Q4b: how work reaches `main` — **direct-to-main (fast-forward)** or **PR-based (required CI green)**. Asked after Q5, because the Q4b recommendation depends on the multi-agent mode. | **Q4a: use it.** **Q4b: direct-to-main if Q5 = single agent; PR-based if Q5 = multi-agent.** | Whether `plan/` is created. The completion-event sentence in `CONVENTIONS.md` §Plan Folder, `plan/README.md`, `AGENTS.md` §Plan folder. The integration block in `AGENTS.md` / `CONVENTIONS.md` §Git contract. Which variant of `_agent/prompts/autonomous.md` Step 6 is kept (`git merge --ff-only` vs. `gh pr create` → CI → merge). |
| 5 | **Multi-agent setup** — pick one of three: (1) single agent; (2) multi-agent, shared checkout; (3) multi-agent, separate worktrees / PR branches. The three options differ in LOCKS semantics, WORKLOG layout, and CURRENT_FOCUS handling — switching later is not free. | **Single agent.** Small projects shouldn't pay the coordination cost. | `_agent/ROLES.md` content. `AGENTS.md` §Multi-agent workflow and `CONVENTIONS.md` §Multi-Agent Rules pick one of three variants. Mode 2 turns LOCKS on as a filesystem mutex. Mode 3 sets LOCKS to advisory, adds `_agent/WORKLOG.md merge=union` to `.gitattributes`, gitignores `_agent/CURRENT_FOCUS.md`, and writes `_agent/IN_FLIGHT.md` as the committed cross-worktree dashboard. |
| 6 | **Git contract** — Conventional Commits, `Rationale:` footer, signed commits, ADR-revision tags, `Co-Authored-By` trailer. | **Conventional Commits ON, `Rationale:` footer ON, signed commits ON, ADR-revision tags OFF, `Co-Authored-By` trailer OFF.** | `AGENTS.md` §Git contract and `CONVENTIONS.md` §Git Contract bullets. |
| 7 | **Optional artefacts** — `GLOSSARY.md`, `domains/`, technology-ADR template. | **Defer all three.** Add when scale demands (terminology drift, >20 ADRs, technology decisions splitting from product). | Whether those files / directories are created. |
| 8 | **Verify gate** — what command(s) decide a change is shippable. | *No default — project-specific.* | Whether `_agent/prompts/autonomous.md` is written. The skill **refuses** to write the autonomous prompt without a real verify gate. The Step 4 line of the prompt is filled with your command. |
| 9 | **Existing-content conflicts** (existing repos only) — current commit format, branch policy, ADR style, status names the new layout must respect. | *No default — project-specific.* Skipped on fresh repos. | Merge behaviour during writes; commit-message notes calling out each compromise. |
| 10 | **Domain-specific hard rules** to enforce from day one — e.g. vendor-naming restriction, regulated-evidence posture (attribution, retention, e-signatures), language mandate, mandatory user-story personas, separated audit streams. | **None from day one.** Add later when a concrete requirement appears; pre-emptive hard rules accumulate as cruft. | Additional sections in `CONVENTIONS.md` and additional bullets in `AGENTS.md` §Hard rules. |

## 4. Output files

After sign-off, the skill writes:

| File | Source template | Notes |
|------|-----------------|-------|
| `CONVENTIONS.md` | `templates/CONVENTIONS.md` | Spec the other files reference. Written first. |
| `AGENTS.md` | `templates/AGENTS.md` | Hard rules entry point. |
| `CLAUDE.md` | `templates/CLAUDE.md` | Single line `@AGENTS.md`. |
| `adr/0000-template.md` | `templates/adr-capability.md` | Capability-ADR template. |
| `adr/NNNN-template.md` | `templates/adr-technology.md` | Technology-ADR template. Only if Q2 said split. `NNNN` is the project-defined cutoff (default 0100). |
| `plan/README.md` | `templates/plan-README.md` | Plus empty `plan/todo/.gitkeep` and `plan/done/.gitkeep`. |
| `_agent/ROLES.md` | `templates/_agent-ROLES.md` | Single `default-agent` by default. |
| `_agent/LOCKS.md` | `templates/_agent-LOCKS.md` | Empty header. Created only if Q5 enabled LOCKS discipline. |
| `_agent/WORKLOG.md` | `templates/_agent-WORKLOG.md` | Empty table header. |
| `_agent/CURRENT_FOCUS.md` | `templates/_agent-CURRENT_FOCUS.md` | Live snapshot, initial state "none". |
| `_agent/HANDOFF.md` | `templates/_agent-HANDOFF.md` | Fresh-agent entry point. |
| `_agent/IN_FLIGHT.md` | `templates/_agent-IN_FLIGHT.md` | Only in Q5 mode 3 (worktree). Committed cross-worktree dashboard. |
| `.gitattributes` (entry) | (none — inline) | Only in Q5 mode 3: appends `_agent/WORKLOG.md merge=union`. |
| `.gitignore` (entry) | (none — inline) | Only in Q5 mode 3: adds `_agent/CURRENT_FOCUS.md` so it stays local-only. |
| `INDEX.md` | (none — generated) | Stub header + empty table. |
| `_agent/prompts/autonomous.md` | `templates/_agent-prompts-autonomous.md` | Only if Q8 confirmed a verify gate. |

## 5. After scaffolding

The repo is now ready for ADR-driven work. Typical first steps:

1. **Author the first ADR.** Copy `adr/0000-template.md` to
   `adr/0001-<slug>.md`. Fill in Context, Capability statement, User
   stories, Acceptance criteria. Set status to `Proposed`.
2. **Get it Accepted.** Update Approvals table, change status to
   `Accepted`, regenerate `INDEX.md`.
3. **Queue the work.** Create `plan/todo/0001-<slug>.md` naming the
   owning ADR, scope, exit criteria. Lower-numbered files run first.
4. **Implement.** Code against the ADR's numbered acceptance criteria.
   Add tests that map back to those criteria.
5. **Integrate.** Per the Q4b integration model: fast-forward to
   `main` and push (direct-to-main), or open a PR, wait for CI green,
   and merge (PR-based).
6. **Ship.** Once on `main`:
   `git mv plan/todo/0001-<slug>.md plan/done/<YYYY-MM-DD>-<slug>.md`,
   amend the file with a "Shipped at HEAD `<sha>`" footer, advance the
   ADR's `status:` from `Accepted` to `Implemented`, regenerate
   `INDEX.md`, append a row to `_agent/WORKLOG.md`.

## 5a. Lifecycle skills

The manual workflow in §5 is exactly what the lifecycle skills
automate. They all share three properties: they **read `CONVENTIONS.md`
first** and honour the repo's recorded choices (ADR shape, status
lifecycle, integration model, multi-agent mode); they **refuse on an
un-bootstrapped repo** and point at `/project-bootstrap`; and they keep
`INDEX.md` and the `_agent/` coordination files in sync as a side
effect.

| Skill | Replaces these manual steps |
|-------|------------------------------|
| `/new-adr` | §5 steps 1–2: pick number, choose shape, fill template, set `Proposed`, regen INDEX, wire domain README, supersede linkage. Offers to walk to `Accepted` and to create the plan item. |
| `/new-plan` | §5 step 3: create `plan/todo/NNNN`, name owning ADR(s), scope, exit criteria, dependencies, queue position. |
| `/ship-item` | §5 steps 5–6: verify gate → integrate (ff or PR per Q4b) → `git mv` todo→done with footer → ADR `Accepted`→`Implemented` → regen INDEX → WORKLOG → live snapshot. The most order-sensitive operation; let the skill do it. |
| `/add-convention` | Assesses whether a convention is worth codifying (triages out one-offs, duplicates, churn-prone, vague), routes it to AGENTS.md / CONVENTIONS.md / GLOSSARY / an ADR, then writes it. |
| `/audit` | Lints the repo against its own conventions and reports a punch list — numbering, INDEX sync, plan coverage, section completeness, status validity, cross-refs, language mandate, and **ADR-privacy leaks into user-visible code**. Offers to fix the mechanical issues. |
| `/brainstorm` | Decomposes a problem into candidate ADRs + plan items with dependency edges and ordering. Proposes drafts only; writes nothing until approved, then hands off to `/new-adr` and `/new-plan`. |
| `/agent-wave` | Orchestrates parallel worktree subagents over the queue. Asks wave width, budget (items/waves; hours as a soft cap), and supervision (checkpoint vs. continuous). Requires multi-agent mode; refuses mode 1. |

### agent-wave: what it can and can't do

In-session subagents are bounded by the session, not by wall-clock
hours. `/agent-wave` therefore measures budget in **items and waves**,
with hours as a *soft cap*. For a genuinely long-running, session-
outliving fleet, schedule `_agent/prompts/autonomous.md` via
`/schedule` (or use remote agents) instead — `/agent-wave` points you
there rather than over-promising.

It requires a multi-agent mode (Q5 mode 2 or 3) and works best in mode
3 (separate worktrees), where each subagent gets an isolated checkout.
Continuous-unsupervised runs are recommended only with PR-based
integration, so CI gates each merge.

## 6. Customising or extending

The templates are deliberately small and self-contained. To customise:

- **Edit a template.** Files in `skills/project-bootstrap/templates/`
  are read verbatim by the skill, then placeholders are filled from
  the assessment answers. Change the template, the next bootstrap
  reflects it.
- **Add a new template.** Put it in `templates/` and reference it from
  `SKILL.md` §Step 5 (Output sequence).
- **Tighten a skill's auto-trigger.** Edit the `description:`
  frontmatter in that skill's `SKILL.md`. Skills match user requests
  against this string — be specific about phrasings, be terse about
  scope. With several skills in one plugin, watch for trigger collision
  (e.g. "add a convention" vs. "add an ADR") and keep descriptions
  distinct; the slash commands are the unambiguous entry point.
- **Add a new lifecycle skill.** Create `skills/<name>/SKILL.md`. Follow
  the shared pattern: a Step 0 that checks the repo is bootstrapped and
  reads `CONVENTIONS.md`, then act in a way that honours the recorded
  choices. Reuse the templates under
  `skills/project-bootstrap/templates/` rather than duplicating them.
- **Fork the plugin.** This repo is small enough to fork and host
  internally if you want a private convention set.

## 7. Re-running

The skill is idempotent in spirit, but be careful: re-running on a
repo that already has scaffolded files will trigger the "existing
repo" path and ask Q9 about conflicts. Use it intentionally — to add
the parts you skipped the first time, not to wipe and rebuild.

## 8. Updating the plugin

### As a recipient (someone who installed the plugin)

When the plugin author pushes changes, refresh your installation:

```
/plugin marketplace update evolvehq
/plugin install project-bootstrap@evolvehq
```

The first command pulls the latest marketplace listing (a `git fetch`
behind the scenes against the marketplace repo). The second reinstalls
the plugin at its now-current version. To pick up the changes in an
*already-running* Claude Code session, run `/reload-plugins` instead
of restarting.

To check what's installed and the resolved version:

```
/plugin list
```

To remove the plugin entirely:

```
/plugin uninstall project-bootstrap@evolvehq
```

### As the author (you, maintaining this repo)

1. Edit the skill body (`skills/project-bootstrap/SKILL.md`), templates
   (`skills/project-bootstrap/templates/*.md`), or supporting docs.
2. Bump the `version` field in `.claude-plugin/plugin.json` following
   semver — patch for fixes, minor for new questions / templates,
   major for breaking changes to the skill flow.
3. Update this `USAGE.md` and `README.md` if behaviour changed.
4. Commit with a Conventional Commit message
   (`feat(skill): ...`, `fix(template): ...`, `docs: ...`).
5. Push to `origin/main`. Recipients refresh per the section above.

If you tag releases, use `git tag vX.Y.Z` matching `plugin.json`.
Tags help recipients pin to a specific version via
`/plugin install project-bootstrap@evolvehq@vX.Y.Z` (when supported by
their Claude Code version).

## 9. Troubleshooting

- **Skill not appearing in the available-skills list.** Confirm the
  plugin is installed (`/plugin list`) or `--plugin-dir` was passed.
  Restart the Claude Code session.
- **Slash command not found.** The slash form is `/project-bootstrap`
  exactly (hyphenated, lowercase).
- **Skill asks questions the project already answered.** That's by
  design — the skill does not persist state between invocations.
  Answer briefly; the second run is faster.
- **Existing repo merge looks wrong.** Stop and surface it. The skill
  is instructed to preserve existing content; if it overwrote
  something, that's a bug — file an issue with the merge diff.
