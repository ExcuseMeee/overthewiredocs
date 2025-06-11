# OverTheWire Bandit: Level 30 → Level 31

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 30.

## Problem Statement

The goal for Bandit Level 30 was to find the password for Bandit 31. This was located within a Git repository accessible via SSH at `ssh://bandit30-git@localhost/home/bandit30-git/repo` on port `2220`. The password for the `bandit30-git` user was the same as the current `bandit30` password. The task involved cloning the repository and finding the password within it.

## My Attempts

My attempts focused on a systematic investigation of the Git repository, similar to previous levels, to uncover the hidden password.

1.  **Repository Cloning:**
    After logging into `bandit30`, I navigated to `/tmp/` and created a temporary directory using `mktemp -d`. Within this directory, I cloned the repository, explicitly specifying the non-standard SSH port `2220`:

    ```
    git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo .
    ```

    I provided the `bandit30` password when prompted, and the repository was successfully cloned.

2.  **Initial File Check:**
    Inside the cloned repository, I executed `ls` and found a `README.md` file. Reading its content confirmed it did not contain the password.

3.  **Investigating Git History and Branches (Unsuccessful):**

    - I performed `git status` to ensure the working tree was clean.

    - I checked the commit history with `git log`, which only showed the initial commit.

    - I listed local and remote-tracking branches using `git branch` and `git branch -a`, but found no additional branches.

    - I also attempted to explore the `.git` directory directly, looking for clues in its internal structure, but found nothing immediately useful.

## The Correct Solution

The solution involved checking for Git tags, which are often used to mark significant points in a repository's history.

1.  **Discovering Git Tags:**
    Since the password was not found in active branches or their linear history, I decided to check for Git tags. I executed `git tag`:

    ```
    git tag
    ```

    This command revealed a single tag named `secret`.

2.  **Inspecting the Tagged Content:**
    Knowing that tags can point to specific commits (and thus, specific states of the repository), I used `git show` to inspect the content pointed to by the `secret` tag:

    ```
    git show secret
    ```

    This command displayed the content associated with the `secret` tag, which included the password for Bandit Level 31.

## What I Learned

- **Git Tags (`git tag`, `git show <tag_name>`):** Learned that Git tags are used to mark specific points in history as important. These points (often commits) can contain valuable information not immediately visible in the active branches. `git show <tag_name>` is the command to inspect the content a tag points to.

- **Comprehensive Git Investigation:** Reinforced the importance of checking all common Git components (`status`, `log`, `branch`, `branch -r`, and `tag`) when searching for hidden information in a repository. Not all clues are in the main branch or its linear history.

- **Beyond Branches and Commits:** Understood that Git stores various types of references (branches, tags, HEAD, remote-tracking branches) which all point to commits, trees, or blobs, and all can be sources of information.
