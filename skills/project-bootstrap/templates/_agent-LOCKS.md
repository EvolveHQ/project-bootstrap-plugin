# Agent Locks

Append a row before editing a file. Remove the row on commit.

Format: `<agent-id> | <path> | <ISO-8601 timestamp>`

<!-- Mode 3 (worktree) advisory header — the skill leaves this in
place when Q5 selected separate-worktrees mode:

**Advisory only under separate-worktree mode.** GitHub draft PRs and
branch assignment are the authoritative lock; LOCKS rows are an
intent declaration to help coordinate before a PR exists. Do not
rely on this file as a filesystem mutex — the filesystem can't
conflict across worktrees in the first place.
-->

---
