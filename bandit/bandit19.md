# OverTheWire Bandit: Level 19 → Level 20

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 19.

## Problem Statement

The goal for Bandit Level 19 was to get the password for Bandit 20. This involved using a `setuid` binary in the home directory, executing it without arguments to learn its usage, and then finding the password in `/etc/bandit_pass` after using the binary.

## My Attempts

My initial steps focused on identifying and understanding the special binary.

1.  **Listing Home Directory Contents:**
    After logging in, I ran `ls` to see what files were present in the home directory. I found a single file named `bandit20-do`.

2.  **Misunderstanding `setuid`:**
    Initially, I misunderstood the instruction to "use the `setuid` binary" and attempted to execute "setuid" as a command:

    ```bash
    setuid
    ```

    This resulted in a "command not found" warning, indicating that `setuid` itself is not a directly executable command in that context.

3.  **Identifying and Executing the Correct Binary:**
    I then realized that `bandit20-do` was the actual "setuid binary" mentioned in the goal. Following the instruction to execute it without arguments to find its usage, I ran:

    ```bash
    ./bandit20-do
    ```

    Executing this binary revealed that it allowed me to run commands as the `bandit20` user.

## The Correct Solution

With the knowledge that `bandit20-do` could execute commands as the `bandit20` user, the solution was straightforward: use it to read the password file for `bandit20`.

1.  **Executing `cat` as `bandit20`:**
    Knowing the password location (`/etc/bandit_pass/bandit20`) and the binary's capability, I used `bandit20-do` to execute the `cat` command on the password file:

    ```bash
    ./bandit20-do cat /etc/bandit_pass/bandit20
    ```

2.  **Retrieving the Password:**
    This command successfully displayed the password for Bandit Level 20 on my screen.

## What I Learned

* **Setuid Binaries:** Gained a practical understanding of `setuid` binaries. These are executable files that run with the permissions of the file's owner (in this case, `bandit20`), rather than the permissions of the user executing them (`bandit19`). This allows lower-privileged users to perform specific tasks requiring higher privileges.

* **Executing Local Binaries:** Reinforced the standard way to execute a binary located in the current directory using `./` (e.g., `./bandit20-do`), especially when the directory is not in the system's `PATH`.

* **Privilege Escalation via `setuid`:** Learned how a `setuid` binary can be a mechanism for privilege escalation, allowing a user to temporarily gain permissions necessary to access protected resources (like `/etc/bandit_pass/bandit20`).

* **Reading Password Files:** Confirmed the common location for Bandit level passwords (`/etc/bandit_pass/<level_name>`).