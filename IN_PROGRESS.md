# In Progress

Active tasks for the current development cycle.

Workflow: `ROADMAP.md` → start a task → move here → finish → move to `HISTORY.md`.

When a PR closes a task, the **same PR** must update both `IN_PROGRESS.md` (remove) and `HISTORY.md` (add). Do not defer either to a follow-up commit on the protected branch after merge.

---

### Bootstrap new worktree on `git wt switch`: auto-copy `.env` and offer dependency install

When `git wt switch` creates (or first enters) a worktree, getting it to a runnable state still takes manual work: copying ignored env files from the main worktree and running the right package manager by hand. This task makes `switch` do both, while keeping the current stdout contract intact.

- [ ] Detect the project's language/toolchain by scanning the new worktree for marker files (e.g. `package.json` + lockfile variant for npm/pnpm/yarn/bun, `pyproject.toml` / `requirements.txt` / `Pipfile` / `poetry.lock`, `Cargo.toml`, `go.mod`, `Gemfile`, `composer.json`). Support multiple matches (polyglot repos) by listing all detected toolchains.
- [ ] After the worktree is ready, prompt the user (on stderr, reading from the controlling tty) whether to run the install command for each detected toolchain. Show the exact command before running it. Default answer must be safe for non-interactive shells.
- [ ] Auto-copy `.env` (and `.env.local`, `.env.*` variants if present) from the main worktree into the new worktree without prompting. Skip silently if no env files exist in the source. Never overwrite an existing file in the destination — log a warning to stderr instead.
- [ ] Preserve the stdout contract: the final line of stdout must remain the destination path. All prompts, progress, and logs go to stderr via the existing `info` / `warn` / `report` helpers ([bin/git-wt:23-45](bin/git-wt#L23-L45)).
- [ ] Add non-interactive flags: at minimum `--no-deps` (skip the install prompt entirely), `--no-env` (skip env-file copy), and `--yes` / `-y` (auto-accept the install prompt for the detected toolchain). Define corresponding env vars (e.g. `GWT_NO_DEPS`, `GWT_NO_ENV`, `GWT_ASSUME_YES`) so CI can opt out without arg plumbing. When stdin is not a tty, default to non-interactive (skip install, still copy env).
- [ ] Document the new flags and the auto-copy behavior in `git wt help` output and in the README usage section. Mention which env files are copied and the "never overwrite" rule.
- [ ] Update the bundled skill at [skills/git-wt/SKILL.md](skills/git-wt/SKILL.md) so agents know about the new flags and the env-copy behavior.

**Acceptance:** running `git wt switch <new-branch>` in a Node repo with a `.env` in the main worktree results in (a) the new worktree containing a copy of `.env`, (b) a prompt offering to run the detected install command, (c) the destination path still being the only thing on stdout, and (d) `git wt switch --no-deps --no-env <branch>` and the equivalent env-var form completing without any prompts. `git wt help` lists the new flags.
