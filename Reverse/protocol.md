# HASBLCTF Writeup: Protocol

**CTF Name:** HASBLCTF  
**Category:** Reverse Engineering / PWN  
**Challenge/Task Name:** Pr0t0c0l1337  
**Difficulty:** Medium  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10102`

## Objective
Activate the Pr0t0c0l1337 by reverse-engineering the custom binary protocol and crafting a payload to trigger the flag-reading function.

## Solution / Walkthrough

### 1. Reverse Engineering the Protocol
Analyzing the decompiled binary (`disasm.c`), we find the function `sub_4011e5` which implements a custom binary protocol parser. It validates the incoming data byte-by-byte according to a strict structure:

| Offset | Length | Expected Value | Description |
|--------|--------|----------------|-------------|
| 0–1 | 2 bytes | `0x5227` (LE: `\x52\x27`) | Magic header |
| 2–145 | 144 bytes | `0x84` repeated | Padding validation |
| 146–149 | 4 bytes | `root` | Keyword string |
| 150 | 1 byte | Command ID | Function selector |

### 2. Identifying the Flag Command
The command byte at offset 150 controls which function is called:
- `0` → `sub_401356` (responds "pong")
- `1` → `sub_40136c`
- `2` → Exits the program
- **`3` → `sub_401390` — reads and prints `flag.txt`!**

### 3. Crafting the Exploit
The payload is constructed as follows:
```python
from pwn import *

r = remote('34.77.68.154', 10102)

payload  = b'\x52\x27'       # Magic header (0x2752 in little-endian)
payload += b'\x84' * 144     # 144 bytes of padding (each must be 0x84)
payload += b'root'           # Keyword
payload += b'\x03'           # Command 3 → read flag.txt

r.send(payload)
print(r.recvall().decode())
```

### 4. Retrieving the Flag
Upon receiving the valid payload, the server processes all checks successfully and triggers command `3`, which reads and returns the flag.

**Flag:**  
`HASBL{TH3_PR0T0C0L_1337_IS_ACTIVATED}`
