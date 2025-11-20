# Verifiable Weighted Raffle System

This system provides a fair and verifiable way to draw winners from a list of participants, respecting weighted entries. It uses a nonce + seed mechanism to ensure unpredictability before the draw while allowing anyone to verify the results afterward.

## Files

*   `participants.json`: Stores the list of participants and their corresponding entry weights. Edit this file to manage raffle entries.
*   `main.py`: The main script to run the raffle draw. It reads participants from `participants.json`, generates a nonce, calculates a seed, shuffles entries, selects winners, and prints the results along with verification data.
*   `verify.py`: A tool to verify the results of a previous raffle draw. It takes the Base Seed and Nonce from a specific draw as input and recalculates the winners.

## Core Concepts & Components

1.  **Weighted Entries (`participants.json`)**:
    *   Participants and their number of entries are defined in the `participants.json` file as a JSON object (dictionary).
    *   The keys are the participant identifiers (e.g., usernames), and the values are their corresponding integer weights (number of entries).
    *   The system creates a "virtual hat" where each participant's name appears a number of times equal to their weight, ensuring probabilities are proportional to entries.

2.  **Nonce + Seed Mechanism**:
    *   **Base Seed**: A predetermined string (`BASE_SEED` variable in `main.py`) specific to the raffle series. This should be known or announced beforehand.
    *   **Nonce**: A random, unpredictable value generated *at the time of the draw* (`secrets.token_hex(16)` in `main.py`). This prevents predicting the outcome beforehand. For high-stakes raffles, this nonce ideally comes from a public, verifiable source generated *after* entries close (e.g., future block hash, lottery number). This script *simulates* this by generating it locally. The generated nonce is printed in the draw output.
    *   **Hashing (SHA-256)**: The `Base Seed` and the generated `Nonce` are combined and then processed using the secure SHA-256 hashing algorithm. This produces a unique, fixed-length "Final Seed". Any tiny change to the Base Seed or Nonce results in a completely different Final Seed.
    *   **PRNG Seeding**: Python's standard pseudo-random number generator (`random`) is seeded with the `Final Seed` (the SHA-256 hash). This makes the subsequent "random" operations (like shuffling) deterministic and reproducible *if you know the Final Seed*.

3.  **Shuffling & Unique Selection**:
    *   The "virtual hat" (list with weighted entries) is shuffled using the seeded random number generator. The order after shuffling is entirely determined by the `Final Seed`.
    *   The script iterates through the shuffled list and selects the first `N` (defined by `NUM_WINNERS`) *unique* participants encountered. This ensures no one can win more than once in a single draw.

## Verifiable Fairness

The system achieves verifiable fairness through these combined mechanisms:

*   **Transparency**: The participant list (`participants.json`), the `Base Seed`, the algorithm (code), and the hashing method (SHA-256) are open. The specific `Nonce` used for a draw is published in its results.
*   **Unpredictability**: Because the `Nonce` is generated randomly at draw time (or ideally, obtained from a future public source), the outcome cannot be predicted before the draw occurs.
*   **Reproducibility**: Anyone can take the published `Base Seed`, the specific `Nonce` from a given draw, and the `participants.json` file (as it was at the time of the draw), run the `verify.py` script (or manually perform the steps: combine, hash, seed, build list, shuffle, select), and they *must* arrive at the exact same list of winners. This proves that the announced winners are the legitimate result of the defined random process and weren't manipulated.

## How to Use

### Setup

1.  Ensure you have Python 3 installed.
2.  Save the files `main.py`, `verify.py`, and `participants.json` in the same directory.
3.  **Edit `participants.json`**: Update this file with your list of participants and their corresponding weights (number of entries). Ensure it's valid JSON.

### Running the Raffle

1.  Open a terminal or command prompt in the directory containing the files.
2.  Run the main script: `python main.py`
3.  The script will ask for:
    *   **`Winners Quantity`**  (the desired number of winners for the draw).
    *   **`Letters and numbers for randomness`** (the base seed).
5.  The script will output:
    *   Draw details (timestamp, number of winners, entry counts).
    *   The **`Base Seed`** used. <<< **RECORD THIS!**
    *   The **`Nonce Generated`** for this specific run. <<< **RECORD THIS!**
    *   The calculated Final Seed (SHA-256 Hash).
    *   The list of **`🏆 WINNERS 🏆`**. <<< **RECORD THESE!**

### Verifying a Raffle Draw

1.  Obtain the `Base Seed` and the specific `Nonce` that were published with the results of the raffle run you want to verify.
2.  Ensure your `participants.json` file accurately reflects the state of entries *at the time that specific raffle was run*. If entries changed later, verification against the *original* state is needed.
3.  Open a terminal or command prompt in the directory containing the files.
4.  Run the verification script: `python verify.py`
5.  Enter the exact `Base Seed` when prompted.
6.  Enter the exact `Nonce` from the original draw when prompted.
7.  Enter the *number* of winners that were *announced* for that draw when prompted.
8.  The script will recalculate the process and output the `Derived Winners`.
9.  **Compare**: Carefully compare the `Derived Winners` list printed by `verify.py` to the winners announced in the original raffle output. **If they match exactly, the draw is verified.** If they don't match, double-check your inputs (Base Seed, Nonce, Num Winners) and ensure the `participants.json` used by `verify.py` is correct for the time of the draw.
