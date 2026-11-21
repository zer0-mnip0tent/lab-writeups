# OverTheWire Bandit: Level 28 → Level 29

**Focus:** Recovering previously committed information from Git history  
**Tools:** `ssh`, `git`, `git clone`, `git log`, `git show`, `cat`

## Objective

A Git repository is hosted on the OverTheWire server and can be accessed through SSH on port `2220`.

The password for `bandit28-git` is the same as the password for `bandit28`. The objective is to clone the repository locally and retrieve the password for the next level.

Official challenge: <https://overthewire.org/wargames/bandit/bandit29.html>

## Solution

### 1. Clone the repository

The repository must be cloned from the local machine rather than by attempting to access its files through an interactive SSH session.

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
```

Enter the password for `bandit28` when prompted.

Navigate into the newly created repository:

```bash
cd repo
```

### 2. Inspect the README

List the repository contents:

```bash
ls -la
```

Read the README:

```bash
cat README.md
```

The file contains a credentials section with the username for the next level. However, the password field contains a placeholder:

```text
password: xxxxxxxxxx
```

The placeholder indicates that the original value is no longer present in the current version of the file.

Instead of looking for another file, the next step is to investigate the repository's commit history.

### 3. Examine the commit history

Git records changes through commits. Even when information is removed from a file in a later commit, the earlier committed version can remain available in the repository history.

Start by listing the commits:

```bash
git log --oneline
```

For a more useful view, inspect the changes introduced by each commit:

```bash
git log -p
```

To restrict the output to the README:

```bash
git log -p -- README.md
```

The `-p` option displays the patches associated with the commits, making it possible to examine how the file changed over time.

### 4. Locate the previous version

Inspect the differences involving the password field.

A change similar to the following is the important clue:

```diff
- password: <previous value>
+ password: xxxxxxxxxx
```

In a Git diff:

- `-` represents a line removed from the previous version.
- `+` represents a line added in the newer version.

The previous version of the README therefore contains information that is absent from the current working copy.

A particular commit can also be inspected directly:

```bash
git show <commit-hash>
```

Alternatively, retrieve the README from that commit:

```bash
git show <commit-hash>:README.md
```

Examine the historical version to identify the original credential. The password itself is omitted from this write-up.

## Why this works

Git maintains a history of committed changes rather than storing only the latest version of each file.

Replacing a password with a placeholder in a later commit does not automatically remove the original password from earlier commits.

`git log -p` makes these changes visible by displaying the patch associated with each commit.

This is also an important security consideration: removing a committed secret from the latest version of a repository does not guarantee that the secret has been removed from its history.

## Troubleshooting notes

### `git log` only displays commit information

The normal `git log` command primarily displays commit metadata, including author, date and commit message.

Use:

```bash
git log -p
```

to inspect the actual changes introduced by each commit.

### Understanding the diff symbols

The `-` and `+` symbols do not indicate errors. They identify removed and added lines respectively.

The removed line is particularly important when investigating information that was deleted from a repository.

### Inspecting a specific commit

Use `git show <commit-hash>` to inspect a particular commit instead of repeatedly browsing the entire history.

## Key takeaways

- Git preserves committed versions of files across subsequent changes.
- Removing a secret from the latest version does not erase it from previous commits.
- `git log` lists commit history, while `git log -p` also displays changes.
- `git show` can inspect individual commits and historical file versions.
- Sensitive information should never be committed to a repository in the first place.

## References

- [OverTheWire Bandit: Level 28 → 29](https://overthewire.org/wargames/bandit/bandit29.html)
- [Git Documentation: git-log](https://git-scm.com/docs/git-log)
- [Git Documentation: git-show](https://git-scm.com/docs/git-show)
