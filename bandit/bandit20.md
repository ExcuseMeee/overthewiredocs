# OverTheWire Bandit: Level 20 → Level 21

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 20.

## Problem Statement

The goal for Bandit Level 20 was to obtain the password for Bandit 21. This required using a `setuid` binary in the home directory that connects to `localhost` on a user-specified port. This binary would read a line of text from that connection, compare it to the Bandit 20 password, and if correct, transmit the Bandit 21 password. A key hint suggested connecting to my "own network daemon."

## My Attempts

My process involved identifying the binary, understanding its function, and then finding the correct port to interact with.

1.  **Binary Identification and Usage:**
    After logging in, I executed `ls` and found the binary `suconnect`. Running `file suconnect` confirmed it was a setuid executable. Executing `./suconnect` without arguments showed it required a port number as a command-line argument.

2.  **Port Discovery and Misinterpretation:**
    I ran `nmap localhost` to discover open ports, which included `50001/tcp unknown`. Probing this port with `nmap -p50001 -sV localhost` still showed "unknown" service, but the Nmap fingerprint snippet `"Wrong! Please enter the correct current password."` made me initially conclude it was the target. My misunderstanding was that `suconnect` would _send_ the password to this port.

3.  **Corrected Understanding and Initial Test:**
    I corrected my understanding: `suconnect` _reads_ the Bandit 20 password from the specified port, then transmits the next password. Testing ports with `nc` revealed port `1234` provided the Bandit 20 password. I then ran `./suconnect 1234`, which outputted:

    ```
    Read: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
    Password matches, sending next password
    ```

    However, the Bandit 21 password did not appear in my terminal or in any new files.

4.  **Interpreting "Transmit" and Hint:**
    This led to the realization that `suconnect`, acting as a client, would "transmit" the password _back to the server it connected to_. The level's hint about "connecting to your own network daemon" became crucial, indicating I needed to _be_ the server.

## The Correct Solution

The solution involved setting up a listening service to capture the password transmitted by `suconnect`.

1.  **Setting up a Listener:**
    I initiated a `tmux` session and created two panes. In one pane, I started a Netcat listener on an arbitrary unused port (e.g., 5000):

    ```bash
    nc -lkp 5000
    ```

    I first tested this listener by connecting from the other `tmux` pane (`nc localhost 5000`) and successfully exchanged messages.

2.  **Executing `suconnect` to the Listener:**
    In the second `tmux` pane, I executed `suconnect`, directing it to connect to my listener port:

    ```bash
    ./suconnect 5000
    ```

3.  **Sending the Required Password:**
    As `suconnect` connected to my `nc` listener, I sent the Bandit 20 password into the listener pane and pressed Enter.

4.  **Receiving the Next Password:**
    Upon successful password validation, `suconnect` transmitted the Bandit 21 password, which appeared directly in my `nc` listener pane.

## What I Learned

- **Client-Server Interaction:** This level deeply clarified the roles of client and server in network communication. `suconnect` acts as a client that initiates a connection and expects to receive data, while I had to set up `nc` as a generic server to receive data back.

- **`nc` for Listening (`-lkp`):** Mastered using `nc` in listen mode (`-l`) with port specification (`-p`) and persistence (`-k` for some versions, though `-l` implies it often for single connections, useful for repeated tries or understanding behavior) to create temporary network services.

- **Interpreting `setuid` Binary Behavior:** Understood that a `setuid` binary's "transmission" often means printing to its connected socket, requiring a listener on the other end to capture the output.

- **`tmux` for Multitasking:** Effectively used `tmux` to manage multiple terminal sessions within a single SSH connection, essential for simultaneously running a listener and the client binary.

- **Careful Reading of Hints:** The importance of interpreting challenge notes (e.g., "connect to your own network daemon") to guide the solution.
