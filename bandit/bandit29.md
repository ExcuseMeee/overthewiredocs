# OverTheWire Bandit: Level 29 → Level 30

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 29.

## Problem Statement

The goal for Bandit Level 29 was to find the password for Bandit 30. This password was located within a Git repository accessible via SSH at `ssh://bandit29-git@localhost/home/bandit29-git/repo` on port `2220`. The `bandit29-git` user's password was the same as the current `bandit29` password. The task involved cloning the repository and finding the password within it.

## My Attempts

My attempts focused on thoroughly investigating the Git repository, including its remote state and internal structure, to find the password.

1.  **Repository Cloning:**
    After logging into `bandit29`, I navigated to `/tmp/` and created a temporary directory using `mktemp -d`. Within this directory, I cloned the repository, explicitly specifying the non-standard SSH port `2220`:

    ```
    git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo .
    ```

    I provided the `bandit29` password when prompted, and the repository was successfully cloned.

2.  **Initial Password Discovery and Obfuscation:**
    Inside the cloned repository, I executed `ls` and found a file named `README.md`. Reading `README.md` revealed a line stating: `password: <no passwords in production!>`.

3.  **Investigating `master` Branch History and Remotes (Unsuccessful):**

    - I executed `git status` and `git branch` to confirm the repository was up-to-date locally on the `master` branch and that no other local branches existed.

    - I then performed `git fetch` and `git pull` to check for any new data or changes from the remote, but neither command yielded new information.

    - I explored Git's internal object database by navigating into the `.git/objects` directory and also by checking out previous commits, hoping to find the password in unreferenced or historical blobs. These attempts were unsuccessful.

## The Correct Solution

The solution involved discovering and inspecting a hidden remote-tracking branch that contained the password.

1.  **Discovering Remote Branches:**
    Recognizing that the password wasn't in the `master` branch's current state or its accessible history, I looked for other branches. I executed `git branch -r` (list remote-tracking branches), which revealed additional remote branches, including `origin/dev`.

2.  **Switching to the `dev` Branch:**
    I then switched to the `dev` branch to inspect its contents:

    ```
    git checkout dev
    ```

3.  **Locating and Retrieving the Password:**
    Within the `dev` branch, I executed `ls` and once again found the `README.md` file. Reading the content of this `README.md` file successfully revealed the clear password for Bandit Level 30.

## What I Learned

- **Remote-Tracking Branches (`git branch -r`):** Learned that `git clone` by default only checks out the primary branch (`master`/`main`). Critical information or features might reside in other, non-default branches, which can be discovered using `git branch -r`.

- **Branch Switching (`git checkout`):** Reinforced the use of `git checkout` to switch between branches, including remote-tracking branches (which are then created locally for inspection).

- **Decentralized Information in Git:** Understood that not all relevant information is necessarily in the `master` branch or its direct history. Data might be found in other branches (e.g., development branches) or even dangling objects.

- **"No Passwords in Production" Hint:** Interpreted this as a deliberate clue to look _outside_ the expected "production" (master) branch for sensitive data.

- **Systematic Git Investigation:** The process reinforced a systematic approach to Git challenges: check local status, pull/fetch remotes, list all branches (local and remote), and then investigate promising branches.
