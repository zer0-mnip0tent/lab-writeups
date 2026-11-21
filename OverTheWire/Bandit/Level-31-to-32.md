# OverTheWire Bandit: Level 31 → Level 32

**Focus:** Staging ignored files, creating commits and pushing changes to a remote branch  
**Tools:** `ssh`, `git`, `git clone`, `git status`, `git add`, `git commit`, `git push`

## Objective

A Git repository is hosted on the OverTheWire server and is accessible through SSH on port `2220`.

The credentials for `bandit31-git` are the same as those for `bandit31`.

The task is to create and push a file to the remote repository with the following requirements:

| Property | Required value |
|---|---|
| Filename | `key.txt` |
| Content | `May I come in?` |
| Branch | `master` |

Official challenge: <https://overthewire.org/wargames/bandit/bandit32.html>

## Solution

### 1. Clone the repository

Execute the following command from the local machine:

```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
```

Enter the password for `bandit31` when prompted.

Navigate into the cloned repository:

```bash
cd repo
```

### 2. Inspect the repository instructions

List the available files:

```bash
ls -la
```

Read the README:

```bash
cat README.md
```

The file specifies the required filename, content and target branch.

The next step is to create the requested file in the root of the cloned repository, alongside the README.

### 3. Create the required file

Create `key.txt` with the specified content:

```bash
echo "May I come in?" > key.txt
```

Verify the file:

```bash
cat key.txt
```

At this point, the file exists in the working directory.

However, Git does not necessarily track every file present in that directory.

### 4. Investigate why Git does not detect the file

Check the repository status:

```bash
git status
```

The output reports:

```text
nothing to commit, working tree clean
```

Despite the file existing, Git does not list it as an untracked file.

Inspect the hidden files:

```bash
ls -la
```

The listing includes `.gitignore`.

Read its contents:

```bash
cat .gitignore
```

The ignore rule is:

```text
*.txt
```

This pattern instructs Git to ignore files with the `.txt` extension, including `key.txt`.

This explains why the file exists on the filesystem but does not appear in the normal Git status output.

### 5. Force-add the ignored file

Normally, a file is staged using:

```bash
git add key.txt
```

However, the ignore rule prevents this file from being added normally.

Git provides the `-f` option to force-add an ignored file:

```bash
git add -f key.txt
```

Verify the staging area:

```bash
git status
```

The output should now contain:

```text
Changes to be committed:
    new file: key.txt
```

The file is now staged and ready to be committed.

### 6. Create a commit

A commit records the staged changes in the local Git repository.

Create the commit:

```bash
git commit -m "Add key.txt"
```

If Git reports that the author identity is unknown, configure a name and email for this repository:

```bash
git config user.name "Your Name"
git config user.email "you@example.com"
```

These settings are repository-specific because `--global` is not used.

Retry the commit:

```bash
git commit -m "Add key.txt"
```

The staged file is now recorded in a new commit on the local `master` branch.

Verify the latest commit:

```bash
git log --stat -1
```

The output should now include `key.txt`.

### 7. Push the commit to the remote repository

A local commit does not automatically update the remote repository.

The final step is to push the current branch:

```bash
git push origin master
```

Here:

- `origin` identifies the remote repository.
- `master` specifies the branch to update.

The remote server processes the push and checks the submitted file against the challenge requirements.

A successful challenge validation returns the information required for the next level. The password itself is omitted from this write-up.

## Why this works

Git uses `.gitignore` to prevent specified files from being added accidentally.

However, ignore rules do not make files inaccessible to Git. The `-f` option overrides the ignore rule for a specific addition.

The workflow consists of three distinct stages:

1. **Working directory:** The file is created or modified.
2. **Staging area:** `git add` prepares the changes for a commit.
3. **Local repository:** `git commit` records the staged changes.

Finally, `git push` transfers the local commit to the remote repository.

The remote server can then validate the submitted file and respond to the challenge.

## Troubleshooting notes

### `fatal: not a git repository`

Running Git commands from outside the cloned repository produces this error.

For example, attempting to push while the terminal is in `~` does not work because the home directory is not the cloned repository.

Navigate into the correct directory:

```bash
cd /path/to/repo
```

Verify that the `.git` directory exists:

```bash
ls -la
```

Then confirm the repository status:

```bash
git status
```

### `git status` reports a clean working tree

A file can exist on the filesystem while being ignored by Git.

Inspect `.gitignore`:

```bash
cat .gitignore
```

If the file matches an ignore pattern, use:

```bash
git add -f <filename>
```

### `git ls-tree master key.txt` returns nothing

`git ls-tree` inspects the contents of a committed Git tree.

A newly created or staged file does not appear in the current commit until it has been committed.

Use `git status` to check the staging area instead.

### `Author identity unknown`

Git requires author information when creating commits.

Configure the name and email locally using `git config`, then retry the commit.

### Understanding the push destination

Check the configured remote:

```bash
git remote -v
```

The repository normally uses `origin` as its remote name.

The challenge specifically requires the `master` branch, so the push must target that branch.

## Key takeaways

- `.gitignore` prevents matching files from being added through normal Git operations.
- `git add -f` force-adds an ignored file.
- `git status` distinguishes between untracked, staged and committed changes.
- `git commit` records staged changes locally.
- Git may require an author name and email before creating commits.
- `git push origin master` sends the local `master` branch to the remote repository.
- A successful local commit does not automatically mean that the remote repository has been updated.

## References

- [OverTheWire Bandit: Level 31 → 32](https://overthewire.org/wargames/bandit/bandit32.html)
- [Git Documentation: gitignore](https://git-scm.com/docs/gitignore)
- [Git Documentation: git-add](https://git-scm.com/docs/git-add)
- [Git Documentation: git-status](https://git-scm.com/docs/git-status)
- [Git Documentation: git-commit](https://git-scm.com/docs/git-commit)
- [Git Documentation: git-push](https://git-scm.com/docs/git-push)
