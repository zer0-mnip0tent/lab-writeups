# OverTheWire Bandit: Level 30 → Level 31

**Focus:** Enumerating Git tags and inspecting annotated tag objects  
**Tools:** `ssh`, `git`, `git clone`, `git tag`, `git show`

## Objective

A Git repository is hosted on the OverTheWire server and can be accessed through SSH on port `2220`.

The password for `bandit30-git` is the same as the password for `bandit30`. The objective is to clone the repository and find the password for the next level.

Official challenge: <https://overthewire.org/wargames/bandit/bandit31.html>

## Solution

### 1. Clone the repository

From the local machine, execute:

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
```

Enter the password for `bandit30` when prompted.

Navigate into the repository:

```bash
cd repo
```

### 2. Inspect the repository contents

List the files:

```bash
ls -la
```

Inspect the README:

```bash
cat README.md
```

The README contains the following message:

```text
just an epmty file... muahaha
```

The file provides no useful credentials.

### 3. Examine the commits and branches

Since the README contains no useful information, inspect the commit history:

```bash
git log --oneline
```

Then enumerate the available branches:

```bash
git branch -a
```

The repository contains only the default `master` branch.

Neither the current file contents nor the available branch information provides the required password.

### 4. Inspect Git tags

Git also supports tags, which are references used to identify particular points or objects in a repository.

Unlike branches, tags are not necessarily visible through the normal branch listing.

List the available tags:

```bash
git tag
```

The relevant tag is named:

```text
secret
```

### 5. Inspect the tag

Use `git show` to inspect the tag:

```bash
git show secret
```

The output includes the annotated tag information and its associated message.

The message contains the password required for the next level.

The password is omitted from this write-up.

## Why this works

Git provides several types of references, including branches and tags.

A branch normally moves forward as new commits are added. A tag is generally used as a fixed reference to a particular object or point in the repository.

Annotated tags also contain their own metadata and message.

In this challenge, the required information is stored in the tag rather than in the current README or the default branch.

The command:

```bash
git tag
```

lists available tags, while:

```bash
git show secret
```

displays the information associated with the relevant tag.

## Troubleshooting notes

### `git tag` produces no output

The intended repository for this level contains a tag named `secret`.

If the command produces no output, verify that the correct repository was cloned:

```bash
git remote -v
```

The remote should point to the `bandit30-git` repository.

If necessary, fetch the remote tags:

```bash
git fetch --tags origin
```

List them again:

```bash
git tag
```

The recorded terminal session initially showed no output from `git tag`, so the repository location and fetched references should be checked before proceeding.

### `git ls-tree` displays usage information

The `git ls-tree` command requires a tree-ish argument, such as `HEAD` or a branch name.

For example:

```bash
git ls-tree HEAD
```

It lists the contents of a particular Git tree. It is not a command for listing Git tags.

### Understanding `git cat-file`

`git cat-file -p <object-id>` displays the contents of a Git object.

Using the blob ID obtained from the current README's tree simply displays that README's contents again. It does not automatically locate other tags or hidden credentials.

The tag is the relevant reference type for this challenge.

## Key takeaways

- Git repositories contain different types of references.
- `git branch -a` and `git tag` inspect different parts of a repository.
- `git tag` lists available tags.
- Annotated tags can contain their own messages and metadata.
- `git show` can inspect commits, tags and other Git objects.
- A clean working tree does not mean that a repository contains no other useful Git references.

## References

- [OverTheWire Bandit: Level 30 → 31](https://overthewire.org/wargames/bandit/bandit31.html)
- [Git Documentation: git-tag](https://git-scm.com/docs/git-tag)
- [Git Documentation: git-show](https://git-scm.com/docs/git-show)
- [Git Documentation: git-fetch](https://git-scm.com/docs/git-fetch)
