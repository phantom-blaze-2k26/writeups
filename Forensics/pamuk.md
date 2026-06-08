# HASBLCTF Writeup: Pamuk

**CTF Name:** HASBLCTF  
**Category:** Forensics  
**Challenge/Task Name:** Pamuk  
**Difficulty:** Easy  
**Author:** Phantom Blaze  
**Target:** `pamuk.txt` (local file)

## Objective
Fight alongside with Pamuk, in order to end the homework once and for all. Decode the hidden flag from the provided text file.

## Solution / Walkthrough

### 1. Initial Analysis
The `pamuk.txt` file contains a large block of Base64 encoded data. Running it through a Base64 decoder reveals that the decoded output is a JPEG image — a picture of a cat wearing sunglasses.

### 2. Visual Inspection
Upon viewing the decoded JPEG image, we can see a red string of hexadecimal characters overlaid across the middle of the image:
```
484153424c7b50346d756b5f773174685f676c34733565357d
```

### 3. Decoding the Hex String
Converting this hexadecimal string to ASCII text:

```python
flag_hex = "484153424c7b50346d756b5f773174685f676c34733565357d"
print(bytes.fromhex(flag_hex).decode())
```

This yields our flag.

**Flag:**  
`HASBL{P4muk_w1th_gl4s5e5}`
