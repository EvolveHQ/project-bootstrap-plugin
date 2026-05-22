# Current Focus

This file is the LIVE SNAPSHOT of any in-flight session. It is short on
purpose — the durable record lives in git (`git log`),
`_agent/WORKLOG.md`, and `plan/done/`. The queued work lives in
`plan/todo/`.

<!-- Mode 3 (worktree) note — the skill adds this file to
`.gitignore` and leaves this comment in place:

Under separate-worktree mode, this file is gitignored and stays local
to each worktree. The cross-worktree dashboard is `_agent/IN_FLIGHT.md`
(committed). Update freely; this file never merges.
-->

If status files and git disagree, git is authoritative; correct this
file.

## Active state

- **Branch:** main
- **Active item:** none — no work in flight.
- **Blockers:** none.
- **Uncommitted work:** none.

## Last shipped

<commit-sha> — <one-line description>.

## Next item

`ls plan/todo/` for the queue; the lowest-numbered file runs next.
