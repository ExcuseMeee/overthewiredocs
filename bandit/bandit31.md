# OverTheWire Bandit: Level 31 → Level 32

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 31.

## Problem Statement

The goal for Bandit Level 31 was to find the password for Bandit 32. This involved a Git repository accessible via SSH at `ssh://bandit31-git@localhost/home/bandit31-git/repo` on port `2220`. The task required cloning the repository, creating a file named `key.txt` with specific content (`'May I come in?'`), pushing it to the `master` branch, and retrieving the password for the next level.

## My Attempts

Initial attempts focused on preparing the repository and the required file for pushing.

1.  **Repository Cloning:**
    After logging into `bandit31`, a temporary directory was created in `/tmp/` using `mktemp -d`. The repository was then cloned into this directory, specifying the non-standard SSH port:

    ```
    git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo .
    ```

    The `bandit31` password was provided when prompted, and the clone succeeded.

2.  **Reading Instructions and File Creation:**
    The `README.md` file was read, instructing the push of `key.txt` with content `'May I come in?'` to the `master` branch. `key.txt` was created with this exact content using `vim`.

3.  **Handling `.gitignore`:**
    An attempt to stage `key.txt` with `git add key.txt` failed due to `*.txt` being present in `.gitignore`. The addition was then forced:

    ```
    git add -f key.txt
    ```

    This successfully staged the file.

4.  **Committing and Pushing Attempt:**
    After staging, the change was committed:

    ```
    git commit -m "Add key.txt for Bandit 32"
    ```

    Finally, `git push` was executed to send the commit to the remote repository.

## The Correct Solution

The solution involved creating and pushing the correctly configured `key.txt` file, which triggered a server-side process that released the password.

1.  **Repository Setup and File Creation:**
    As described in "My Attempts," the repository was cloned into a temporary directory. The `key.txt` file was then created with the specified content and explicitly forced to be staged despite the `.gitignore` rule:

    ```
    git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo .
    # ... (create key.txt using vim) ...
    git add -f key.txt
    ```

2.  **Committing the File:**
    Next, the `key.txt` file was committed to the local repository:

    ```
    git commit -m "Add key.txt for Bandit 32"
    ```

3.  **Executing `git push`:**
    The crucial step was to execute `git push` to send the new commit to the remote server:

    ```
    git push
    ```

4.  **Retrieving the Password from Remote Output:**
    The `git push` operation returned a response containing output directly from the remote server (prefixed with `remote:`). This output included the password for Bandit 32:

    ```
    remote: ### Attempting to validate files... ####
    remote:
    remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
    remote:
    remote: Well done! Here is the password for the next level:
    remote: xxxxx
    remote:
    remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
    To ssh://localhost:2220/home/bandit31-git/repo
     ! [remote rejected] master -> master (pre-receive hook declined)
    error: failed to push some refs to 'ssh://localhost:2220/home/bandit31-git/repo'
    ```

    The "remote rejected" message indicated a `pre-receive` Git hook on the server intentionally declined the push after validating the file and providing the password, ensuring the repository's state remained clean.

## What I Learned

- **Git Hooks (`pre-receive`):** Gained practical understanding of server-side Git hooks. A `pre-receive` hook can validate incoming pushes, generate output to the client (like the password here), and then reject the push to prevent unintended changes to the repository's history.

- **Force-Adding Files (`git add -f`):** Learned how to bypass `.gitignore` rules to stage files for commit using the `-f` (force) flag.

- **Password Retrieval from Remote Output:** Discovered a common CTF pattern where the challenge's flag is embedded directly within the remote server's output during a Git operation (like a push).

- **Purposeful Push Rejection:** Understood that a `git push` being rejected is not always an error to fix, but can be a deliberate part of the challenge's design.
