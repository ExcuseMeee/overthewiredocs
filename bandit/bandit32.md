# OverTheWire Bandit: Level 32 → Level 33

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 32.

## Problem Statement

The goal for Bandit Level 32 was to escape from a restricted shell to gain access to Bandit 33. The challenge implied that the login shell for `bandit32` was not a standard `bash` shell, but something else that needed to be identified and broken out of.

## My Attempts

My attempts focused on diagnosing the restricted shell's behavior and identifying its nature, then finding a way to escape.

1.  **Observing Restricted Shell Behavior:**
    After logging into `bandit32`, all input appeared to be converted to uppercase. Attempting to execute normal commands like `ls` resulted in errors like `sh: 1: LS: Permission denied`, indicating that the shell was interpreting commands as uppercase versions and then failing to execute them.

2.  **Identifying the Custom Shell:**
    To understand the shell's configuration, I logged into the previous level (`bandit31`) and read the `/etc/passwd` file. For `bandit32`, the shell was identified as `/home/bandit32/uppershell`.

3.  **Attempting Shell Changes via Environment Variables (Unsuccessful):**
    Within the `uppershell`, I tried executing `$SHELL` and `$BASH` (expecting them to trigger a new shell session). However, `$SHELL` only restarted the `uppershell` itself, and `$BASH` had no discernible effect.

4.  **Leveraging `$0` for Shell Spawn:**
    Recognizing that the `uppershell` itself was likely a simple script running with `sh` (as suggested by the `sh: 1: ...` error format and the shell's shebang, if examined), and knowing that `$0` typically holds the path to the currently executing script/shell, I reasoned that executing `$0` might re-invoke the underlying `sh` interpreter.

## The Correct Solution

The solution involved exploiting a nuance of shell execution to gain a more functional shell, then changing the `SHELL` environment variable to finally get `bash`.

1.  **Spawning an `sh` Session:**
    Within the `uppershell`, executing `$0` successfully bypassed the uppercase conversion for subsequent input and spawned a standard `sh` session. This allowed for normal, case-sensitive command input.

2.  **Changing the `SHELL` Environment Variable:**
    Inside the new `sh` session, the `SHELL` environment variable still pointed to `/home/bandit32/uppershell`. To achieve a `bash` shell, I set the `SHELL` environment variable to `/bin/bash`:

    ```bash
    export SHELL=/bin/bash
    ```

3.  **Executing the New Shell:**
    Finally, executing `$SHELL` again, this time within the standard `sh` session, successfully launched a full `bash` shell as the `bandit33` user.

4.  **Retrieving the Password:**
    From the `bandit33` `bash` shell, I simply read the password file:

    ```bash
    cat /etc/bandit_pass/bandit33
    ```

    This command successfully displayed the password for Bandit Level 33.

## What I Learned

- **Restricted Shell Bypasses:** Gained a powerful technique for escaping restricted shells by:

  - Identifying the underlying interpreter (`/bin/sh` via analysis of `/etc/passwd` and error messages).

  - Leveraging `$0` to re-execute the underlying interpreter, often bypassing input filtering or restrictions of the custom shell.

- **Shell vs. Environment Variables:** Deepened understanding of the distinction between shell variables (like special parameters such as `$0`) and environment variables (`$SHELL`, `PATH`).

- **`SHELL` Environment Variable:** Learned how the `SHELL` environment variable influences the default shell used and how manipulating it in conjunction with its execution (`$SHELL`) can lead to gaining a more privileged or functional shell.

- **Creative Problem Solving:** This level highlighted the need for creative thinking to exploit unexpected behaviors or features of seemingly simple programs (`uppershell`, shell variable expansion) to achieve a full shell.
