# OverTheWire Bandit: Level 27 → Level 28

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 27.

## Problem Statement

The goal for Bandit Level 27 was to find the password for Bandit 28. This password was located within a Git repository accessible via SSH. The repository URL was `ssh://bandit27-git@localhost/home/bandit27-git/repo` on port `2220`. The password for the `bandit27-git` user was the same as the current `bandit27` password. The task involved cloning the repository and then finding the password within it.

## My Attempts

My attempts focused on correctly cloning the Git repository, overcoming common `git` and `ssh` interaction issues.

1.  **Initial Clone Attempt in Home Directory:**
    After logging into `bandit27`, I tried to clone the repository directly in the home directory:

    ```
    git clone ssh://bandit27-git@localhost/home/bandit27-git/repo
    ```

    This resulted in a `fatal: could not create work tree dir 'repo': Permission denied` error, indicating that I lacked write permissions in the home directory to create the repository's working directory.

2.  **Moving to `/tmp` and Subsequent SSH Error:**
    I changed to the `/tmp/` directory, created a temporary folder, and then attempted to clone there:

    ```
    git clone ssh://bandit27-git@localhost/home/bandit27-git/repo .
    ```

    This time, I received several errors, including:

    ```
    Could not create directory '/home/bandit27/.ssh' (Permission denied).
    Failed to add the host to the list of known hosts (/home/bandit27/.ssh/known_hosts).
    !!! You are trying to log into this SSH server on port 22, which is not intended.
    bandit27-git@localhost: Permission denied (publickey).
    fatal: Could not read from remote repository.
    ```

    I deduced that the primary issue was `git clone` attempting to use the default SSH port `22` instead of the specified `2220`. The `known_hosts` and `publickey` permission denied errors were a symptom of this incorrect port attempt, as SSH key authentication requires the correct host/port combination.

## The Correct Solution

The solution involved explicitly specifying the non-standard SSH port `2220` within the Git clone URL.

1.  **Specifying the SSH Port in `git clone`:**
    After consulting the `git clone` manual for port specification, I executed the clone command, including the port number `2220` directly in the URL:

    ```
    git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo .
    ```

    When prompted, I provided the password for `bandit27-git` (which was the same as the `bandit27` password). This command successfully cloned the Git repository into the current temporary directory.

2.  **Locating and Retrieving the Password:**
    Inside the newly cloned repository directory, I executed `ls` and found a file named `README`. Reading the content of this `README` file revealed the password for Bandit Level 28.

## What I Learned

- **`git clone` and SSH Ports:** Crucially learned how to specify a non-standard SSH port when cloning a Git repository via SSH. The format is `ssh://user@host:port/path/to/repo.git`. This is a common requirement when working with Git servers not using the default SSH port 22.

- **Git/SSH Integration Issues:** Understood that `git clone` relies on `ssh` for authentication and transport when using `ssh://` URLs, and thus, `ssh` configuration (like `known_hosts`) and port issues can directly impact `git` operations.

- **Permissions for Cloning:** Reconfirmed that cloning a Git repository requires write permissions in the target directory where the repository's working tree will be created.

- **Common CTF Pattern: README files in Repos:** Identified a common CTF pattern where passwords or clues are placed in `README` files within cloned repositories.
