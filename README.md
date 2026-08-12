# git-worktree

Small `zsh` helper to create and delete Git worktrees, with optional tab automation for [Herdr](https://herdr.dev), [Zellij](https://zellij.dev) and [Muxy](https://muxy.app).

## Features

- Creates a worktree for an existing or new branch
- Deletes a worktree and optionally deletes the branch after merge checks
- Supports `del|delete -f|--force` to skip prompts, delete the branch, and close the current tab automatically
- Opens the worktree in a new tab (or workspace) when run inside Herdr, Zellij or Muxy
- Runs an optional repo-local setup script inside the new worktree
- Changes the new shell into the worktree before running setup
- Works from any subdirectory inside the Git repository

## Requirements

- `git`
- `zsh`
- `fzf`
- `herdr`, `zellij` or `muxy` for automatic tab opening
- `jq` (only needed under Herdr, to parse its JSON output)

The script still works without a multiplexer; it prints the target path and exits.

## Multiplexer behavior

The active multiplexer is detected automatically (`$HERDR_PANE_ID` vs `$ZELLIJ` vs `$MUXY_PANE_ID`):

- **Herdr** — delegates to `herdr worktree create` (or `worktree open` if the branch already has a worktree), so the worktree lands in Herdr's configured location (`[worktrees].directory`) and opens as its own Herdr workspace, exactly like the UI does. `WT_WORKTREES_DIR` and `-c/--current` are ignored here. On `del`, the workspace is closed with `herdr worktree remove`.
- **Zellij** — creates the worktree with `git worktree add` under `WT_WORKTREES_DIR` (default: `../worktrees`) and opens it in a new Zellij tab (or the current one with `-c`).
- **Muxy** — delegates to `muxy create-worktree`, so the worktree lands in Muxy's configured location (`muxy.general.defaultWorktreeParentPath`) and opens as a native Muxy tab. `WT_WORKTREES_DIR` and `-c/--current` are ignored here. On `del`, it runs `muxy refresh-worktrees` so Muxy's sidebar stays in sync.

## Install

Clone the repository somewhere in your `PATH`, or symlink the executable:

```bash
git clone https://github.com/victor-falcon/git-worktree.git ~/Projects/git-worktree
# The link name is what you type: `worktree` here, rename it if you prefer another command
ln -s ~/Projects/git-worktree/bin/git-worktree ~/.local/bin/worktree
```

You can also add an alias:

```bash
# Same idea: the alias name is the command you type
alias worktree="$HOME/Projects/git-worktree/bin/git-worktree"
```

## Usage

```bash
worktree ls                              # List existing worktrees with fzf and open one in a new tab

worktree <branch-name>               # Create a new worktree
worktree -c <branch-name>            # Create a new worktree in the current tab (Zellij only)
worktree --current <branch-name>     # Same

worktree -s                          # Run the setup script inside the current worktree

worktree del [-f]                    # Delete current worktree, it will ask you before deleting branch

worktree help
```

## Optional Setup Scripts

After creating a worktree, the script looks for a setup script and runs it inside the new worktree. Lookup order:

1. **Personal override**, outside the repo: `~/.config/worktree-setup/<repo-folder-name>.sh` (e.g. `~/.config/worktree-setup/my-app.sh` for a repo cloned into `my-app/`). Useful for repos where you can't (or don't want to) commit a setup script. Change the directory with `WT_PERSONAL_SETUP_DIR`.
2. **In-repo scripts**, first match wins:
   - `worktree.sh`
   - `.local-dev/scripts/setup-worktree.sh`

The script runs with the new worktree as the working directory and receives the source repository path as `$1`, so you can copy files from it (`cp "$1/.env" .`). It is made executable automatically if it isn't. You can change the in-repo lookup order by editing `bin/git-worktree`.

## Configuration

Environment variables:

- `WT_DEFAULT_BRANCH`: override the branch used for merge checks before branch deletion
- `WT_WORKTREES_DIR`: override the parent directory used to store worktrees (Zellij / no multiplexer; ignored under Herdr and Muxy, which use their own configured location)
- `WT_PERSONAL_SETUP_DIR`: override the directory holding personal per-repo setup scripts (default: `~/.config/worktree-setup`)

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
- Under Herdr, `-f/--force` closes the worktree's workspace without prompting
- Muxy worktrees created by `muxy create-worktree` are removed from Git by `del`; Muxy prunes them from its sidebar via `refresh-worktrees` (there is no Muxy CLI command to delete a worktree directly)
- Before asking to delete a branch, the merge check refreshes `origin/<default-branch>` and prefers that ref when available
- The squash-merge detection uses Git plumbing to compare branch changes before deletion
- The script is designed for local developer workflows, not for server automation

## License

MIT
