# In Flight

Cross-worktree dashboard. One row per active worktree. Updated by the
agent owning the worktree when it opens, and removed when the worktree
closes (PR merged or abandoned).

Useful when several agents work in separate worktrees / PR branches —
this file replaces the single-checkout `_agent/CURRENT_FOCUS.md` as
the shared "what's happening right now" view. `CURRENT_FOCUS.md` is
gitignored under this mode and stays local to each worktree.

If this file disagrees with reality, reality wins — check
`git worktree list` and the live PRs, then correct here.

| Agent | Worktree branch | Queue item | Started (ISO-8601) | PR link |
|-------|-----------------|------------|---------------------|---------|
