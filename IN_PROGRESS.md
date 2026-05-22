# In Progress

Active tasks for the current development cycle.

Workflow: `ROADMAP.md` → start a task → move here → finish → move to `HISTORY.md`.

When a PR closes a task, the **same PR** must update both `IN_PROGRESS.md` (remove) and `HISTORY.md` (add). Do not defer either to a follow-up commit on the protected branch after merge.

---

### Configure tracking policy for `.claude/settings.json`

The repo currently leaves `.claude/settings.json` untracked. Each Claude Code session accumulates a different set of approved permissions, so committing the file as-is causes recurring merge conflicts, while leaving it ignored loses any shared baseline. Decide and apply a stable policy.

- [ ] Decide whether the file should be tracked, ignored, or split (e.g. tracked baseline + local override).
- [ ] If tracked: prune ad-hoc / per-session entries from the committed copy and document the convention in `CLAUDE.md`.
- [ ] If ignored: add `.claude/settings.json` (or the whole `.claude/`) to `.gitignore` and, optionally, ship a `.claude/settings.example.json` baseline.
- [ ] Make the rule consistent across worktrees so a clean clone does not show the file as a permanent untracked entry.

**Acceptance:** the chosen policy is documented in `CLAUDE.md`, the repo no longer shows `.claude/settings.json` as a permanent untracked file in clean clones, and the rule applies consistently across worktrees.

**Decision (2026-05-22):** ignore the whole `.claude/` directory. Reasons: each Claude Code session accumulates its own permission set so a shared baseline would cause recurring merge conflicts; the directory holds per-user agent state, not project source. No `.claude/settings.example.json` baseline will ship — start fresh per clone.
