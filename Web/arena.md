# HASBLCTF Writeup: Arena

**CTF Name:** HASBLCTF  
**Category:** Web Exploitation  
**Challenge/Task Name:** ARENA.EXE  
**Difficulty:** Easy/Medium  
**Author:** Phantom Blaze  
**Target URL:** `http://34.77.68.154:10206`

## Objective
ARENA.EXE is a browser-based survival game. The objective is to retrieve the flag by bypassing the client-side seed validation used to initialize the game state.

## Solution / Walkthrough

### 1. Analyzing the Game Logic
Upon inspecting the web application's source code, we see the following game flow:
- The client generates a random `seed` (from 0 to 9999).
- The client sends this seed to the server via the `/api/start` endpoint using a POST request.
- The server responds with the seed, or, as observed in the code logic, a special `{status: 'flag', flag: '...'}` response if a secret condition is met.

### 2. Exploring the Hint System
The shop allows the purchase of an "Intel File" for 9999 coins. To speed up the process, we can analyze the hint API (`/api/buy-hint` and `/api/hint`). We discover that the `/api/buy-hint` endpoint accepts the cost (`9999`) from the client directly in the request body without verifying the player's actual balance server-side.

By sending a raw POST request to `/api/buy-hint` with `{"coins": 9999}`, the server replies with a token, which we then send to `/api/hint`.
This reveals the hint:
> `"negative leet as seed"`

### 3. Exploitation
"Leet" is famously associated with the number `1337`. "Negative leet" translates to `-1337`. The client-side logic only generates positive random numbers from 0 to 9999, so it would never generate this naturally. This aligns perfectly with the flag's post-completion message: *"You bypassed the client-side seed validation. The server never lies. The client always does."*

We bypass the client-side validation by directly submitting the payload to the `/api/start` endpoint:
```python
import requests

r = requests.post("http://34.77.68.154:10206/api/start", json={"seed": -1337})
print(r.json())
```

### 4. Retrieving the Flag
The server correctly processes the special seed and replies directly with the flag.

**Flag:**  
`HASBL{7his_S33d_Is_S0_Lucky}`
