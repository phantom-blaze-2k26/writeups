# HASBLCTF Writeup: Digits

**CTF Name:** HASBLCTF  
**Category:** Forensics  
**Challenge/Task Name:** Digits  
**Difficulty:** Easy/Medium  
**Author:** Phantom Blaze  
**Target File:** `digits.bin`

## Objective
Decode a binary file containing streams of 0s and 1s to reconstruct the original hidden image and retrieve the flag.

## Solution / Walkthrough

### 1. Initial Analysis
The provided `digits.bin` file contains raw binary digit characters (`0` and `1` as ASCII text). Each character represents a single bit of the original file.

### 2. Binary-to-Bytes Conversion
The file is a text representation of binary data — every 8 characters form one byte. We read the entire file, group the bits into bytes, and reconstruct the original binary data:

```python
with open('digits.bin', 'r') as f:
    bits = f.read().strip()

# Convert every 8 bits to a byte
data = bytearray()
for i in range(0, len(bits), 8):
    byte = int(bits[i:i+8], 2)
    data.append(byte)

with open('output.jpg', 'wb') as f:
    f.write(data)
```

### 3. Inspecting the Output
The reconstructed file is a JPEG image. Opening it reveals a scenic photograph of Istanbul with the flag printed on it.

### 4. Reading the Flag
The flag is clearly visible in the decoded image.

**Flag:**  
`HASBL{Istanbul_1s_b3atiful_c1ty}`
