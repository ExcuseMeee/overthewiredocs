# OverTheWire Bandit: Level 24 → Level 25

This document details the challenge, the steps taken, and the key learnings from completing Bandit Level 24.

## Problem Statement

The goal for Bandit Level 24 was to obtain the password for Bandit 25. This was guarded by a daemon listening on port `30002` on `localhost`. The daemon required two inputs: the Bandit 24 password and a secret 4-digit numeric pincode (ranging from `0000` to `9999`). The challenge implied that the pincode could only be found by brute-forcing all 10,000 combinations, and importantly, that new connections were not needed for each attempt, suggesting a persistent connection.

## My Attempts

My approach focused on understanding the daemon's interaction and then devising a brute-force strategy.

1.  **Initial Daemon Interaction:**
    I connected to the daemon using `nc`:

    ```
    nc localhost 30002
    ```

    The daemon responded, confirming its functionality as described. I then manually entered a wrong password and observed the exact failure message:

    ```
    Wrong! Please enter the correct current password and pincode. Try again.
    ```

    This provided the precise string to filter out during brute-forcing.

2.  **Generating Pincode Combinations:**
    I knew I needed to generate all 10,000 4-digit numeric combinations (0000-9999). I considered using `printf "%04d"` within a loop, as it's ideal for padding numbers with leading zeros.

3.  **Programmatic Input Challenge:**
    A key challenge was how to programmatically send multiple inputs (password then pincode) sequentially to `nc` over a single, persistent connection without it closing after the first input. Manual input was not feasible for 10,000 attempts.

4.  **Filtering Output:**
    Another challenge was to avoid having the correct password buried in thousands of "Wrong!" messages. I knew I needed a way to filter the output.

## The Correct Solution

The solution involved combining a loop to generate all password/pincode combinations with `nc` for persistent connection and `grep` for output filtering.

1.  **Constructing the Brute-Force Command:**
    I used a `for` loop with `seq` to iterate from 0 to 9999. Inside the loop, `printf` was used to format each attempt as two lines: the Bandit 24 password, followed by the 4-digit padded pincode. This entire sequence was wrapped in a subshell `()` to ensure all inputs were generated before being piped as a single stream to `nc`.

    The `nc -N` flag was used to ensure `nc` properly handles the end of the input stream without immediate shutdown, maintaining the connection until all data is sent. Finally, `grep -v "Wrong!"` was employed to filter out all the incorrect attempt messages, leaving only the successful response.

    The complete command executed was:

    ```
    { for i in $(seq 0 9999); do printf "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 %04d\n" $i; done; } | nc -N localhost 30002 | grep -v "Wrong!"
    ```

2.  **Retrieving the Password:**
    Executing this command successfully displayed the password for Bandit Level 25, as all "Wrong!" messages were filtered out.

## What I Learned

- **Brute-Forcing over Persistent Connections:** Gained practical experience in setting up a brute-force attack against a network service that maintains a persistent connection.

- **`printf` for Formatted Output:** Mastered using `printf "%0Nd"` for padding numbers with leading zeros, essential for generating fixed-width numeric inputs.

- **Piping Multiple Inputs Programmatically:** Learned to use a subshell `()` and `for` loops with `printf` to generate a continuous stream of multiple lines of input, which can be piped to a command like `nc` to simulate sequential user input.

- **`nc -N` for Connection Handling:** Understood the importance of the `nc -N` flag (shutdown network writes on EOF) for ensuring `nc` sends all piped data before closing the connection, crucial for persistent brute-forcing.

- **`grep -v` for Output Filtering:** Effectively used `grep -v` to filter out unwanted output (e.g., error messages) from a large stream, making it easy to spot the successful result. This is a vital skill for automating and analyzing command-line output.

- **Problem-Solving Strategy:** Reinforced the process of breaking down a complex problem (brute-forcing a network service) into smaller, manageable steps, including understanding the service's expected input/output and leveraging appropriate Linux utilities.
