# Roadmap

Backlog of work for this project. Tasks flow: `ROADMAP.md` → `IN_PROGRESS.md` → `HISTORY.md`.

Each task lives here as a heading with whatever description it needs (acceptance criteria, design notes, sub-tasks). When work starts, move the block to `IN_PROGRESS.md`.

---

## High Priority

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

## Medium Priority

### Notify users of new `git-wt` versions and offer a `self-update` command

Users have no way to know that a newer `git-wt` has been published. The installer is pull-based (`git pull && ./install.sh` against their original clone), so without a nudge they stay on whatever version they first installed. This task adds a lightweight version check that runs on every `git wt` invocation (rate-limited via a local cache) and a `git wt self-update` subcommand that performs the upgrade end-to-end.

- [ ] Add a version-check helper to [bin/git-wt](bin/git-wt) that compares the embedded `VERSION` ([bin/git-wt:6](bin/git-wt#L6)) against the latest one published on `main`. Fetch the raw `bin/git-wt` from the upstream repo (`https://raw.githubusercontent.com/<owner>/<repo>/main/bin/git-wt`) with `curl` (fall back to `wget`) and extract the `VERSION=` line via `grep`/`awk`. Hardcode the upstream URL as a constant near `VERSION`.
- [ ] Cache the result in `${XDG_CACHE_HOME:-$HOME/.cache}/git-wt/version-check` as a tiny text file (timestamp + latest version + current version at check time). Default TTL is 24h, overridable via `GWT_VERSION_CHECK_TTL` (seconds). Skip the network call entirely if the cache is fresh.
- [ ] Invoke the helper at the start of every subcommand dispatch. When a newer version is detected, print a single concise message to **stderr** (never stdout — `switch` must keep its final-line-is-path contract; see [bin/git-wt:23-45](bin/git-wt#L23-L45)) suggesting `git wt self-update`. Use the existing `info`/`warn` helpers so `NO_COLOR` is honored.
- [ ] Implement `git wt self-update`: locate the original clone (persist its path during install — e.g. write `${XDG_CONFIG_HOME:-$HOME/.config}/git-wt/install.conf` from [install.sh](install.sh)), run `git -C <clone> pull --ff-only` followed by `<clone>/install.sh`, then report the new version on stderr. If the clone is missing or dirty, fail with an actionable error pointing the user at the manual command. Do not auto-run `self-update` from the version-check itself — only suggest it.
- [ ] Add opt-out controls: `--no-version-check` flag and `GWT_NO_VERSION_CHECK=1` env var, both of which short-circuit the helper before any network call. Always skip the check when stdin is not a tty **and** common CI env vars are set (`CI`, `GITHUB_ACTIONS`), so automated runs stay silent and offline-safe.
- [ ] Fail soft on every network/parse error: never block the real subcommand because the check could not reach GitHub. Log nothing on failure unless `GWT_DEBUG=1` is set.
- [ ] Document the behavior, the new flag/env vars, and the `self-update` subcommand in `git wt help` output, in the README usage section, and in the bundled skill at [skills/git-wt/SKILL.md](skills/git-wt/SKILL.md). Mention the cache location and how to clear it.
- [ ] Update [uninstall.sh](uninstall.sh) to remove the cache file and the install config alongside the binary and wrapper block.

**Acceptance:** running any `git wt` subcommand with an outdated local `VERSION` prints a single stderr line suggesting `git wt self-update` (and nothing on stdout that would break `switch`'s path contract); subsequent invocations within the TTL hit the cache and make no network calls; `git wt self-update` upgrades the binary in place from the recorded clone and reports the new version; `--no-version-check`, `GWT_NO_VERSION_CHECK=1`, and a CI-like environment all suppress the check; the helper never aborts a subcommand when GitHub is unreachable; `git wt help` documents the new flag, env vars, and subcommand.

## Low Priority / Ideas

### Configure tracking policy for `.claude/settings.json`

The repo currently leaves `.claude/settings.json` untracked. Each Claude Code session accumulates a different set of approved permissions, so committing the file as-is causes recurring merge conflicts, while leaving it ignored loses any shared baseline. Decide and apply a stable policy.

- [ ] Decide whether the file should be tracked, ignored, or split (e.g. tracked baseline + local override).
- [ ] If tracked: prune ad-hoc / per-session entries from the committed copy and document the convention in `CLAUDE.md`.
- [ ] If ignored: add `.claude/settings.json` (or the whole `.claude/`) to `.gitignore` and, optionally, ship a `.claude/settings.example.json` baseline.
- [ ] Make the rule consistent across worktrees so a clean clone does not show the file as a permanent untracked entry.

**Acceptance:** the chosen policy is documented in `CLAUDE.md`, the repo no longer shows `.claude/settings.json` as a permanent untracked file in clean clones, and the rule applies consistently across worktrees.
