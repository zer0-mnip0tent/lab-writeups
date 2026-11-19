# OverTheWire Bandit: Level 18 → Level 19

## Objective

The password for the next level is stored in a file named `readme` in the home directory of `bandit18`.

However, the account's `.bashrc` has been modified to terminate the SSH login session immediately, preventing normal interactive access.

The objective is to retrieve the contents of `readme` without relying on a conventional interactive SSH session.

## Approach

A normal SSH connection attempts to provide an interactive login shell. In this level, that connection is immediately terminated, displaying:

```text
Byebye!
```

Rather than attempting to establish an interactive session, SSH can be used to execute a command directly on the remote machine.

The `ssh` client accepts a remote command as an argument. When provided, the command is executed on the remote host instead of starting a login shell.

This allows the contents of `readme` to be retrieved directly.

## Solution

Execute the following command from the local terminal:

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org 'cat ~/readme'
```

Enter the password for `bandit18` when prompted.

The remote command reads `readme` and prints its contents to the local terminal. The returned string is the password required to access `bandit19`.

## Command Breakdown

| Component | Description |
|---|---|
| `ssh` | Establishes a secure connection to the remote machine. |
| `-p 2220` | Specifies port 2220 for the SSH connection. |
| `bandit18@bandit.labs.overthewire.org` | Specifies the remote username and hostname. |
| `cat ~/readme` | Executes `cat` on the remote machine to display the contents of `readme`. |

The `~` symbol represents the home directory of the remote user, `bandit18`.

## Why This Works

The important distinction is between requesting an interactive login shell and requesting execution of a remote command.

Without a command, SSH attempts to establish an interactive session. In this challenge, the modified shell configuration causes that session to terminate immediately.

By appending `cat ~/readme` to the SSH command, the client requests execution of that command on the remote host instead of starting a login shell.

The output of the remote command is returned through the SSH connection and displayed in the local terminal.

This is a feature of SSH, not an authentication bypass or an exploitation of a vulnerability.

## Key Takeaways

- **SSH Remote Command Execution:** SSH can execute commands directly on a remote machine without opening an interactive login shell.
- **Interactive vs. Non-interactive Sessions:** These are different ways of invoking a remote shell and executing commands.
- **Bash Configuration:** `.bashrc` is a Bash startup configuration file whose behaviour depends on how Bash is invoked.
- **Remote Command Execution:** Commands can be executed remotely and their standard output returned to the local terminal.

An important distinction is that this method does not download the remote file. It executes `cat` on the remote machine and transfers the resulting output through SSH.

## References

- [OverTheWire – Bandit Level 18 → 19](https://overthewire.org/wargames/bandit/bandit19.html)
- [OpenSSH – ssh(1) Manual](https://man.openbsd.org/ssh.1)
- [GNU Bash – Startup Files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html)
