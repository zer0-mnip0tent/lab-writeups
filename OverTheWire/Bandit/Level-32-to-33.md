# OverTheWire Bandit: Level 32 → Level 33

**Focus:** Escaping a restricted uppercase shell using shell parameter expansion  
**Tools:** `sh`, `whoami`, `cat`

## Objective

After the Git challenges, the next level introduces a restricted shell that converts entered commands to uppercase.

Ordinary Linux commands become unusable because Linux command names and paths are case-sensitive.

The objective is to escape the restricted shell and obtain access to a normal shell environment.

Official challenge: <https://overthewire.org/wargames/bandit/bandit33.html>

## Solution

### 1. Examine the restricted shell

After connecting as `bandit32`, the terminal displays:

```text
WELCOME TO THE UPPERCASE SHELL
>>
```

Attempts to execute ordinary commands immediately produce errors.

For example:

```text
>> ls
sh: 1: LS: Permission denied
```

The shell converts the entered command to uppercase before attempting to execute it.

Another attempt using an absolute path produces the same problem:

```text
>> /bin/ls
sh: 1: /BIN/LS: not found
```

This demonstrates that typing an absolute executable path does not bypass the uppercase conversion.

### 2. Understand the limitation

Linux command names and paths are case-sensitive.

For example:

```text
/bin/ls
```

and:

```text
/BIN/LS
```

are different paths.

Consequently, ordinary commands such as `ls`, `cat`, `pwd` and `whoami` cannot be executed successfully through the restricted interface.

The next step is to consider shell features that process input rather than relying on ordinary command names typed directly.

### 3. Investigate shell parameters

Shells provide special parameters that contain information about the currently running shell and its execution environment.

One of these is:

```bash
$0
```

The special parameter `$0` expands to the name of the shell or shell script.

In this challenge, the expanded value can be used to invoke a normal shell without directly typing its lowercase command name.

### 4. Escape the uppercase shell

Enter:

```bash
$0
```

After execution, the prompt changes from:

```text
>>
```

to:

```text
$
```

This indicates that the restricted interface has been escaped and a normal shell is now available.

The distinction is important: the uppercase wrapper is no longer restricting ordinary command execution.

### 5. Verify the shell environment

Check the current user:

```bash
whoami
```

The resulting identity should be `bandit33`.

The normal shell can now execute commands without the uppercase restriction.

Retrieve the password for the final available level:

```bash
cat /etc/bandit_pass/bandit33
```

The password is intentionally omitted from this write-up.

## Why this works

`$0` is a special shell parameter that represents the name used to invoke the current shell or script.

When the shell processes `$0`, it expands the parameter before executing the resulting command.

In this challenge, the expansion invokes a new normal shell rather than requiring a lowercase command to be entered directly through the uppercase interface.

This works because the restriction affects the literal input, while the shell can still perform parameter expansion.

The important distinction is between typing a lowercase command and allowing the shell to generate the command through expansion.

## Troubleshooting notes

### `Permission denied` after entering ordinary commands

Errors such as:

```text
sh: 1: LS: Permission denied
```

are expected while using the restricted shell.

The command has been converted to uppercase, so the system attempts to execute a different command name.

### Absolute paths do not solve the problem

Typing:

```bash
/bin/ls
```

does not help because the path itself is also converted to uppercase.

The restriction is not limited to command names.

### `$PATH` is not a command

Entering:

```bash
$PATH
```

expands the variable into a list of executable search directories.

It does not execute a shell or directly display the contents of those directories.

This is why the attempt resulted in a command-not-found error.

### `cat` without a filename

Running:

```bash
cat
```

without specifying a file causes `cat` to read from standard input.

Anything entered into the terminal is echoed back by `cat`, which explains why the text appeared twice.

Press `Ctrl + D` to signal end-of-file and return to the shell prompt.

### The `uppershell` entry

After escaping, listing the current directory can reveal an entry named `uppershell`.

This is part of the challenge environment. Removing it is unnecessary for completing the level.

## Key takeaways

- Restricted shells can impose limitations on ordinary command execution.
- Linux command names and paths are case-sensitive.
- Absolute paths do not help when the entire input is transformed.
- `$0` is a special shell parameter representing the shell or script name.
- Shell parameter expansion can provide a way to execute commands without typing their literal names.
- A changed prompt can indicate that a different shell environment has been started.
- `cat` without a filename reads from standard input rather than opening a file.

## References

- [OverTheWire Bandit: Level 32 → 33](https://overthewire.org/wargames/bandit/bandit33.html)
- [GNU Bash Reference Manual: Special Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)
- [Linux Manual: bash](https://man7.org/linux/man-pages/man1/bash.1.html)
- [Linux Manual: sh](https://man7.org/linux/man-pages/man1/sh.1p.html)
