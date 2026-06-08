# HASBLCTF Writeup: Cat (VIC Cipher)

**CTF Name:** HASBLCTF  
**Category:** Cryptography  
**Challenge/Task Name:** VIC Cipher  
**Difficulty:** Medium/Hard  
**Author:** Phantom Blaze  
**Target File:** `chall (7).py`

## Objective
Reverse-engineer a custom cipher implementation that combines a Straddling Checkerboard Cipher with keystream addition to decrypt the encrypted flag.

## Solution / Walkthrough

### 1. Analyzing the Encryption
The provided Python script (`chall (7).py`) implements a multi-layered encryption scheme:

1. **Preprocessing**: Special characters are expanded into words:
   - `{` → `LCURL`
   - `}` → `RCURL`
   - `_` → `SCORE`
   - Digits → spelled out (e.g., `1` → `ONE`, `3` → `THREE`)

2. **Straddling Checkerboard Encoding**: Each letter is mapped to 1 or 2 digits using a straddling checkerboard table (a variant of the VIC cipher used by Soviet spies).

3. **Keystream Addition**: A keystream is generated from the key `PHANTOM` via **chain addition** (each digit is the sum of the previous two, mod 10). This keystream is added digit-by-digit (mod 10) to the encoded digits.

### 2. Reversing the Cipher

To decrypt:
1. **Subtract the keystream**: Generate the same keystream from key `PHANTOM` and subtract it (mod 10) from each digit of the ciphertext.
2. **Decode the checkerboard**: Map the resulting digits back to letters using the inverse checkerboard table.
3. **Reverse preprocessing**: Replace word expansions back to their original characters (`LCURL` → `{`, `SCORE` → `_`, etc.).

### 3. The Decryption Script

```python
key = "PHANTOM"

# Generate keystream via chain addition
keystream_seed = [ord(c) - ord('A') for c in key]
keystream = keystream_seed[:]
while len(keystream) < len(ciphertext):
    keystream.append((keystream[-1] + keystream[-2]) % 10)

# Subtract keystream from ciphertext
decoded_digits = [(c - k) % 10 for c, k in zip(ciphertext, keystream)]

# Decode straddling checkerboard
plaintext = decode_checkerboard(decoded_digits)

# Reverse word expansions
plaintext = plaintext.replace("LCURL", "{").replace("RCURL", "}")
plaintext = plaintext.replace("SCORE", "_")
# ... reverse digit words
```

### 4. Retrieving the Flag
Running the decryption produces the original plaintext flag.

**Flag:**  
`HASBL{V1C_15_N07_JUST_4_C1PH3R}`
