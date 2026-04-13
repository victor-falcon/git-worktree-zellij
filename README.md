# git-worktree-zellij

Small `zsh` helper to create and delete Git worktrees, with optional Zellij tab automation.

## Features

- Creates a worktree for an existing or new branch
- Deletes a worktree and optionally deletes the branch after merge checks
- Supports `del|delete -f|--force` to skip the dirty-worktree prompt
- Opens the worktree in a new Zellij tab when run inside Zellij
- Runs an optional repo-local setup script inside the new worktree
- Changes the new Zellij shell into the worktree before running setup
- Works from any subdirectory inside the Git repository

## Requirements

- `git`
- `zsh`
- `zellij` for automatic tab opening

The script still works without Zellij; it prints the target path and exits.

## Install

Clone the repository somewhere in your `PATH`, or symlink the executable:

```bash
git clone https://github.com/victor-falcon/git-worktree-zellij.git ~/Projects/git-worktree-zellij
ln -s ~/Projects/git-worktree-zellij/bin/git-worktree-zellij ~/.local/bin/git-worktree-zellij
```

You can also add an alias:

```bash
alias worktree="$HOME/Projects/git-worktree-zellij/bin/git-worktree-zellij"
```

## Usage

```bash
worktree <branch-name>
worktree <branch-name> -c
worktree <branch-name> --current
worktree new <branch-name>
worktree new -c <branch-name>
worktree new --current <branch-name>
worktree del [-f] [branch-name]
worktree delete [-f] [branch-name]
worktree help
```

## Worktree Layout

By default, worktrees are created next to the repo root under:

```text
../worktrees/<repo-name>/<branch-name>
```

Example:

```text
~/Projects/my-app
~/Projects/worktrees/my-app/feature/login
```

## Optional Setup Scripts

After creating a worktree, the script looks for the first existing setup script in the source repository and runs it inside the new worktree:

- `worktree.sh`
- `.local-dev/scripts/setup-worktree.sh`

You can change the lookup order by editing `bin/git-worktree-zellij`.

## Configuration

Environment variables:

- `WT_DEFAULT_BRANCH`: override the branch used for merge checks before branch deletion
- `WT_WORKTREES_DIR`: override the parent directory used to store worktrees

Delete options:

- `-f`, `--force`: remove the worktree without prompting when it has uncommitted changes
- `-c`, `--current`, `--current-tab`: reuse the current Zellij tab; when reusing, ask before renaming it

Example:

```bash
export WT_DEFAULT_BRANCH=main
export WT_WORKTREES_DIR="$HOME/Projects/worktrees"
```

## Notes

- Branch deletion is cautious: merged branches are deleted normally, unmerged branches require confirmation
- The squash-merge detection uses Git plumbing to compare branch changes before deletion
- The script is designed for local developer workflows, not for server automation

## License

MIT
