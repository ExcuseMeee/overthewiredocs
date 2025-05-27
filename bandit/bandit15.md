# OverTheWire Bandit: Level 15 → Level 16

This document outlines the challenge, my attempts, the correct solution, and the key learnings from completing Bandit Level 15.

## Problem Statement

The goal for Bandit Level 15 was to find the password for Bandit 16 by submitting the current level's password to **port 30001 on localhost** using **SSL/TLS encryption**.

## My Attempts

My initial attempt used `nc` (netcat) to connect:
```
nc localhost 30001
```
This failed, either hanging or showing garbled output. This indicated that a direct TCP connection was insufficient, as the service required SSL/TLS encryption.

## The Correct Solution

Recognizing the SSL/TLS requirement, I turned to `openssl`'s `s_client` subcommand.

1.  **Establishing the SSL/TLS Connection:**
    I connected using:

    ```
    openssl s_client localhost:31046
    ```

    After some initial SSL handshake output, the terminal became interactive.

2.  **Submitting the Password:**
    I typed the current level's password into the interactive prompt and pressed Enter.

3.  **Retrieving the Next Password:**
    The server responded with the password for Bandit Level 16.

4.  **Automated Approach (Learned Post-Completion):**
    A more efficient method is to pipe the password directly using the `-quiet` flag:

    ```
    echo "YOUR_BANDIT_15_PASSWORD" | openssl s_client -quiet localhost:30001
    ```

## What I Learned

* **SSL/TLS Requirement:** Standard tools like `nc` don't support encryption; `openssl s_client` is necessary for SSL/TLS connections.

* **`openssl s_client` Usage:** Learned to use `openssl s_client` for secure connections and its interactive mode.

* **Piping Input & `-quiet`:** The `-quiet` flag is key for automating input via pipes, suppressing verbose output and allowing direct data submission.

* **Nmap's Default Port Scan:** `nmap` only scans the 1000 most common ports by default. Custom ports (like 30001) require explicit specification with the `-p` flag (e.g., `nmap -p 30001 localhost`).

* **Effective `man` Page Navigation:** The hint in the problem statement emphasized the importance of carefully reading `man` pages, especially for specific sections like "CONNECTED COMMANDS."