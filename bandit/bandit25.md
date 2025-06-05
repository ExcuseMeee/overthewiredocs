# OverTheWire Bandit: Level 25 → Level 26

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 25.

## Problem Statement

The goal for Bandit Level 25 was to log in to Bandit 26. The key challenge was that `bandit26`'s login shell was not `/bin/bash` but a custom, restricted program. I needed to identify this shell, understand its functionality, and find a way to "break out" of it to gain a full interactive shell.

## My Attempts

My attempts focused on diagnosing the restricted shell's behavior and identifying its nature.

1.  **SSH Key Preparation:**
    I first logged into `bandit25`, located the SSH key for `bandit26`, and used `scp` to transfer it to my local device.

2.  **Initial Login Attempts and Behavior Observation:**

    - **Direct SSH Login:** I attempted `ssh -p 2220 -i bandit26key bandit26@bandit.labs.overthewire.org`. This successfully authenticated but immediately logged me out after displaying the full login banner. This indicated a `.bashrc`-like script causing an immediate exit in an interactive shell.

    - **SSH with Command Execution:** Next, I tried executing a command directly: `ssh -p 2220 -i bandit26key bandit26@bandit.labs.overthewire.org pwd`. This displayed only a partial login banner and then hung indefinitely, providing no output or error. This behavior strongly suggested a highly restricted, non-standard shell that did not interpret arbitrary commands.

3.  **Identifying the Restricted Shell:**
    To determine what program was being executed as `bandit26`'s shell, I looked into the `/etc/passwd` file (which is world-readable) from my `bandit25` session. The entry for `bandit26` showed its shell was `/usr/bin/showtext`.

4.  **Analyzing `/usr/bin/showtext`:**
    I then read the content of `/usr/bin/showtext`:

    ```bash
    #!/bin/sh

    export TERM=linux

    exec more ~/text.txt
    exit 0
    ```

    This script clearly indicated that it immediately executes `more` to display the contents of `~/text.txt`, and then exits. This explained the immediate logout (after `more` finishes if the file is small) and the hanging (if `more` is still active, waiting for input).

## The Correct Solution

The solution leveraged the interactive capabilities of the `more` pager to escape into a full shell.

1.  **Activating `more` Mode:**
    Since `more` only becomes interactive if the content it displays exceeds the terminal window's size, I reduced the size of my terminal window. This forced `more` to activate its interactive paging mode when `/usr/bin/showtext` executed.

2.  **Escaping to `vi` Mode:**
    Once in `more`'s interactive mode, I pressed the `v` key. This caused `more` to invoke `vi` (or `vim` in many systems) to edit the current file (`~/text.txt`). This effectively put me into a powerful text editor environment.

3.  **Changing `vi`'s Shell:**
    Knowing that `vi`/`vim` allows executing external commands through its own shell (accessed via `:shell` or `!command`), and that `vi` uses its own `shell` option to determine which shell to use, I changed `vi`'s default shell from `/bin/sh` (or whatever `showtext` might indirectly inherit) to `/bin/bash`:

    ```vim
    :set shell=/bin/bash
    ```

4.  **Spawning a Bash Shell:**
    Finally, I executed `vi`'s shell escape command:

    ```vim
    :shell
    ```

    This successfully dropped me into a full `bash` shell as the `bandit26` user. From there, I could navigate to `/etc/bandit_pass/bandit26` and retrieve the password.

## What I Learned

- **Restricted Shells:** Experienced firsthand how `ssh` logins can be restricted to custom programs or limited shells, which don't allow arbitrary command execution.

- **`/etc/passwd` for Shell Identification:** Confirmed the utility of `/etc/passwd` (specifically its 7th field) for identifying a user's configured shell.

- **Pager Escapes (`more`/`less` to `vi`/`vim`):** Discovered a common privilege escalation/bypass technique: using an interactive pager's features (`v` to enter `vi`/`vim`) to gain access to a more powerful environment.

- **`vi`/`vim` Shell Escapes and Shell Settings:** Learned that `vi`/`vim` provides shell escape commands (`:shell`, `!command`) and that its `shell` option (`:set shell=...`) can be modified to specify the desired shell to escape to. This is a crucial skill for breaking out of jailed or restricted environments.

- **Terminal Size Importance:** Realized how terminal window size can affect the interactive behavior of tools like `more` or `less`.
