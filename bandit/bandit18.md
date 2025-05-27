# OverTheWire Bandit: Level 18 → Level 19

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 18.

## Problem Statement

The goal for Bandit Level 18 was to find the password for the next level (Bandit 19). The password was located in a file named `readme` in the home directory. The primary obstacle was that the `.bashrc` file had been modified to immediately log out any user connecting via SSH.

## My Attempts

My initial approach followed the standard SSH login procedure, which immediately encountered the `.bashrc` modification.

1.  **Normal SSH Login:**
    I attempted to SSH into `bandit18` as usual:

    ```bash
    ssh -p 2220 bandit18@bandit.labs.overthewire.org
    ```

    After entering the password, I was immediately logged out, confirming the `.bashrc` behavior described in the challenge.

2.  **Researching `ssh` Command Execution:**
    Realizing the interactive shell was being terminated, I consulted the `ssh` man page. I discovered that `ssh` allows you to specify a command to execute directly on the remote host without allocating a pseudo-terminal or running a login shell interactively.

3.  **Listing Directory Contents:**
    To confirm this method would work, I first tried listing the home directory contents:
    ```bash
    ssh -p 2220 bandit18@bandit.labs.overthewire.org ls
    ```
    This command successfully returned the expected listing, showing just the `readme` file.

## The Correct Solution

The correct solution involved leveraging the `ssh` command's ability to execute commands directly on the remote server, bypassing the interactive login shell and the problematic `.bashrc`.

1.  **Executing `cat readme` Remotely:**
    Since `ls` confirmed the `readme` file existed and could be accessed, I used `cat` to read its contents directly:

    ```bash
    ssh -p 2220 bandit18@bandit.labs.overthewire.org cat readme
    ```

2.  **Retrieving the Password:**
    This command successfully outputted the password for Bandit Level 19 to my local terminal.

## What I Learned

- **`ssh` Command Execution:** I learned that `ssh` can execute commands directly on a remote host (e.g., `ssh user@host command`), which is crucial for bypassing `.bashrc` scripts or other login-time configurations that might prevent an interactive shell.
- **Bypassing Login Shell Issues:** This challenge provided a practical scenario where a non-interactive command execution via SSH is necessary to circumvent restrictions imposed by remote login scripts.
- **Non-Interactive File Reading:** Confirmed the ability to read file contents (`cat`) on a remote system without needing to enter an interactive shell.
