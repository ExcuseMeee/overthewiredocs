# OverTheWire Bandit: Level 21 → Level 22

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 21.

## Problem Statement

The goal for Bandit Level 21 was to discover the password for Bandit 22. This password was associated with a program running automatically via `cron`, the time-based job scheduler. The task was to examine the configuration in `/etc/cron.d/` to identify the command being executed.

## My Attempts

My approach involved navigating the file system to locate and understand the `cron` configurations.

1.  **Locating `cron` Configuration Files:**
    After logging in, I changed my directory to `/etc/cron.d/` and executed `ls` to list its contents. This revealed several files, including `cronjob_bandit22`, `cronjob_bandit23`, and `cronjob_bandit24`. I also used `file` on all these files to confirm they were ASCII text.

2.  **Investigating `cronjob_bandit22`:**
    I read the content of `cronjob_bandit22`, which contained:

    ```
    @reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
    * * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
    ```

    Consulting `man 5 crontab`, I interpreted these lines to mean that the script `/usr/bin/cronjob_bandit22.sh` was being executed as the `bandit22` user every minute and also once at system reboot, with all its output redirected to `/dev/null`.

3.  **Analyzing the Executed Script:**
    Next, I navigated to `/usr/bin/` and read the content of `cronjob_bandit22.sh`:

    ```bash
    #!/bin/bash
    chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    ```

    This script's content clearly showed that it first set permissive read/write permissions (`644`) on a file in `/tmp/` and then copied the content of `/etc/bandit_pass/bandit22` into that `/tmp` file.

## The Correct Solution

The solution was to simply read the temporary file where the `cron` job was regularly writing the password.

1.  **Reading the Password File:**
    Recognizing that the `cron` script made the password available in `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv` with read permissions, I executed `cat` on that file:

    ```bash
    cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
    ```

    This command successfully displayed the password for Bandit Level 22.

## What I Learned

- **`cron` Job Scheduling:** Gained practical experience with `cron` jobs, particularly those defined in `/etc/cron.d/`. Understood the format and common scheduling patterns (`* * * * *`, `@reboot`).

- **`man 5 crontab`:** Learned to use `man 5 crontab` to understand the specific syntax and behavior of `cron` configuration files.

- **Script Analysis:** Practiced analyzing simple shell scripts to understand their execution flow, including file permission changes (`chmod`) and input/output redirection (`cat ... > ...`, `&> /dev/null`).

- **Temporary Files (`/tmp`):** Identified a common CTF pattern where sensitive information is temporarily written to world-readable files in the `/tmp` directory by automated processes. This emphasizes the importance of secure file handling in real systems.
