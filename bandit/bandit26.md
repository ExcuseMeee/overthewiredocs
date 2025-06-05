# OverTheWire Bandit: Level 26 → Level 27

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 26.

## Problem Statement

The goal for Bandit Level 26 was to obtain the password for Bandit 27. This level directly piggybacked off the solution from Bandit Level 25, which involved gaining a full `bash` shell as `bandit26`. Once inside this `bash` shell, a new executable, `bandit27-do`, was available in the home directory.

## My Attempts

My primary attempt was to directly use the provided executable with `cat`.

1.  **Accessing the `bandit26` Shell:**
    I began by utilizing the solution from Bandit Level 25, which allowed me to break out of the restricted shell and gain a full `bash` shell as the `bandit26` user.

2.  **Identifying the Executable:**
    Once in the `bandit26` shell, I executed `ls` to list the contents of the home directory, confirming the presence of the `bandit27-do` executable.

## The Correct Solution

The solution was straightforward, involving the direct execution of the `bandit27-do` binary to read the password file.

1.  **Executing `bandit27-do` with `cat`:**
    Knowing the `bandit27-do` executable likely functioned similarly to previous "do" binaries (i.e., running commands with elevated privileges), I directly executed it to read the password file:

    ```
    ./bandit27-do cat /etc/bandit_pass/bandit27
    ```

2.  **Retrieving the Password:**
    This command successfully displayed the password for Bandit Level 27 to my terminal.

## What I Learned

- **Chained Exploitation/Solutions:** This level highlighted how solutions in wargames often build upon previous ones. Gaining a full shell in one level can directly enable solving the next, even if the explicit mechanism (like a `setuid` binary) is different.

- **Direct Binary Execution:** Reinforced the pattern of directly executing binaries provided by the challenge (`./<binary_name> <command>`) to perform privileged actions, especially when access to the password file is implied.

- **Common Password Location:** Reaffirmed the `/etc/bandit_pass/<level_name>` path as the consistent location for passwords.
