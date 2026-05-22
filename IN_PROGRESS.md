# In Progress

Active tasks for the current development cycle.

Workflow: `ROADMAP.md` → start a task → move here → finish → move to `HISTORY.md`.

When a PR closes a task, the **same PR** must update both `IN_PROGRESS.md` (remove) and `HISTORY.md` (add). Do not defer either to a follow-up commit on the protected branch after merge.

---

### Add `--from <base>` flag to `git wt switch`

`git wt switch <new-branch>` always cuts the new branch from the current worktree's `HEAD`. When the skill at [skills/git-wt/SKILL.md](skills/git-wt/SKILL.md) follows its base-branch policy and the base (e.g. `dev`) is not checked out in any worktree (Path B), it has to fall back to raw `git worktree add -b <new> <dest> <base>` because the CLI cannot accept an explicit base. This task adds a `--from <base>` flag so a single `git wt switch` call covers both paths and the skill no longer needs the raw fallback.

- [ ] Parse `--from <base>` and `--from=<base>` in `cmd_switch` ([bin/git-wt:95-159](bin/git-wt#L95-L159)). Flag may appear before or after the branch name. Reject unknown flags and `--from` without a value.
- [ ] When `--from` is supplied, validate `<base>` with `git rev-parse --verify --quiet <base>^{commit}`. On failure, error with `base '<base>' does not resolve to a commit`. No auto-fetch — keeping the CLI policy-free; the skill is responsible for refreshing the base.
- [ ] When `--from` is supplied **and** the branch already exists anywhere (worktree, local ref, or `origin/<branch>`), error out instead of silently ignoring `--from`. Distinct messages for each case so the user knows which check fired.
- [ ] When `--from` is supplied and the branch does not exist, create the worktree with `git worktree add -b <branch> <dest> <base>`. Do not set tracking — same as the current "new branch from HEAD" path. The `report` already suggests `git push -u origin <branch>` for upstream setup.
- [ ] Preserve the stdout contract: destination path remains the final line of stdout. All errors and progress go to stderr via the existing helpers ([bin/git-wt:23-45](bin/git-wt#L23-L45)).
- [ ] Update `usage()` in [bin/git-wt:47-62](bin/git-wt#L47-L62) to document the new flag and the "errors if branch already exists" rule.
- [ ] Bump `VERSION` at [bin/git-wt:6](bin/git-wt#L6) to `0.2.0` (new public CLI surface).
- [ ] Update [skills/git-wt/SKILL.md](skills/git-wt/SKILL.md) base-branch policy: replace the raw `git worktree add` fallback in Path B with `git wt switch <new> --from <base>`. Simplify Path A so it also ends with `git wt switch <new> --from <base>` instead of `cd <base-wt> && git wt switch <new>`. The stash/pull/pop dance in Path A stays (it updates the base ref); only the final worktree-creation step becomes uniform.
- [ ] Update the README usage section if it documents `switch` flags.
- [ ] Manual smoke test in a throwaway repo: (a) `switch <new> --from <existing-local-ref>` cuts correctly; (b) `switch <new> --from origin/<branch>` works without setting tracking; (c) `switch <existing> --from <base>` errors with a clear message; (d) `switch <new> --from <bogus>` errors; (e) the stdout last-line contract still holds (`tail -n1` yields the destination path).

**Acceptance:** the skill's Base branch policy in [skills/git-wt/SKILL.md](skills/git-wt/SKILL.md) no longer contains any raw `git worktree add` invocation; both Path A and Path B end with `git wt switch <new> --from <base>`; `git wt switch <new> --from <base>` cuts a new branch from `<base>` when the branch does not exist; the same call errors with a clear message when the branch already exists anywhere; `--from` with a non-resolvable base errors out; stdout's last line remains the destination path; `git wt help` documents the flag; `VERSION` is bumped to `0.2.0`.
