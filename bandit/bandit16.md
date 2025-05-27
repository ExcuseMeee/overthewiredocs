# OverTheWire Bandit: Level 16 → Level 17

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 16.

## Problem Statement

Bandit Level 16 required finding the next level's credentials by submitting the current password to a server on `localhost` within port range `31000-32000`. The challenge involved identifying listening servers, distinguishing SSL/TLS from plain TCP, and locating the single server providing the correct credentials. A hint suggested checking "CONNECTED COMMANDS" in the `man` page for SSL negotiation messages.

## My Attempts

My approach involved systematically discovering and probing ports.

1.  **Initial Port Scan with `nmap`:**
    I scanned `localhost` for open ports in the `31000-32000` range:

    ```
    nmap -p31000-32000 localhost
    ```

    This revealed five open ports, all initially showing "unknown" services:
    `31046/tcp`, `31518/tcp`, `31691/tcp`, `31790/tcp`, `31960/tcp`.

2.  **Service Version Detection with `nmap -sV`:**
    To identify SSL/TLS and specific services, I ran a version detection scan with `--version-intensity 2`:

    ```
    nmap -p31046-31960 -sV --version-intensity 2 localhost
    ```

    _(Self-correction: Scanning the full `31000-32000` range with `-sV` would have taken similar time, as Nmap only probes already open ports.)_

    The results were:

    | PORT | STATE | SERVICE | VERSION |
    | ---- | ----- | ------- | ------- | 
    | 31046/tcp | open | echo | |
    | 31518/tcp | open | ssl/echo | |
    | 31691/tcp | open | echo | |
    | 31790/tcp | open | ssl/unknown | |
    | 31960/tcp | open | echo | |
    
    Port 31790's fingerprint included: `"Wrong! Please enter the correct current password."`

3.  **Further `nmap -sV` Probing on 31790:**
    Targeted `-sV` scans on port 31790 (default and max intensity `9`) still reported `ssl/unknown`, indicating a custom or unrecognized application protocol over SSL.

4.  **Initial `nc` Connection Attempt (Unsuccessful):**
    I briefly considered or attempted `nc localhost 31046` (a plain-text service) but quickly realized the challenge's SSL/TLS requirement for the key server.

## The Correct Solution

Port `31790` was the target due to its `ssl/unknown` service and the password-related hint in its Nmap fingerprint. The other services were misleading echoes.

1.  **Connecting with `openssl s_client`:**
    I used `openssl s_client` to establish an SSL/TLS connection:

    ```
    openssl s_client localhost:31790
    ```

    The connection displayed certificate verification messages (e.g., `self-signed certificate`, `CN = SnakeOil`), but the connection proceeded.

2.  **Submitting the Password:**
    After the certificate output, I typed the `bandit16` password into the interactive prompt.

3.  **Retrieving Credentials:**
    The server responded with the Bandit Level 17 credentials, an RSA private key.

## What I Learned

- **`nmap` Port Scanning:** Learned to use `-p` for specific ranges and that `-sV` only probes open ports. Default `--version-intensity` is 7.

- **Interpreting `nmap` Output:** Distinguished `echo` from `ssl/echo` and understood `ssl/unknown` as an encrypted, unrecognized service. Recognized hints in Nmap fingerprints.

- **SSL/TLS vs. Plain TCP:** Confirmed `openssl s_client` is essential for encrypted services, unlike `nc`.

- **Self-Signed Certificates:** Understood `openssl s_client` connects to self-signed certificates, issuing warnings but allowing communication.

- **Interactive vs. Automated Input:** Reinforced interactive `openssl s_client` use and the utility of `echo "password" | openssl s_client -quiet ...` for automation.

- **SSH Key Credentials:** Realized next-level credentials can be SSH keys, requiring different handling.
