# git-wt

A small wrapper around `git worktree` that lets you switch worktrees the way
you switch branches — including actually moving your shell into the target
directory.

```sh
git wt switch feature/login   # creates worktree if missing, cds into it
git wt list                   # shows all worktrees, marks current
git wt rm feature/login       # removes the worktree (warns if dirty)
```

## Why

`git worktree` is great but clumsy day-to-day: a subprocess can't change your
shell's `cwd`, so there's no built-in "switch to this worktree" command.
`git-wt` solves that with a binary plus a tiny shell function that `cd`s into
the path the binary prints.

## Install

```sh
git clone https://github.com/AkaLab-Tech/git-wt.git
cd git-wt
./install.sh
```

The installer:

1. Copies `bin/git-wt` to `~/.local/bin/git-wt` (override with `GWT_INSTALL_DIR`).
2. Injects a wrapper function into `~/.zshrc` and/or `~/.bashrc`, delimited
   by `# >>> git-wt >>>` / `# <<< git-wt <<<` markers (idempotent).
3. Warns if `~/.local/bin` isn't on your `$PATH`.
4. Suggests installing `fzf` if missing (optional, enables interactive picker).

Restart your shell (or `source` your rc) afterwards.

## Usage

| Command | What it does |
| --- | --- |
| `git wt switch <branch>` | Switch to the worktree for `<branch>`. Creates one if it doesn't exist (tracks `origin/<branch>` if it exists remotely, otherwise creates a new branch from `HEAD`). |
| `git wt switch <branch> --from <base>` | Cut a new `<branch>` from `<base>` (any commit-ish: local ref, `origin/<name>`, SHA). Errors if `<branch>` already exists in a worktree, locally, or on `origin`. Does not set upstream tracking and does not auto-fetch. |
| `git wt switch <branch> [--no-env] [--no-deps] [--yes\|-y]` | On create, `git wt switch` also copies `.env` / `.env.*` from the main worktree (never overwriting) and prompts for the install command of every detected toolchain (npm/pnpm/yarn/bun, poetry/pipenv/pip, cargo, go, bundler, composer). `--no-env` skips the env-file copy, `--no-deps` skips the install prompt, `--yes` / `-y` auto-accepts every prompt. Env-var equivalents (set to `1`): `GWT_NO_ENV`, `GWT_NO_DEPS`, `GWT_ASSUME_YES`. Non-tty stdin behaves as `--no-deps` (env copy still runs); use `--yes` to force an install in CI. |
| `git wt switch` | With no arg, opens an `fzf` picker over existing worktrees. |
| `git wt list` | Pretty-prints worktrees, marks the one you're in with `*`. |
| `git wt rm <branch>` | Removes the worktree for `<branch>`. Prompts if it has uncommitted changes. |
| `git wt self-update` | Pulls the latest changes in the recorded clone and re-runs `install.sh`. The clone path is written to `${XDG_CONFIG_HOME:-$HOME/.config}/git-wt/install.conf` by `install.sh`. Errors if the clone is missing or has uncommitted changes. |
| `git wt help` | Show help. |
| `git wt version` | Show version. |

## Staying up to date

Every `git wt` subcommand (except `help` / `version` / `self-update`) runs a rate-limited version check once per day and prints a single stderr nudge if a newer `git-wt` is available upstream. The result is cached at `${XDG_CACHE_HOME:-$HOME/.cache}/git-wt/version-check`; delete that file to force a refresh.

| Knob | Effect |
| --- | --- |
| `--no-version-check` | One-shot opt-out for the current invocation. |
| `GWT_NO_VERSION_CHECK=1` | Persistent opt-out (e.g. in your shell rc). |
| `GWT_VERSION_CHECK_TTL=<seconds>` | Override the 24h cache TTL. |
| `GWT_DEBUG=1` | Log network/parse failures (otherwise silent). |
| Non-tty stdin **and** (`CI` or `GITHUB_ACTIONS` set) | Auto-skipped. |

`git wt self-update` is the matching upgrade path: it `git pull --ff-only`s the recorded clone and re-runs `install.sh` from there. Run it manually whenever the nudge appears.

## Layout

Worktrees are created next to the main repo:

```
~/Work/
├── myrepo/                       # main worktree
└── myrepo-worktrees/
    ├── feature-login/            # branch "feature/login"
    └── hotfix-prod/              # branch "hotfix/prod"
```

Branch names containing `/` are flattened to `-` for the directory name.

## Requirements

- `git` 2.5+ (for worktree support)
- `bash` or `zsh`
- `fzf` (optional — only for argument-less `git wt switch`)

## Uninstall

```sh
./uninstall.sh
```

Removes the binary and the wrapper block from your shell rc files.

## License

MIT — see [LICENSE](LICENSE).
