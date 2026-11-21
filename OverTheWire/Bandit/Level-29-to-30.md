# OverTheWire Bandit: Level 29 → Level 30

**Focus:** Enumerating remote Git branches and inspecting alternative development histories  
**Tools:** `ssh`, `git`, `git clone`, `git log`, `git branch`, `git show`

## Objective

A Git repository is hosted on the OverTheWire server and is accessible through SSH on port `2220`.

The credentials for `bandit29-git` are the same as those for `bandit29`. The objective is to clone the repository locally and locate the password for the next level.

Official challenge: <https://overthewire.org/wargames/bandit/bandit30.html>

## Solution

### 1. Clone the repository

Clone the repository from the local Kali terminal:

```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
```

Enter the password for `bandit29` when prompted.

Navigate into the cloned repository:

```bash
cd repo
```

### 2. Inspect the current README

List the available files:

```bash
ls -la
```

Read the README:

```bash
cat README.md
```

The README contains information about the next level, but the password field displays:

```text
no passwords in production!
```

The current file does not provide the required credential.

### 3. Examine the commit history

Inspect the repository history:

```bash
git log -p
```

The output shows changes to the README, including a username update.

However, the password field remains a placeholder rather than containing an actual credential.

This indicates that searching through the current commit history is not sufficient.

### 4. Enumerate the available branches

Git repositories can contain multiple branches, each representing a separate line of development.

The default branch does not necessarily contain every file or change available in the repository.

List all local and remote branches:

```bash
git branch -a
```

The output includes:

```text
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/sploits-dev
```

The `origin/dev` and `origin/sploits-dev` entries indicate additional remote branches.

The current `master` branch is not the only available version of the repository.

### 5. Inspect the alternative branch

The relevant branch is `origin/dev`.

Its README can be inspected without changing the current branch:

```bash
git show origin/dev:README.md
```

Alternatively, create a local branch that tracks the remote development branch:

```bash
git switch -c dev origin/dev
```

Once switched, inspect the file normally:

```bash
cat README.md
```

The README in the development branch contains the credentials required for the next level.

The password is intentionally omitted from this write-up.

## Why this works

A Git branch is a movable reference to a commit. Different branches can contain different files, changes and commit histories.

The `master` branch contained a README with placeholder credentials, while the development branch contained the required information.

The command:

```bash
git branch -a
```

lists both local branches and remote-tracking branches.

The `origin/dev` reference identifies the development branch on the remote repository.

Using:

```bash
git show origin/dev:README.md
```

allows a file to be inspected directly from that branch without switching the working directory.

## Troubleshooting notes

### `git branch` only displays `master`

The default `git branch` command lists local branches only.

Use:

```bash
git branch -a
```

to include remote-tracking branches.

### Understanding `origin/dev`

`origin` is the default name assigned to the remote repository when cloning.

`origin/dev` represents the remote-tracking reference for its development branch. It is not a separate directory on the local filesystem.

### Switching branches

When switching to a remote branch for the first time, create a local tracking branch:

```bash
git switch -c dev origin/dev
```

This creates the local `dev` branch based on `origin/dev`.

### Inspecting without switching

If only a particular file needs to be examined, use:

```bash
git show origin/dev:README.md
```

This avoids changing the currently checked-out branch.

## Key takeaways

- A Git repository can contain multiple independent development branches.
- `git branch -a` lists both local and remote-tracking branches.
- The default branch does not necessarily contain all relevant information.
- `git show branch:path` can inspect a file from another branch without switching.
- `git switch -c` can create a local branch based on a remote-tracking branch.

## References

- [OverTheWire Bandit: Level 29 → 30](https://overthewire.org/wargames/bandit/bandit30.html)
- [Git Documentation: git-branch](https://git-scm.com/docs/git-branch)
- [Git Documentation: git-switch](https://git-scm.com/docs/git-switch)
- [Git Documentation: git-show](https://git-scm.com/docs/git-show)
