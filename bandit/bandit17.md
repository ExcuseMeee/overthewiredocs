# OverTheWire Bandit: Level 17 → Level 18

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 17.

## Problem Statement

The goal for Bandit Level 17 was to find the password for the next level (Bandit 18). This password was located in `passwords.new` and was explicitly stated to be the _only_ line that had changed between `passwords.old` and `passwords.new` in the home directory.

## My Attempts

My primary attempt to solve this level involved using the `diff` command to find the changed line between the two files.

1.  **Setting up the SSH Key:**
    Before attempting the challenge, I first ensured the RSA private key obtained from Level 16 had the correct permissions for SSH:

    ```bash
    chmod 600 bandit17_key
    ```

2.  **Using `diff` to Find Changes:**
    To identify the single changed line, I executed the `diff` command:

    ```bash
    diff passwords.old passwords.new
    ```

    The output I received was:

    ```
    42c42
    < C6XNBdYOkgt5ARXESMKWWOUwBeaIQZ0Y
    ---
    > x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO
    ```

    This `diff` output, in "context format," indicated that line 42 in `passwords.old` was changed (`c`) to line 42 in `passwords.new`. The line prefixed with `<` showed the old content, and the line prefixed with `>` showed the new content.

## The Correct Solution

The correct solution involved interpreting the output of the `diff` command to extract the password.

1.  **Executing the `diff` command:**
    As performed in my attempts, the key step was running:

    ```bash
    diff passwords.old passwords.new
    ```

2.  **Interpreting the Output:**
    The output `42c42 < old_password --- > new_password` clearly showed that line 42 was changed. The line starting with `>` represented the new content in `passwords.new`.

3.  **Identifying the Password:**
    Based on this, the password for Bandit Level 18 was the string found after the `>` on the changed line.

The password is: `x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO`

## What I Learned

- **`diff` Command:** I learned how to effectively use the `diff` command to compare two files and pinpoint differences, which is invaluable for tracking changes in various contexts.
- **Interpreting `diff` Output:** I gained a clear understanding of the `diff` context format (`NcN`, `<`, `---`, `>`) and how to quickly interpret it to identify lines that have been added, deleted, or changed.
- **File Permissions (`chmod`):** This level reinforced the critical importance of using `chmod 600` to set secure permissions on private key files. This is a fundamental security practice in Linux, as SSH clients will refuse to use keys with more permissive settings.
