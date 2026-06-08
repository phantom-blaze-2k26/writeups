# HASBLCTF Writeup: Lineup

**CTF Name:** HASBLCTF  
**Category:** Web Exploitation  
**Challenge/Task Name:** Lineup  
**Difficulty:** Medium/Hard  
**Author:** Phantom Blaze  
**Target URL:** `http://34.77.68.154:10204`

## Objective
Find the correct football lineup (11 players in a 4-2-3-1 formation) by discovering hidden hints across the website, then brute-force the remaining positions to unlock the flag.

## Solution / Walkthrough

### 1. Discovering the 5 Hidden Hints
The website contains 5 hints scattered across various locations that require thorough web reconnaissance:

| # | Location | Hint Content |
|---|----------|-------------|
| 1 | JS source map (`.map` file) | GK = Polish player |
| 2 | CSS source map (`.map` file) | ST = Norwegian, Premier League |
| 3 | `/robots.txt` | LW = Brazilian, Spanish club + reveals `/secret-lineup` |
| 4 | `/secret-lineup` hidden `<span>` | CAM = German, PL, age 23 |
| 5 | `X-Hint-5` HTTP header | RB = English, La Liga |

### 2. Resolving the Hints to Players

| Position | Hint | Player |
|----------|------|--------|
| **GK** | Polish | Wojciech Szczesny |
| **ST** | Norwegian, Premier League | Erling Haaland |
| **LW** | Brazilian, Spanish club | Vinicius Jr |
| **CAM** | German, PL, age 23 | Florian Wirtz |
| **RB** | English, La Liga | Trent Alexander-Arnold |

### 3. Brute-Forcing the Remaining 6 Positions
With 5 positions locked in, the remaining positions (CB×2, LB, CDM×2, RW) needed to be brute-forced. I built a script that tried all combinations from a curated list of top players:

```python
import requests
from itertools import permutations

known = {
    "GK": "Wojciech Szczesny",
    "RB": "Trent Alexander-Arnold",
    "ST": "Erling Haaland",
    "LW": "Vinicius Jr",
    "CAM": "Florian Wirtz"
}

candidates = {
    "CB": ["Virgil van Dijk", "William Saliba", ...],
    "LB": ["Theo Hernandez", "Alphonso Davies", ...],
    "CDM": ["Rodri", "Aurelien Tchouameni", ...],
    "RW": ["Bukayo Saka", "Mohamed Salah", ...]
}

# Try all combinations via POST to /api/submit-lineup
# Found the correct answer after 359 attempts
```

### 4. The Winning Lineup (4-2-3-1)

| Position | Player |
|----------|--------|
| **GK** | Wojciech Szczesny |
| **RB** | Trent Alexander-Arnold |
| **CB** | Virgil van Dijk |
| **CB** | William Saliba |
| **LB** | Theo Hernandez |
| **CDM** | Rodri |
| **CDM** | Aurelien Tchouameni |
| **CAM** | Florian Wirtz |
| **LW** | Vinicius Jr |
| **RW** | Bukayo Saka |
| **ST** | Erling Haaland |

**Flag:**  
`HASBL{7his_734m_C4n_Win_Ch4mpi0ns_L34gu3}`
