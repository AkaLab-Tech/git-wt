# Roadmap

Backlog of work for this project. Tasks flow: `ROADMAP.md` → `IN_PROGRESS.md` → `HISTORY.md`.

Each task lives here as a heading with whatever description it needs (acceptance criteria, design notes, sub-tasks). When work starts, move the block to `IN_PROGRESS.md`.

---

## High Priority

## Medium Priority

## Low Priority / Ideas

### Configure tracking policy for `.claude/settings.json`

The repo currently leaves `.claude/settings.json` untracked. Each Claude Code session accumulates a different set of approved permissions, so committing the file as-is causes recurring merge conflicts, while leaving it ignored loses any shared baseline. Decide and apply a stable policy.

- [ ] Decide whether the file should be tracked, ignored, or split (e.g. tracked baseline + local override).
- [ ] If tracked: prune ad-hoc / per-session entries from the committed copy and document the convention in `CLAUDE.md`.
- [ ] If ignored: add `.claude/settings.json` (or the whole `.claude/`) to `.gitignore` and, optionally, ship a `.claude/settings.example.json` baseline.
- [ ] Make the rule consistent across worktrees so a clean clone does not show the file as a permanent untracked entry.

**Acceptance:** the chosen policy is documented in `CLAUDE.md`, the repo no longer shows `.claude/settings.json` as a permanent untracked file in clean clones, and the rule applies consistently across worktrees.
