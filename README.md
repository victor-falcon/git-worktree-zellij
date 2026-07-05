# git-worktree-zellij

Small `zsh` helper to create and delete Git worktrees, with optional tab automation for [Zellij](https://zellij.dev) and [Muxy](https://muxy.app).

## Features

- Creates a worktree for an existing or new branch
- Deletes a worktree and optionally deletes the branch after merge checks
- Supports `del|delete -f|--force` to skip prompts, delete the branch, and close the current tab automatically
- Opens the worktree in a new tab when run inside Zellij or Muxy
- Runs an optional repo-local setup script inside the new worktree
- Changes the new shell into the worktree before running setup
- Works from any subdirectory inside the Git repository

## Requirements

- `git`
- `zsh`
- `fzf`
- `zellij` or `muxy` for automatic tab opening

The script still works without a multiplexer; it prints the target path and exits.

## Multiplexer behavior

The active multiplexer is detected automatically (`$ZELLIJ` vs `$MUXY_PANE_ID`):

- **Zellij** — creates the worktree with `git worktree add` under `WT_WORKTREES_DIR` (default: `../worktrees`) and opens it in a new Zellij tab (or the current one with `-c`).
- **Muxy** — delegates to `muxy create-worktree`, so the worktree lands in Muxy's configured location (`muxy.general.defaultWorktreeParentPath`) and opens as a native Muxy tab. `WT_WORKTREES_DIR` and `-c/--current` are ignored here. On `del`, it runs `muxy refresh-worktrees` so Muxy's sidebar stays in sync.

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
worktree ls                              # List existing worktrees with fzf and open one in a new tab

worktree <branch-name>               # Create a new worktree
worktree -c <branch-name>            # Create a new worktree in the current tab (Zellij only)
worktree --current <branch-name>     # Same

worktree del [-f]                    # Delete current worktree, it will ask you before deleting branch

worktree help
```

## Optional Setup Scripts

After creating a worktree, the script looks for the first existing setup script in the source repository and runs it inside the new worktree:

- `worktree.sh`
- `.local-dev/scripts/setup-worktree.sh`

You can change the lookup order by editing `bin/git-worktree-zellij`.

## Configuration

Environment variables:

- `WT_DEFAULT_BRANCH`: override the branch used for merge checks before branch deletion
- `WT_WORKTREES_DIR`: override the parent directory used to store worktrees (Zellij / no multiplexer; ignored under Muxy, which uses its own configured location)

Delete options:

- `-f`, `--force`: force-remove the worktree, delete the branch without asking, and close the current tab automatically
- `-c`, `--current`, `--current-tab`: reuse the current tab; when reusing, ask before renaming it (Zellij only)

Example:

```bash
export WT_DEFAULT_BRANCH=main
export WT_WORKTREES_DIR="$HOME/Projects/worktrees"
```

## Notes

- Branch deletion is cautious by default: merged branches are deleted normally, unmerged branches require confirmation
- With `-f/--force`, branch deletion is unconditional (`git branch -D`) and the current tab is closed automatically
- Muxy worktrees created by `muxy create-worktree` are removed from Git by `del`; Muxy prunes them from its sidebar via `refresh-worktrees` (there is no Muxy CLI command to delete a worktree directly)
- Before asking to delete a branch, the merge check refreshes `origin/<default-branch>` and prefers that ref when available
- The squash-merge detection uses Git plumbing to compare branch changes before deletion
- The script is designed for local developer workflows, not for server automation

## License

MIT
