# docflow

A Claude Code plugin for **ADR-driven, documentation-led projects**. It
installs a `bootstrap` skill that scaffolds (or retrofits) an
**Architecture Decision Record (ADR)** catalogue, a plan queue, and
`AGENTS.md` conventions into any repository, plus a set of **lifecycle
skills** that author, queue, ship, and audit ADRs — so the project can
be driven by both humans and coding agents from a small set of canonical
files.

## Skills

| Skill | Slash command | Purpose |
|-------|---------------|---------|
| bootstrap | `/bootstrap` | Scaffold or retrofit the whole convention set. Start here. |
| new-adr | `/new-adr` | Author one ADR — next contiguous number, right shape, INDEX + domain wiring, supersede linkage. |
| new-plan | `/new-plan` | Add a `plan/todo` item tracing to its owning ADR(s). |
| ship-item | `/ship-item` | Run the completion event: verify → integrate → `todo`→`done` → ADR `Accepted`→`Implemented` → INDEX/WORKLOG. |
| add-convention | `/add-convention` | Assess whether a convention is worth codifying, route it to the right home (or to an ADR), then add it. |
| audit | `/audit` | Lint the repo against its own conventions — numbering, INDEX sync, plan coverage, **ADR-privacy leaks**, more. |
| brainstorm | `/brainstorm` | Decompose a problem into candidate ADRs + plan items (proposes drafts; writes nothing until approved). |
| agent-wave | `/agent-wave` | Orchestrate a wave of parallel worktree subagents over the queue, with checkpoint or continuous supervision. |

The lifecycle skills all **read `CONVENTIONS.md` first** and honour the
choices the bootstrap recorded (ADR shape, status lifecycle, integration
model, multi-agent mode). They refuse to run on an un-bootstrapped repo
and point you at `/bootstrap`.

## What `/bootstrap` installs

- `AGENTS.md` — hard rules for coding agents (the entry point).
- `CLAUDE.md` — one-liner re-exporting `AGENTS.md` so Claude Code picks
  it up automatically.
- `CONVENTIONS.md` — authoring rules for ADRs, naming, status
  lifecycle, audit trail, and git contract.
- `INDEX.md` — generated table of all ADRs.
- `adr/` — ADR catalogue with a capability-ADR template (and an
  optional technology-ADR template).
- `plan/todo/` and `plan/done/` — implementation queue. `git mv` from
  `todo/` to `done/` is the completion event.
- `_agent/` — multi-agent coordination: `ROLES.md`, `LOCKS.md`,
  `WORKLOG.md`, `CURRENT_FOCUS.md`, `HANDOFF.md`, and an optional
  unsupervised-run prompt under `prompts/`.

Optional, off by default: `GLOSSARY.md`, `domains/<slug>/README.md`
groupings, project-specific hard rules (vendor-naming restriction,
regulated-evidence posture, language mandate, audit-stream separation).

## Why

Documentation-led projects rot when conventions live in someone's head.
This plugin makes the conventions explicit, machine-readable, and
applied uniformly — so a fresh contributor (human or agent) can pick up
the repo with no oral handover.

It works equally well on **fresh repos** (scaffolds from zero) and on
**existing repos** (retrofits, preserving and merging existing files
rather than overwriting them).

## Install

### From this marketplace

```
/plugin marketplace add EvolveHQ/docflow
/plugin install docflow@evolvehq
```

### Local development (no install)

```
claude --plugin-dir <path-to-this-repo>
```

### Direct skill clone (no plugin lifecycle)

```
git clone https://github.com/EvolveHQ/docflow ~/.claude/skills/docflow-src
ln -s ~/.claude/skills/docflow-src/skills/bootstrap ~/.claude/skills/bootstrap
```

On Windows, copy `skills/bootstrap/` (and the other `skills/*` dirs you
want) into `%USERPROFILE%\.claude\skills\` instead of symlinking.

## Quick start

In any repo, run:

```
/bootstrap
```

or just say *"set up documentation-led conventions in this repo"*,
*"bootstrap ADRs and a plan queue"*, or *"scaffold AGENTS.md and the
_agent/ layout"*. The skill auto-triggers on those phrasings.

The skill will:

1. Detect whether the repo is fresh or existing, and state which.
2. Ask 10 assessment questions to tune the conventions to the project
   — one at a time, with a recommended option for each.
3. Summarise the resulting plan and ask for sign-off.
4. Write (or Edit, for existing repos) the files.
5. Commit each logical group with a Conventional Commit message.
6. **On existing repos**, offer to backfill ADRs, `plan/done/`, and
   `CONVENTIONS.md` additions from the existing code and git history
   — drafts only, approved in batches before anything commits.

## Updating

Recipients refresh installations with:

```
/plugin marketplace update evolvehq
/plugin install docflow@evolvehq
```

See [USAGE.md §Updating the plugin](USAGE.md#8-updating-the-plugin)
for the author-side flow (version bumps, release tags) and recipient
options including `/reload-plugins` for live sessions.

## Full usage and customisation guide

See [USAGE.md](USAGE.md) for the assessment questions, what each
answer changes, the file-by-file output, the backfill flow, and how
to extend or override the templates.

## Layout

```
docflow/
  .claude-plugin/
    plugin.json          # plugin manifest
    marketplace.json     # marketplace listing (this repo is its own marketplace)
  skills/
    bootstrap/
      SKILL.md           # bootstrap: assessment + output sequence + backfill
      templates/         # files the bootstrap reads and writes into target repos
    new-adr/SKILL.md     # lifecycle skills — operate on a bootstrapped repo,
    new-plan/SKILL.md    #   read CONVENTIONS.md, honour its choices
    ship-item/SKILL.md
    add-convention/SKILL.md
    audit/SKILL.md
    brainstorm/SKILL.md
    agent-wave/SKILL.md
  README.md
  USAGE.md
```

Only the `bootstrap` skill uses `skills/bootstrap/templates/`. The
lifecycle skills act on the copies the bootstrap wrote into the target
repo (e.g. its `adr/0000-template.md`), so they carry no templates of
their own.

## License

MIT. Use it, fork it, change it. If you improve a template, a PR is
welcome.
