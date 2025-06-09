# OverTheWire Bandit: Level 28 → Level 29

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 28.

## Problem Statement

The goal for Bandit Level 28 was to find the password for Bandit 29. This was located within a Git repository accessible via SSH at `ssh://bandit28-git@localhost/home/bandit28-git/repo` on port `2220`. The `bandit28-git` user's password was the same as the current `bandit28` password. The task involved cloning the repository and then finding the password within it.

## My Attempts

My attempts focused on successfully cloning the Git repository and then investigating its history.

1.  **Repository Cloning:**
    After logging into `bandit28`, I navigated to `/tmp/` and created a temporary directory using `mktemp -d`. Within this directory, I cloned the repository, explicitly specifying the non-standard SSH port `2220` (a lesson from a previous level):

    ```
    git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo .
    ```

    I provided the `bandit28` password when prompted, and the repository was successfully cloned.

2.  **Initial Password Discovery and Obfuscation:**
    Inside the cloned repository, I executed `ls` and found a file named `README.md`. Upon reading `README.md`, I found that the password was present but obfuscated (represented by 'x's).

3.  **Investigating Git History:**
    To find the original password, I used `git log` to inspect the repository's commit history. This revealed that the password had existed in a past commit before a later commit obfuscated it. My new goal became to recover the content of that past commit.

## The Correct Solution

The solution involved using Git's history manipulation capabilities to revert the obfuscation and reveal the original password.

1.  **Identifying the Obfuscating Commit:**
    From the `git log` output, I identified the specific commit that introduced the obfuscation (i.e., the commit _after_ the one containing the clear password).

2.  **Reverting the Obfuscation:**
    I used the `git revert` command on the commit that performed the obfuscation. This command creates a _new commit_ that undoes the changes of the specified commit, effectively restoring the `README.md` to its state before the obfuscation.

    ```
    git revert <obfuscating_commit_hash>
    ```

3.  **Retrieving the Password:**
    After the `git revert` operation completed (which might involve a `vi` editor prompt to confirm the revert message, just save and exit), I re-read the `README.md` file. It now contained the clear password for Bandit Level 29.

## What I Learned

- **Git History Examination (`git log`):** Reinforced the importance of using `git log` to inspect repository history, identifying when and how specific changes were introduced or modified. This is crucial for understanding how files evolve.

- **Git Revert:** Learned how to use `git revert` to undo specific commits by creating new "undo" commits. This is a safe way to reverse changes in Git history without rewriting it.

- **Recovering Past Data:** Discovered a practical application of Git's version control: retrieving previous versions of files that have been modified or obfuscated in later commits.

- **Persistence of Information in Git:** Understood that even when information is changed or removed from the current working copy, it often remains recoverable in the Git history.
